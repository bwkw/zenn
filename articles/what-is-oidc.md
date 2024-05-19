---
title: "OIDC完全に理解した"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["認証", "認可", "OIDC"]
published: false
---

:::message
本記事では、OAuth 2.0をOAuth、OpenID Connect 1.0をOIDCと表記するため、適宜読み替えていただけますと幸いです。
:::

# TL;DR

# はじめに
こんにちは、レバテック開発部レバテックプラットフォームチームの内藤です。

最近、弊チームでは認証・認可の勉強会を実施しました。
内容としては、ネットの記事や、認証認可の世界ではお馴染みの[OAuth屋](https://twitter.com/authyasan)さんの本を適宜議論を挟みながら読むといったものです。

読んだ本は以下に載せておくので興味ある人は是非お手に取ってみて下さい！（技術書にしてはビビるぐらいに安いです）

https://authya.booth.pm/items/1296585
https://authya.booth.pm/items/1550861
https://authya.booth.pm/items/1877818

今回は、折角なのでOIDCについての理解を整理しておこうと思います。

OAuthについては、弊チームのxxさんが整理してくださっているので、そちらも是非覗いてみて下さい！

# 基礎編
## とは？
2007年に米国で設立された非営利の国際標準化団体[OpenID Foundation(OIDF)](https://openid.net/)の公認団体である[OpenID Foundation Japan](https://www.openid.or.jp/)によれば以下のように説明されています。（[参考](https://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html)）

> OpenID Connect 1.0 は, OAuth 2.0 [RFC6749] プロトコルの上にシンプルなアイデンティティレイヤーを付与したものである. このプロトコルは Client が Authorization Server の認証結果に基づいて End-User のアイデンティティを検証可能にする. また同時に End-User の必要最低限のプロフィール情報を, 相互運用可能かつ RESTful な形で取得することも可能にする.

## OAuthとの違い
## フロー
## 脆弱性
### IDトークンの導入
IDトークンは、ユーザーの認証情報を含む署名付きのJWT（JSON Web Token）で、OIDCではIDトークンによってエンドユーザーの認証を行います。JWTは[RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)で規定されており、セキュリティとデータ整合性を確保するために広く使用されています。

JWTは以下の3つの部分から構成されています。

1. **ヘッダー**
2. **ペイロード**
3. **署名**

#### ヘッダー

ヘッダーには、以下の情報が含まれます。

- **typ（トークンのタイプ）**
    - 例: `"JWT"`
- **alg（署名アルゴリズム）**
    - 例: `"HS256"`（HMAC SHA256）

#### ペイロード

ペイロードには、以下のようなトークンに関する情報が含まれます。

- **iss（トークンを発行した認証サーバーの識別子）**
    - 例: `"https://auth.example.com"`
- **sub（ユーザーの一意の識別子）**
    - 例: `"1234567890"`
- **aud（トークンの受け取り手、通常はクライアントID）**
    - 例: `"my_client_id"`
- **exp（トークンの有効期限）**
    - 例: `1622563200`（Unix時間）
- **iat（トークンの発行時刻）**
    - 例: `1622476800`（Unix時間）

#### 署名

署名は、これまで説明した「ヘッダー」+「.」+「ペイロード」をBase64URLエンコードしたもので、これによりトークンの改ざん防止と発行者の信頼性が保証されます。

これらの内容が実際のJWTに含まれているかは、[[番外編] 実際のサービスでJWTの中身を覗いてみよう]()で検証しています。気になる方はぜひそちらも覗いてみてください！


# 実践編
## ALBでOIDCを使用して、ユーザを認証する方法
## GitHubActionsでOIDCを使用して、AWS認証を行う方法

# おわりに
