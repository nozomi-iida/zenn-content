---
title: "NextAuth.js認証トークンをリフレッシュする方法"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["next", "typescript", "aws", "Cognito"]
published: true
---

NextAuth.js で認証トークン をリフレッシュする機能の実装をやります。
外部の認証サービスや、仕様によって代わりますが例えば Amazon Cognito の認証トークンはデフォルトだと 1 時間で有効期限がきれてしまい、リフレッシュ機能がないと 1 時間に 1 回ユーザーは強制的にログアウトしてしまいます。
1 時間に 1 回ログアウトしてしまうとユーザーはログインするのがめんどくさくなって離脱してしまうことでしょう。
そのため、トークンのリフレッシュ機能は忘れられがちですが優先度の高い機能だと思います！

プロバーダーは何でも良いのですが、今回は Amazon Cognito を使用した認証トークンのリフレッシュ機能を実装します

## 認証周りを作る

まずはササッと cognito の認証機能を作ります。

### 環境変数を設定する

```.env
COGNITO_CLIENT_ID=*********
COGNITO_CLIENT_SECRET=*********
COGNITO_ISSUER=*********
```

### [...nextauth].ts の実装

```pages/api/auth/[...nextauth].ts
import NextAuth from 'next-auth';
import CognitoProvider from 'next-auth/providers/cognito';

export default NextAuth({
  providers: [
    CognitoProvider({
      clientId: process.env.COGNITO_CLIENT_ID,
      clientSecret: process.env.COGNITO_CLIENT_SECRET,
      issuer: process.env.COGNITO_ISSUER,
    }),
  ],
});
```

### `_app.tsx`の編集

```_app.tsx
import { SessionProvider } from 'next-auth/react';
import type { AppProps } from 'next/app';
import '../styles/globals.css';

function MyApp({ Component, pageProps: { session, ...pageProps } }: AppProps) {
  return (
    <SessionProvider session={session}>
      <Component {...pageProps} />
    </SessionProvider>
  );
}

export default MyApp;
```

これで cognito を使った認証機能を実装することができました。

## 更新トークンを使用した認証トークンのリフレッシュ機能の実装

次に記事の本題でもある認証トークンのリフレッシュ機能を実装します。

### 認証トークンの期限を設定する

まず認証トークンの期限を token に設定できるようにします。
Cognito の場合`initialAuth`の API を実行すると`ExpiredIn`にトークンの有効期限の秒数が返ってきます(1 時間の場合は 3600)
トークンの有効期限を jwt コールバックで永続化します。

```pages/api/auth/[...nextauth].ts
export default NextAuth({
  providers: ...,
 callbacks: {
    async jwt({ token, user, account }) {
      // Initial sign in
      if (account && user) {
        return {
          accessToken: account.access_token,
          accessTokenExpires: Date.now() + account.expires_at * 1000,
          refreshToken: account.refresh_token,
          user,
        }
      }
      return token
    },
  },
});
```

`accessTokenExpires`で有効期限を永続化しています。
`accessTokenExpires`では今の時間から`expires_at`を ☓1000 した値を足すことで 1 時間後の UNIX 時間[^1]を有効期限としています。

### NextAuth の型を上書きする

上記のコードは実は型が正しくありません。

```node_modules/../next-auth.d.ts
export interface JWT extends Record<string, unknown>, DefaultJWT {
export interface DefaultJWT extends Record<string, unknown> {
    name?: string | null;
    email?: string | null;
    picture?: string | null;
    sub?: string;
}
```

↑NextAuth の JWT インターフェース

そのため、JWT のインターフェースを拡張して、型を正しくしましょう

```types/next-auth.d.ts
import { DefaultSession } from 'next-auth';

interface UserWithId extends DefaultSession["user"] {
  id?: string;
}

declare module 'next-auth/jwt' {
  /** Returned by the `jwt` callback and `getToken`, when using JWT sessions */
  interface JWT {
    user: UserWithId;
    accessToken?: string;
    refreshToken?: string;
    accessTokenExpires?: number;
    error?: string;
  }
}
```

これで JWT のインターフェースを上書きすることができました。

### 認証トークンをリフレッシュする関数を作成する

次に認証トークンをリフレッシュする関数を作成します。

```pages/api/auth/[...nextauth].ts
import NextAuth from 'next-auth';
import CognitoProvider from 'next-auth/providers/cognito';
import {
  InitiateAuthRequest,
  UserStatusType,
} from 'aws-sdk/clients/cognitoidentityserviceprovider';

const refreshAccessToken = async (token: JWT) => {
  try {
    const params: InitiateAuthRequest = {
      AuthFlow: 'REFRESH_TOKEN',
      ClientId: process.env.COGNITO_CLIENT_ID ?? '',
      AuthParameters: {
        REFRESH_TOKEN: token.refreshToken,
      },
    };
    const res = await cognitoClient.initiateAuth(params).promise();

    return {
      ...token,
      accessToken: res.AuthenticationResult.AccessToken,
      accessTokenExpires: Date.now() + res.AuthenticationResult.ExpiresIn * 1000,
    };
  } catch (error) {
    console.log(error)
    return {
      ...token,
      error: "RefreshAccessTokenError",
    }
  }
};

export default NextAuth ...
```

### 認証トークンを更新する

あとさっき作った認証トークンをリフレッシュする関数を jwt コールバックにいれるだけです。

```pages/api/auth/[...nextauth].ts
export default NextAuth({
  providers: ...,
 callbacks: {
    async jwt({ token, user, account }) {
      // Initial sign in
      if (account && user) {
        return {
          accessToken: account.access_token,
          accessTokenExpires: Date.now() + account.expires_at * 1000,
          refreshToken: account.refresh_token,
          user,
        }
      }

      if(token.accessTokenExpires && Date.now() < token.accessTokenExpires) {
        return token
      }

      return refreshAccessToken(token)
    },
  },
});
```

認証トークンの有効期限が今の時間よりも前であればそのまま token を返し、今の時間よりも認証トークンの有効期限が後の場合は認証トークンをリフレッシュしています。

以上で認証トークンのリフレッシュの実装が完了しました！
認証トークンの有効期限の使い方さえ気をつければ、あとは難なく実装することが出来ると思います。

## 参考

https://next-auth.js.org/tutorials/refresh-token-rotation
https://zenn.dev/yui/articles/b48a74ed717c6c
https://wa3.i-3-i.info/word18474.html

[^1]: UNIX 時間とは「1970 年 1 月 1 日午前 0 時 0 分 0 秒（UTC）」からの経過秒数で表現したものです。
