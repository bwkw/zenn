---
title: "OIDC を Hono × Bun で完全に理解する"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["認証", "OIDC", "JWT", "Hono", "Bun"]
published: false
publication_name: "levtech"
---

:::message
本記事では、OAuth 2.0 を OAuth、OpenID Connect 1.0 を OIDC と表記するため、適宜読み替えていただけますと幸いです。

:::

# TL;DR

- 弊チームで認証・認可の勉強会を実施
- OIDC の特徴、フロー、脆弱性などを完全に理解した()
- Hono × Bun を利用して、OIDC のリライング・パーティーを実装してみた
- 番外編として、実際の JWT の中身がどうなってるのかを覗いてみた

# はじめに

こんにちは、レバテック開発部 契約/請求ドメインチームの内藤です！

最近弊チームでは、認証認可の世界ではお馴染みの [Auth屋](https://authya.booth.pm/) さんの本やネットの記事を読みつつ、適宜議論を交えた「**認証・認可の勉強会**」を実施しました。

先日、この勉強会に際し、弊チームのかにさんが **OAuth** についての記事を公開しました。
https://zenn.dev/levtech/articles/a6e8910df5baa0

<!-- textlint-disable -->

こちらの記事、**認証・認可の基礎から OAuth** までとても分かりやすく書かれているので、まだ読まれていない方は是非読んでみてください👋

<!-- textlint-enable -->

今回はこの記事に続いて、 **OIDC** について基礎から実践まで整理していくので、少し長くはなりますが最後までお付き合いいただけると幸いです！

# 基礎編

## OIDC とは？

<!-- textlint-disable -->

OIDC は、非営利の国際標準化団体 [OpenID Foundation(OIDF)](https://openid.net/) の公認団体である [OpenID Foundation Japan](https://www.openid.or.jp/) によれば以下のように説明されています。（[参考](https://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html#:~:text=OpenID%20Connect%201.0%20%E3%81%AF%2C%20OAuth%202.0%20%5BRFC6749%5D%20%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB%E3%81%AE%E4%B8%8A%E3%81%AB%E3%82%B7%E3%83%B3%E3%83%97%E3%83%AB%E3%81%AA%E3%82%A2%E3%82%A4%E3%83%87%E3%83%B3%E3%83%86%E3%82%A3%E3%83%86%E3%82%A3%E3%83%AC%E3%82%A4%E3%83%A4%E3%83%BC%E3%82%92%E4%BB%98%E4%B8%8E%E3%81%97%E3%81%9F%E3%82%82%E3%81%AE%E3%81%A7%E3%81%82%E3%82%8B.%20%E3%81%93%E3%81%AE%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB%E3%81%AF%20Client%20%E3%81%8C%20Authorization%20Server%20%E3%81%AE%E8%AA%8D%E8%A8%BC%E7%B5%90%E6%9E%9C%E3%81%AB%E5%9F%BA%E3%81%A5%E3%81%84%E3%81%A6%20End%2DUser%20%E3%81%AE%E3%82%A2%E3%82%A4%E3%83%87%E3%83%B3%E3%83%86%E3%82%A3%E3%83%86%E3%82%A3%E3%82%92%E6%A4%9C%E8%A8%BC%E5%8F%AF%E8%83%BD%E3%81%AB%E3%81%99%E3%82%8B.%20%E3%81%BE%E3%81%9F%E5%90%8C%E6%99%82%E3%81%AB%20End%2DUser%20%E3%81%AE%E5%BF%85%E8%A6%81%E6%9C%80%E4%BD%8E%E9%99%90%E3%81%AE%E3%83%97%E3%83%AD%E3%83%95%E3%82%A3%E3%83%BC%E3%83%AB%E6%83%85%E5%A0%B1%E3%82%92%2C%20%E7%9B%B8%E4%BA%92%E9%81%8B%E7%94%A8%E5%8F%AF%E8%83%BD%E3%81%8B%E3%81%A4%20RESTful%20%E3%81%AA%E5%BD%A2%E3%81%A7%E5%8F%96%E5%BE%97%E3%81%99%E3%82%8B%E3%81%93%E3%81%A8%E3%82%82%E5%8F%AF%E8%83%BD%E3%81%AB%E3%81%99%E3%82%8B.)）

<!-- textlint-enable -->

> OpenID Connect 1.0 は, OAuth 2.0 [RFC6749] プロトコルの上にシンプルなアイデンティティレイヤーを付与したものである. このプロトコルは Client が Authorization Server の認証結果に基づいて End-User のアイデンティティを検証可能にする. また同時に End-User の必要最低限のプロフィール情報を, 相互運用可能かつ RESTful な形で取得することも可能にする.

このように、 OIDC は OAuth をベースとしており、その仕組みを理解するために OAuth についても補足しておくと、 [OpenID Foundation Japan](https://www.openid.or.jp/) によれば以下のように説明されています。（[参考](https://openid-foundation-japan.github.io/rfc6749.ja.html#:~:text=OAuth%202.0%20%E3%81%AF%2C%20%E3%82%B5%E3%83%BC%E3%83%89%E3%83%91%E3%83%BC%E3%83%86%E3%82%A3%E3%83%BC%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AB%E3%82%88%E3%82%8BHTTP%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%81%B8%E3%81%AE%E9%99%90%E5%AE%9A%E7%9A%84%E3%81%AA%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%82%92%E5%8F%AF%E8%83%BD%E3%81%AB%E3%81%99%E3%82%8B%E8%AA%8D%E5%8F%AF%E3%83%95%E3%83%AC%E3%83%BC%E3%83%A0%E3%83%AF%E3%83%BC%E3%82%AF%E3%81%A7%E3%81%82%E3%82%8B.%20%E3%82%B5%E3%83%BC%E3%83%89%E3%83%91%E3%83%BC%E3%83%86%E3%82%A3%E3%83%BC%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%81%AB%E3%82%88%E3%82%8B%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E6%A8%A9%E3%81%AE%E5%8F%96%E5%BE%97%E3%81%AB%E3%81%AF%2C%20%E3%83%AA%E3%82%BD%E3%83%BC%E3%82%B9%E3%82%AA%E3%83%BC%E3%83%8A%E3%83%BC%E3%81%A8HTTP%E3%82%B5%E3%83%BC%E3%83%93%E3%82%B9%E3%81%AE%E9%96%93%E3%81%A7%E5%90%8C%E6%84%8F%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%A9%E3%82%AF%E3%82%B7%E3%83%A7%E3%83%B3%E3%82%92%E4%BC%B4%E3%81%86%E5%A0%B4%E5%90%88%E3%82%82%E3%81%82%E3%82%8B%E3%81%8C%2C%20%E3%82%B5%E3%83%BC%E3%83%89%E3%83%91%E3%83%BC%E3%83%86%E3%82%A3%E3%83%BC%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E8%87%AA%E8%BA%AB%E3%81%8C%E8%87%AA%E3%82%89%E3%81%AE%E6%A8%A9%E9%99%90%E3%81%AB%E3%81%8A%E3%81%84%E3%81%A6%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%82%92%E8%A8%B1%E5%8F%AF%E3%81%99%E3%82%8B%E5%A0%B4%E5%90%88%E3%82%82%E3%81%82%E3%82%8B.)）

> OAuth 2.0 は, サードパーティーアプリケーションによるHTTPサービスへの限定的なアクセスを可能にする認可フレームワークである. サードパーティーアプリケーションによるアクセス権の取得には, リソースオーナーとHTTPサービスの間で同意のためのインタラクションを伴う場合もあるが, サードパーティーアプリケーション自身が自らの権限においてアクセスを許可する場合もある.

つまり、OIDC は OAuth をベースにして、**認可だけでなく認証も行えるようにした拡張仕様**と言えます。

例えば、Google ログイン機能において OIDC を利用します。OIDC を利用することで、Google の認証プロバイダを介して安全にユーザーを認証し、プロフィール情報を取得できます。

## 【少し寄り道】OAuth を認証に利用してはいけないのか？

<!-- textlint-disable -->

話は少し逸れますが、上記の説明を受けて、**「OAuth を認証に利用してはいけないのか？」** と疑問に思われる方がいるかもしれません。

<!-- textlint-enable -->

結論から言うと、**「適切に実装出来るのであれば利用しても良いが、セキュリティリスクの観点から利用しない方が良い」** というのが答えになります。

OAuth には様々な認可フローがありますが、特にセキュリティが問題視されているのは、ブラウザベースのアプリケーションで用いられがちな**インプリシットグラント**です。

まず、インプリシットグラントのフローは以下です。
![OAuth インプリシットグラント](/images/what_is_oidc/oauth_implicit_grant.png)

このフローの問題点は、アクセストークンがブラウザに露出していることです。これにより、以下のようなシナリオが発生します。

1. Aサイト(本物) と Bサイト(偽物) が存在します。
2. Bサイト(偽物) に X さんをアクセスさせ、そこで悪意あるユーザーが X さんのアクセストークンを盗みます。
3. 悪意あるユーザーは、Aサイト(本物) にアクセスし、自身のアクセストークンを X さんのアクセストークンに差し替えることで、X さんとして不正にログインが可能になります。

![OAuth インプリシットグラント 攻撃](/images/what_is_oidc/oauth_implicit_grant_attack.png)

では、認可コードグラントはどうでしょうか？まず、認可コードグラントのフローは以下です。
![OAuth 認可コードグラント](/images/what_is_oidc/oauth_authorization_code_grant.png)

このフローでは、ブラウザに露出しているのは認可コードであるため、悪意あるユーザーが入れ替えることが可能なのは認可コードだけと言うことになります。先程と同様のシナリオを考えると、以下のようになります。

1. Aサイト(本物) と Bサイト(偽物) が存在します。
2. Bサイト(偽物) に X さんをアクセスさせ、そこで悪意あるユーザーが X さんの認可コードを盗みます。
3. 悪意あるユーザーは、Aサイト(本物) にアクセスし、自身の認可コードを X さんの認可コードに差し替えても、認可コードはクライアント(サイト)に紐づいているため、 X さんのアクセストークンを取得することは出来ず、X さんとして不正にログインすることはできません。

![OAuth 認可コードグラント 攻撃](/images/what_is_oidc/oauth_authorization_code_grant_attack.png)

このように、**OAuth 認証の脆弱性が顕著になるのはインプリシットグラントにおいて**であり、認可コードグラントのようなフローではセキュリティ上の問題は少なくなります。しかし、OAuth そのものが認証を目的としたプロトコルではないため、ユーザーのアイデンティティを確認するための標準的なメカニズムを欠いています。その結果、ユーザーの認証情報を正確かつ安全に管理することが難しく、信頼性に欠けてしまいます。（[参考: OAuth 認証とは何か？なぜダメなのか](https://ritou.hatenablog.com/entry/2020/12/01/000000)）

したがって、**認証用途においては OAuth の使用は適切ではない**と言えます。ここで、ユーザー認証を安全かつ標準化された方法で提供するために設計されたプロトコルである OIDC が重要となります。少し話が逸れましたが、ここから本題である OIDC について説明していきます。

## OIDC の登場人物

まず、 OIDC で登場するアクターについて定義しておきます。 OIDC では以下 4 つのアクターが定義されています。

<!-- textlint-disable -->

- **エンドユーザー**
  アプリケーションにアクセスし、コンテンツを利用するユーザーです。エンドユーザーは、認証のために ID プロバイダを介してログインします。
- **リライング・パーティー (Relying Party, RP)**
  エンドユーザーの認証情報を使用して保護されたリソースにアクセスするアプリケーションです。リライング・パーティーは、ID プロバイダからの認証情報を用いてユーザーのアカウント管理やアクセス制御を行います。
- **ID プロバイダ (Identity Provider, IdP)**
  エンドユーザーの認証を行い、その情報をリライング・パーティーに提供するサーバーです。ID プロバイダは、エンドユーザーのアイデンティティを確認し、アクセストークン、ID トークンを発行します。
- **UserInfo エンドポイント**
  エンドユーザーの詳細なプロフィール情報を提供するエンドポイントです。リライング・パーティーはこのエンドポイントを使用して、エンドユーザーに関する追加情報（例えば、名前やメールアドレスなど）を取得し、ユーザープロファイルを構築します。

<!-- textlint-enable -->

## OIDC の特徴

OIDC は、ユーザー認証を目的として設計されており、OAuth に比べユーザー認証に関するセキュリティが強化されています。

OIDC と OAuth の関係を一言で言うと、以下のような式で書くことが出来ます。

```text
OIDC = OAuth + IDトークン + UserInfoエンドポイント
```

### ID トークン

<!-- textlint-disable -->

ID トークンは、ユーザーの認証情報を含む署名付きの JWT（JSON Web Token）で、OIDC では ID トークンによってエンドユーザーの認証を行います。JWT は [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) で規定されており、ヘッダー、ペイロード、署名の 3 つの部分から構成されています。

<!-- textlint-enable -->

- **ヘッダー**
  - **typ（トークンのタイプ）**
    - 例： `"JWT "`
  - **alg（署名アルゴリズム）**
    - 例： `"HS256"`（HMAC SHA256）
  - **kid（キー識別子）**
    - 例： `"123456"`（使用されている署名キーを特定するための識別子）
- **ペイロード**
  - **iss（トークンを発行した認証サーバーの識別子）**
    - 例： `"https://auth.example.com"`
  - **sub（ユーザーの一意の識別子）**
    - 例： `"1234567890"`
  - **aud（トークンの受け取り手、通常はクライアントID）**
    - 例： `"client-id"`
  - **iat（トークンの発行時刻）**
    - 例： `1622476800`（Unix 時間）
  - **exp（トークンの有効期限）**
    - 例： `1622563200`（Unix 時間）
  - **その他の個人情報**
    - 例： `{"name": "Alice Doe", "email": "alice.doe@example.com"}`
- **署名**
  - 「ヘッダー」+「.」+「ペイロード」 を秘密鍵で署名し、Base64URL エンコードしたもの
  - トークンの改竄防止と発行者の信頼性を保証

これらの内容が実際の JWT に含まれているかは、[[番外編] JWTの中身を覗いてみよう](#%E7%95%AA%E5%A4%96%E7%B7%A8%3A-jwt%E3%81%AE%E4%B8%AD%E8%BA%AB%E3%82%92%E8%A6%97%E3%81%84%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86)で検証しています。気になる方はぜひそちらも覗いてみてください！

### UserInfo エンドポイント

OIDC では、リライング・パーティーがユーザーのプロフィール情報や認証情報を取得するために、UserInfo エンドポイントを利用します。OAuth 認証では、プロフィール情報の取得方法は各リソースサーバーに依存していましたが、OIDC では統一された仕様として UserInfo エンドポイントが定義されています。

これにより、アクセストークンのスコープに応じて取得できる情報が規定されており、リライング・パーティーは必要なユーザー情報を一貫して取得できます。

| スコープ  | 説明                                                                                           |
| :-------- | :--------------------------------------------------------------------------------------------- |
| `openid`  | ユーザーの一意識別子 (`sub`) を取得します。基本的な認証情報が含まれます。                      |
| `profile` | ユーザーの基本的なプロフィール情報（例: 名前、プロフィール画像、性別）を取得します。           |
| `email`   | ユーザーのメールアドレスを取得します。メールアドレスが公開されている場合に限ります。           |
| `address` | ユーザーの住所情報を取得します。これには住所、都市、州、郵便番号などが含まれる場合があります。 |
| `phone`   | ユーザーの電話番号を取得します。携帯番号や固定電話番号が含まれることがあります。               |

## OIDC のフロー

OIDC には、以下の 3 つの主要なフローがあります。

- **認可コードフロー**
  - 特徴
    - リライング・パーティーが認可コードを受け取り、それをバックエンドサーバーに渡してアクセストークンと ID トークンを取得する。このフローにより、トークンがクライアント側に露出するリスクが低く、セキュアな実装が可能。
  - ユースケース
    - バックエンドサーバーを持ち、コンフィデンシャルクライアントであるアプリケーション
- **インプリシットフロー**
  - 特徴
    - リライング・パーティーが認可サーバーから直接 ID トークンとアクセストークンを受け取る。このため、認可コードを介するステップがなく、フローがシンプル。ただし、セキュリティリスクが高いため、あまり推奨されない。
  - ユースケース
    - バックエンドサーバーを持たないアプリケーション
    - SPA (Single Page Application) 構成のアプリケーション
- **ハイブリッドフロー**
  - 特徴
    - 認可コードフローとインプリシットフローを組み合わせたハイブリッドなフロー
  - ユースケース
    - バックエンドサーバーを持ち、パブリッククライアントとコンフィデンシャルクライアントの両方で構成されるアプリケーション

これらのフローの中で、最もユースケースが多く、広く利用されている「**認可コードフロー**」について、今回は **Google ログイン**を例にとって詳しく説明していきます。

![OIDC 認可コードフロー](/images/what_is_oidc/oidc_authorization_code_flow.png)

### 認可コードの取得

1. **OIDC 開始**
   エンドユーザーが「Google アカウントでログイン」ボタンを押下することにより、OIDC が開始されます。 リライング・パーティーはこのリクエストを受け付けると、 HTTP 302 リダイレクトを用いて、ID プロバイダが提供する**認可エンドポイント**へエンドユーザーをリダイレクトさせます。これを**認証リクエスト**と呼び、以下のようなクエリパラメータが付加されます。

   ```http
   GET /authorize?
       response_type=code
       &client_id=YOUR_CLIENT_ID
       &redirect_uri=https://www.example.com/callback
       &scope=openid email profile

   HTTP/1.1
   Host: accounts.google.com
   ```

   | パラメータ      | 説明                                                       |
   | --------------- | ---------------------------------------------------------- |
   | `response_type` | ここでは認可コード（`code`）を要求していることを示します。 |
   | `client_id`     | OIDC プロバイダから発行されるアプリケーションの識別子。    |
   | `redirect_uri`  | 認証成功後にユーザーがリダイレクトされるURI。              |
   | `scope`         | リクエストするアクセス権限の範囲。                         |

   ::: details 【コラム】response_type による認証フローの切り替え
   認証リクエストに含まれる `response_type` の値によって、使用される認証フローと認証レスポンスに含まれるトークンが変化します。 `response_type` と認証フローの対応関係は以下です。

   | response_type             | 認証フロー           |
   | ------------------------- | -------------------- |
   | `code`                    | 認可コードフロー     |
   | `id_token`                | インプリシットフロー |
   | `id_token` `token`        | インプリシットフロー |
   | `code` `id_token`         | ハイブリッドフロー   |
   | `code` `token`            | ハイブリッドフロー   |
   | `code` `id_token` `token` | ハイブリッドフロー   |

   :::

<!-- textlint-disable -->

:::message
本来、state パラメータも付与されていますが、ここでは省略しています。
state パラメータについては、[OIDC の脆弱性対策](#oidc-%E3%81%AE%E8%84%86%E5%BC%B1%E6%80%A7%E5%AF%BE%E7%AD%96)を参照してください。
:::

2. **ユーザー認証**
エンドユーザーがリダイレクトされた後、ID プロバイダの認証エンドポイントでユーザー認証を行います。ユーザーは自身の認証情報（例：ユーザー名とパスワード、または多要素認証）を入力し認証が成功すると、ID プロバイダはユーザーに対して承認画面を表示し、リライング・パーティーが要求している情報とアクセス権について確認します。
<!-- textlint-enable -->

3. **ユーザー情報提供に同意**
   ユーザーが承認画面でリライング・パーティーの要求に同意すると、ID プロバイダは認可コードを生成し、ユーザーを指定された `redirect_uri` にリダイレクトします。このリダイレクト先の URI には、以下のようなクエリパラメータが付加されます。

   ```http
   HTTP/1.1 302 Found
   Location: https://www.example.com/callback?code=ABCDEFGHIJKLMNOPQRSTUVWXYZ
   ```

   | パラメータ | 説明                                                                                        |
   | ---------- | ------------------------------------------------------------------------------------------- |
   | `code`     | 認可コード。リライング・パーティーはこれを用いてアクセストークン、ID トークンを取得します。 |

### アクセストークンと ID トークンの取得

認可コードを取得したリライング・パーティーは、ID プロバイダの**トークンエンドポイント**にリクエストを送り、アクセストークンと ID トークンを取得します。これを**トークンリクエスト**と呼び、以下のようなクエリパラメータが付加されます。

```http
POST /token HTTP/1.1
    Host: accounts.google.com
    Content-Type: application/x-www-form-urlencoded

    grant_type=authorization_code
    &code=ABCDEFGHIJKLMNOPQRSTUVWXYZ
    &redirect_uri=https://www.example.com/callback
    &client_id=YOUR_CLIENT_ID
    &client_secret=YOUR_CLIENT_SECRET
```

<!-- textlint-disable -->

ID プロバイダは認可コードの検証を行い、検証が成功するとアクセストークンと ID トークンを生成します。**トークンレスポンス**は以下の形式で返されます。

<!-- textlint-enable -->

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "access_token": "YOUR_ACCESS_TOKEN",
  "expires_in": 3600,
  "id_token": "YOUR_ID_TOKEN",
  "token_type": "Bearer",
  "refresh_token": "YOUR_REFRESH_TOKEN"
}
```

| パラメータ      | 説明                                                            |
| --------------- | --------------------------------------------------------------- |
| `access_token`  | リソースサーバーへのアクセスに使用されるトークン。              |
| `expires_in`    | アクセストークンの有効期限（秒単位）。                          |
| `id_token`      | 認証されたユーザーに関する情報を含む JSON Web トークン（JWT）。 |
| `token_type`    | トークンのタイプ（通常は "Bearer"）。                           |
| `refresh_token` | アクセストークンの更新に使用されるトークン。                    |

### ID トークンの検証

ID トークンは JWT 形式で提供され、その正当性を確認するために以下の検証が行われます。

- **署名の検証**
  - トークンが改竄されていないことを確認します。
  - 署名は、ID プロバイダがホストする JWK（JSON Web Key）エンドポイントが提供する公開鍵を使用して検証します。
- **発行者(iss)の確認**
  - トークンの iss クレームが期待される発行者と一致しているかを確認します。
- **受信者(aud)の確認**
  - トークンの aud クレームがリライング・パーティーのクライアント ID と一致しているかを確認します。
- **有効期限(exp)の確認**
  - トークンの exp クレームが現在時刻よりも未来であることを確認し、トークンの期限が切れていないことを確認します。

### プロフィールへのアクセス

リライング・パーティーは、アクセストークンを用いて **UserInfo エンドポイント**にリクエストを送り、ユーザーのプロフィール情報を取得します。この情報には、ユーザーの基本情報（名前、メールアドレス、プロフィール写真など）が含まれます。**UserInfo レスポンス**は以下の形式で返されます。

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sub": "1234567890",
  "name": "Alice Doe",
  "email": "alice.doe@example.com",
  "picture": "https://example.com/alicedoe.jpg"
}
```

## OIDC の脆弱性対策

上記の OIDC の認可コードフローにおいて、**CSRF 攻撃**の脆弱性が存在する可能性があります。

<!-- textlint-disable -->

CSRF 攻撃は、悪意ある第三者がユーザーのブラウザを介して、意図しないリクエストを送信させることで、不正に操作を実行させる攻撃手法です。この攻撃は、ユーザーが意図しない操作を実行してしまうため、認可フローにおいても重大なセキュリティリスクとなります。

<!-- textlint-enable -->

ここで重要となってくるのが、 state パラメータです。[RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.1) にも、state パラメータを使用することで、CSRF 攻撃を防止するように記載されています。

> RECOMMENDED. An opaque value used by the client to maintain state between the request and callback. The authorization server includes this value when redirecting the user-agent back to the client. The parameter SHOULD be used for preventing cross-site request forgery as described in Section 10.12.

まず、CSRF 攻撃が成功するケースを示します。（※ 以降では説明の簡略化のため、 UserInfo エンドポイントを省略します。）

![OIDC CSRF 攻撃成功](/images/what_is_oidc/oidc_csrf_attack.png)

上図では、ユーザーが攻撃者のアクセストークンを利用していることに気付かず、攻撃者の Google Drive にプライベートファイルを誤ってアップロードしてしまっています。

次に、state パラメータを使用することで CSRF 攻撃を防止する方法を説明します。

state パラメータは、認証リクエスト時にリライング・パーティーで生成するランダムな文字列で、認可サーバーに送信されます。認可サーバーは、認可コードと共に state パラメータをレスポンスとして返し、リライング・パーティーはこの state が、認可リクエストを送信した際にセッションに保存されたものと一致するか（= state がセッションに保存された値と一致するか）を確認します。

![OIDC CSRF 攻撃失敗](/images/what_is_oidc/oidc_csrf_attack_state.png)

:::details 【コラム】認可コードフローにおいて、リプレイ攻撃は脆弱性にならないのか？
インプリシットフローやハイブリッドフローにおいては、 nonce パラメータはリプレイ攻撃に対する対策として使用されます。これらのフローでは、ID トークンが直接クライアントに返されるため、リプレイ攻撃のリスクが存在します。リプレイ攻撃とは、攻撃者が有効なトークンを傍受し、後に再利用することで正当なユーザーとして不正に認証を受ける攻撃です。このリスクを軽減するために、nonce パラメータが利用されます。

一方、認可コードフローでは、ブラウザ経由でクライアントに直接送信されるのは認可コードのみです。そして、認可コードは [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2) に記載されている通り、一度しか使用できないため、認可コード自体がリプレイ攻撃に対する防御策となっています。

> The authorization code MUST expire shortly after it is issued to mitigate the risk of leaks. A maximum authorization code lifetime of 10 minutes is RECOMMENDED. The client MUST NOT use the authorization code more than once. If an authorization code is used more than once, the authorization server MUST deny the request and SHOULD revoke (when possible) all tokens previously issued based on that authorization code. The authorization code is bound to the client identifier and redirection URI.

<!-- textlint-disable -->

この意味で、認可コードフローにおいて、リプレイ攻撃は脆弱性ではないと言えるかと思います。

<!-- textlint-enable -->

:::

# 実践編: Hono × Bun × Google 認証 を使用して、ユーザーを認証する

## Google Cloud の設定

1. **Google Cloud プロジェクトの作成**
   まずは、 Google Cloud のプロジェクトを作成します。[こちら](https://console.cloud.google.com/projectcreate)でプロジェクト名を入力し、プロジェクトを作成します。

   ![Google Cloud Project](/images/what_is_oidc/gc_create_new_project.png)

2. **OAuth 同意画面の作成**
   Google 認証を利用するためには、OAuth 2.0 の同意画面を設定する必要があります。この画面は、ユーザーがアプリケーションにアクセスする際に、どの情報が共有されるかを確認できる場所です。以下の手順で作成します。

   1. **User Type の選択**
      ![Google OAuth Consent Screen User Type](/images/what_is_oidc/gc_oauth_consent_screen_user_type.png)

   2. **アプリケーションの情報を入力**
      ![Google OAuth Consent Screen Application](/images/what_is_oidc/gc_oauth_consent_screen_application.png)
      ![Google OAuth Consent Screen Developer](/images/what_is_oidc/gc_oauth_consent_screen_developer.png)

   3. **スコープの設定**
      ![Google OAuth Consent Screen Scope](/images/what_is_oidc/gc_oauth_consent_screen_scope.png)

   4. **テストユーザーの追加**
      ![Google OAuth Consent Screen Test User](/images/what_is_oidc/gc_oauth_consent_screen_test_user.png)

3. **認証情報の作成**
   Google 認証をアプリケーションに組み込むためには、OAuth 2.0 クライアント ID とクライアントシークレットを取得する必要があります。これらの情報は、アプリケーションがユーザーを認証し、Google API にアクセスする際に使用されます。以下の手順で取得します。

   1. **OAuth クライアント ID の選択**
      ![Google OAuth Consent Screen Test User](/images/what_is_oidc/gc_authentication_information.png)

   2. **OAuth クライアント ID の作成**
      ![Google OAuth Consent Screen Test User](/images/what_is_oidc/gc_authentication_information_oauth_client_id.png)

<!-- textlint-disable -->

以上の設定で、 Client ID と Client Secret が作成できたと思いますが、この後のリライング・パーティーの実装で必要になるので、捨てずに保存しておいてください。

<!-- textlint-enable -->

## リライング・パーティーの実装

今回、リライング・パーティーの実装は、**Hono** × **Bun** を使用します。

- [Hono](https://hono.dev/): 高速で軽量なウェブフレームワーク
- [Bun](https://bun.sh/): JavaScript Runtime、超高速でスクリプトの実行が可能

:::message
以降では、 Bun がホストマシンにインストールされていて、bun コマンドが実行できることを前提として進めます。

:::

今回は、再現を容易にするため、リライング・パーティーを実装したリポジトリを用意しました！実際の挙動を再現したい方は、是非以下のリポジトリを手元にクローンしていただき、README の手順通りに実行してみてください👋
https://github.com/bwkw/hono-bun-oidc

実装イメージが湧くように、念の為以下にもコードを記載します。

```typescript
import { Hono } from "hono";
import { jwtVerify, createRemoteJWKSet } from "jose";

// グローバル定数
const AUTH_ENDPOINT = "https://accounts.google.com/o/oauth2/v2/auth";
const TOKEN_ENDPOINT = "https://www.googleapis.com/oauth2/v4/token";
const USERINFO_ENDPOINT = "https://www.googleapis.com/oauth2/v3/userinfo";
const JWKS_URI = "https://www.googleapis.com/oauth2/v3/certs";
const REDIRECT_URI = "http://127.0.0.1:3333/callback";
const CLIENT_ID = "YOUR_CLIENT_ID";
const CLIENT_SECRET = "YOUR_CLIENT_SECRET";

const app = new Hono();

// 認証用ページ
app.get("/auth", (c) => {
  // 認証リクエストを構築
  const responseType = "code";
  const scope = encodeURIComponent(
    "openid https://www.googleapis.com/auth/userinfo.email https://www.googleapis.com/auth/userinfo.profile"
  );
  const authUrl = `${AUTH_ENDPOINT}?response_type=${responseType}&client_id=${encodeURIComponent(
    CLIENT_ID
  )}&redirect_uri=${encodeURIComponent(REDIRECT_URI)}&scope=${scope}`;

  console.log(`Redirecting to: ${authUrl}`);
  return c.redirect(authUrl); // 認可エンドポイントにリダイレクトして、ユーザー認証を開始
});

// コールバック用
app.get("/callback", async (c) => {
  // 認可コードを受け取る
  const url = new URL(c.req.url);
  const code = url.searchParams.get("code");

  if (!code) {
    return c.text("Authorization code not found", 400);
  }

  // 認可コードを使用してトークンをリクエスト
  const tokenResponse = await fetch(TOKEN_ENDPOINT, {
    method: "POST",
    headers: {
      "Content-Type": "application/x-www-form-urlencoded",
    },
    body: new URLSearchParams({
      code: code,
      client_id: CLIENT_ID,
      client_secret: CLIENT_SECRET,
      redirect_uri: REDIRECT_URI,
      grant_type: "authorization_code", // 認可コードを使ってトークンをリクエストするための指定
    }),
  });

  // トークンレスポンスを確認
  const tokenData = await tokenResponse.json();
  console.log("Token response:", tokenData);

  // ID トークンの存在確認
  if (!tokenData.id_token) {
    return c.text("Failed to obtain ID token", 400);
  }

  try {
    // JWKS URI を使ってリモートからキーセットを取得
    const JWKS = createRemoteJWKSet(new URL(JWKS_URI));

    // ID トークンを検証
    const idToken = tokenData.id_token;
    const { payload } = await jwtVerify(idToken, JWKS, {
      issuer: "https://accounts.google.com", // 発行者の確認
      audience: CLIENT_ID, // クライアント ID の確認
    });
    console.log("ID Token verified successfully:", payload); // トークンのペイロードを確認
  } catch (error) {
    console.error("ID Token verification failed:", error);
    return c.text("ID Token verification failed", 400); // 検証が失敗した場合
  }

  // UserInfo エンドポイントから追加情報を取得
  const userInfoResponse = await fetch(USERINFO_ENDPOINT, {
    method: "GET",
    headers: {
      Authorization: `Bearer ${tokenData.access_token}`, // アクセストークンを使用してリクエスト
    },
  });

  const userInfo = await userInfoResponse.json();

  return c.json(userInfo); // ユーザー情報を返す
});

export default {
  port: 3333,
  fetch: app.fetch,
};

console.log("Open http://127.0.0.1:3333/auth");
```

## 動作確認

1. **http://127.0.0.1:3333/auth にアクセスし、認可エンドポイントにリダイレクトして、ユーザー認証を開始**

   ![Operation Check Choose Account](/images/what_is_oidc/operation_check_choose_account.png)

2. **ユーザー情報提供に同意**

   ![Operation Check Consent](/images/what_is_oidc/operation_check_consent.png)

3. **プロフィール情報を表示**

   ![Operation Check Profile](/images/what_is_oidc/operation_check_profile.png)

# 番外編: JWTの中身を覗いてみよう

JWT の中身を確認するには、以下のサイトが利用できます。

https://jwt.io/

<!-- textlint-disable -->

このサイトに、 JWT を貼り付けると、ヘッダー、ペイロード、署名の各部分が自動的にデコードされ、その内容を簡単に確認できます。また、このサイトは JWT のヘッダーを解析し、使用されている署名アルゴリズムを読み取ります。そして、そのアルゴリズムに基づいて署名の検証を行うことも可能です。

<!-- textlint-enable -->

今回の実装で取得した ID トークンをこのサイトに入力すると、以下のような出力結果が得られます。（ ※ 一部の情報はぼかしています）

![JWT](/images/what_is_oidc/jwt.png)

JWT の中身の確認にはとても便利なサイトなので、皆さんも是非使ってみてください！

# おわりに

今回、チームでの認証・認可の勉強会に際し、 OIDC 周りの知識を整理してみました！

<!-- textlint-disable -->

実は、直近弊チームでは認証・認可のリプレースを予定しており、それに伴って勉強会を開きましたが、改めて学習してみると多くの点で理解が不十分であることを痛感しました...

認証・認可はほぼ全ての Web アプリケーションにおいて必須の知識ではありながらも、意外と軽視されがちなものかと思います。以下の参考文献にも載せましたが、[Auth屋](https://authya.booth.pm/) さんの本は認証・認可を学習する上で非常にオススメなので、興味が湧いた方は是非手に取ってみてください！

<!-- textlint-enable -->

# 参考文献

https://authya.booth.pm/items/1296585
https://authya.booth.pm/items/1550861
https://authya.booth.pm/items/1877818
https://www.sakimura.org/2012/02/1487/
https://qiita.com/TakahikoKawasaki/items/4ee9b55db9f7ef352b47
