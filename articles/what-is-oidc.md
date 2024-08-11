---
title: "OIDC完全に理解した"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["認証", "認可", "OIDC"]
published: false
---

:::message
本記事では、OAuth 2.0 を OAuth、OpenID Connect 1.0 を OIDC と表記するため、適宜読み替えていただけますと幸いです。

:::

# TL;DR

# はじめに

こんにちは、レバテック開発部レバテックプラットフォーム開発チームの内藤です！

最近弊チームでは、認証認可の世界ではお馴染みの [Auth屋](https://authya.booth.pm/) さんの本やネットの記事を読みつつ、適宜議論を交えた「**認証・認可の勉強会**」を実施しました。

先日、この勉強会に際し、弊チームのかにさんが **OAuth** についての記事を公開しました。
https://zenn.dev/levtech/articles/a6e8910df5baa0

<!-- textlint-disable -->

こちらの記事、**認証・認可の基礎から OAuth** までとても分かりやすく書かれているので、まだ読まれていない方は是非読んでみてください👋

<!-- textlint-enable -->

今回はこの記事に続いて、 **OIDC** について基礎から実践まで整理していくので、最後までお付き合いいただけると幸いです。

# 基礎編

## OIDC とは？

<!-- textlint-disable -->

2007 年に米国で設立された非営利の国際標準化団体 [OpenID Foundation(OIDF)](https://openid.net/) の公認団体である [OpenID Foundation Japan](https://www.openid.or.jp/) によれば以下のように説明されています。（[参考](https://openid-foundation-japan.github.io/openid-connect-core-1_0.ja.html)）

<!-- textlint-enable -->

> OpenID Connect 1.0 は, OAuth 2.0 [RFC6749] プロトコルの上にシンプルなアイデンティティレイヤーを付与したものである. このプロトコルは Client が Authorization Server の認証結果に基づいて End-User のアイデンティティを検証可能にする. また同時に End-User の必要最低限のプロフィール情報を, 相互運用可能かつ RESTful な形で取得することも可能にする.

このように、 OIDC は OAuth をベースとしており、その仕組みを理解するために OAuth についても補足しておくと、 [OpenID Foundation Japan](https://www.openid.or.jp/) によれば以下のように説明されています。（[参考](https://openid-foundation-japan.github.io/rfc6749.ja.html)）

> OAuth 2.0 は, サードパーティーアプリケーションによるHTTPサービスへの限定的なアクセスを可能にする認可フレームワークである. サードパーティーアプリケーションによるアクセス権の取得には, リソースオーナーとHTTPサービスの間で同意のためのインタラクションを伴う場合もあるが, サードパーティーアプリケーション自身が自らの権限においてアクセスを許可する場合もある.

つまり、OIDC は OAuth をベースにして、**認可だけでなく認証も行えるようにした拡張仕様**と言えます。

例えば、Google ログイン機能において OIDC を利用します。OIDC を利用することで、Google の認証プロバイダを介して安全にユーザーを認証し、プロフィール情報を取得できます。

## 【少し寄り道】OAuth は認証に使ってはいけない？

<!-- textlint-disable -->

話は少し逸れますが、上記の説明を受けて、「**OAuth を認証に使ってはいけないのか？**」と疑問に思われる方がいるかもしれません。

<!-- textlint-enable -->

実際、多くの記事で OAuth は認証用途に使うべきではないという意見が散見されます。
しかし正確には「**一部の OAuth 認可フローを認証に使用するとセキュリティホールが発生する可能性があるため、認証には適さない**」というのが正しい理解になります。

OAuth には様々な認可フローがありますが、セキュリティが問題視されているのは、ブラウザベースのアプリケーションで用いられがちな**インプリシットグラント**です。

まず、インプリシットグラントのフローは以下です。
![OAuth インプリシットグラント](/images/what_is_oidc/oauth_implicit_grant.png)

このフローの問題点は、アクセストークンがブラウザに露出していることです。これにより、悪意あるユーザーが他クライアントでユーザー X さんのアクセストークンを取得した場合、このクライアント上で X さんとして不正にログインできてしまうリスクがあります。
![OAuth インプリシットグラント 攻撃](/images/what_is_oidc/oauth_implicit_grant_attack.png)

では、認可コードグラントはどうでしょうか？まず、認可コードグラントのフローは以下です。
![OAuth 認可コードグラント](/images/what_is_oidc/oauth_authorization_code_grant.png)

このフローでは、ブラウザに露出しているのは認可コードであるため、悪意あるユーザーが入れ替えることが可能なのは認可コードだけと言うことになります。仮に、悪意あるユーザーが他クライアントでユーザー X さんの認可コードを取得出来たとしても、認可コードはクライアントに紐づいているため、以下のようにアクセストークンの取得に失敗し、ログインすることは出来ません。
![OAuth 認可コードグラント 攻撃](/images/what_is_oidc/oauth_authorization_code_grant_attack.png)

このように、OAuth 認証の脆弱性が顕著になるのはインプリシットグラントにおいてであり、認可コードグラントのようなフローではセキュリティ上の問題は少なくなります。しかし、OAuth そのものが認証を目的としたプロトコルではないため、ユーザーのアイデンティティを確認するための標準的なメカニズムを欠いています。その結果、ユーザーの認証情報を正確かつ安全に管理することが難しく、信頼性に欠ける場合があります。例えば、OAuth のアクセストークンは認可のためのものであり、認証を目的としていないため、アイデンティティの確認には適していません。

したがって、認証用途においては OAuth の使用は適切ではないと言えます。ここで、ユーザー認証を安全かつ標準化された方法で提供するために設計されたプロトコルである OIDC が重要となります。少し話が逸れましたが、ここから本題である OIDC について説明していきます。

## OIDC の登場人物

まず、 OIDC で登場するアクターについて定義しておきます。 OIDC では以下 4 つのアクターが定義されています。

<!-- textlint-disable -->

- **エンドユーザー**
  ブログアプリケーションにアクセスし、コンテンツを作成・閲覧しようとするユーザーです。エンドユーザーは、ブログアプリケーションを使用するために Google や Facebook などの ID プロバイダを介して認証を行います。
- **リライング・パーティー (Relying Party, RP)**
  エンドユーザーの認証情報を用いて、保護されたリソースにアクセスしようとするアプリケーションです。この場合、シンプルなブログアプリケーション自体がリライング・パーティーに該当します。ブログアプリケーションは、ID プロバイダからエンドユーザーの認証情報を取得し、その情報を基にユーザーのアカウント管理やアクセス制御を行います。
- **ID プロバイダ (Identity Provider, IdP)**
  エンドユーザーの認証を行い、その情報をリライング・パーティーに提供するサーバーです。この場合、Google や Facebook などの認証プロバイダが ID プロバイダに該当します。ID プロバイダは、エンドユーザーのアイデンティティを確認し、認証トークンを発行します。
- **UserInfo エンドポイント**
  エンドユーザーの詳細なプロフィール情報を提供するエンドポイントです。ブログアプリケーションの例では、Google や Facebook の UserInfo エンドポイントが該当します。ブログアプリケーションはこのエンドポイントを使用して、エンドユーザーに関する追加情報（例えば、名前やメールアドレスなど）を取得し、ユーザープロファイルを構築します。

<!-- textlint-enable -->

## OIDC の特徴

OIDC は、ユーザー認証を目的として設計されており、OAuth に比べユーザー認証に関するセキュリティが強化されています。

OIDC と OAuth の関係を一言で言うと、以下のような式で書くことが出来ます。

```text
OIDC = OAuth + IDトークン + UserInfoエンドポイント
```

**ID トークン**と **UserInfo エンドポイント**について、それぞれ説明していきます。

### ID トークン

<!-- textlint-disable -->

ID トークンは、ユーザーの認証情報を含む署名付きの JWT（JSON Web Token）で、OIDC では ID トークンによってエンドユーザーの認証を行います。JWT は [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) で規定されており、セキュリティとデータ整合性を確保するために広く使用されています。

<!-- textlint-enable -->

JWT は以下の 3 つの部分から構成されています。

1. **ヘッダー**
2. **ペイロード**
3. **署名**

#### ヘッダー

ヘッダーには、以下の情報が含まれます。

- **typ（トークンのタイプ）**
  - 例： `"JWT"`
- **alg（署名アルゴリズム）**
  - 例： `"HS256"`（HMAC SHA256）

#### ペイロード

ペイロードには、以下のようなトークンに関する情報が含まれます。

- **iss（トークンを発行した認証サーバーの識別子）**
  - 例： `"https://auth.example.com"`
- **sub（ユーザーの一意の識別子）**
  - 例： `"1234567890"`
- **aud（トークンの受け取り手、通常はクライアントID）**
  - 例： `"my_client_id"`
- **exp（トークンの有効期限）**
  - 例： `1622563200`（Unix 時間）
- **iat（トークンの発行時刻）**
  - 例： `1622476800`（Unix 時間）

#### 署名

署名は、これまで説明した「ヘッダー」+「.」+「ペイロード」を Base64URL エンコードしたもので、これによりトークンの改ざん防止と発行者の信頼性が保証されます。

これらの内容が実際の JWT に含まれているかは、[[番外編] JWTの中身を覗いてみよう]()で検証しています。気になる方はぜひそちらも覗いてみてください！

### UserInfo エンドポイント

OIDC において、クライアントアプリケーションはユーザーのプロフィール情報や認証情報を取得するために、UserInfo エンドポイントにアクセスします。

## OIDC のフロー

OIDC には、以下の 3 つの主要なフローがあります。

- **認可コードフロー**
  - 特徴
    - クライアントが認可コードを受け取り、それをバックエンドサーバーに渡してアクセストークンと ID トークンを取得するので、トークンが流出するリスクが低く、セキュア
  - ユースケース
    - バックエンドサーバーを持ち、コンフィデンシャルクライアントであるアプリケーション
- **インプリシットフロー**
  - 特徴
    - クライアントは認可サーバーから直接 ID トークンとアクセストークンを受け取るため、認可コードのステップがなく、シンプル
  - ユースケース
    - バックエンドサーバーを持たないアプリケーション
    - SPA 構成のアプリケーション
- **ハイブリッドフロー**
  - 特徴
    - 認可コードフローとインプリシットフローのハイブリッドなフロー
  - ユースケース
    - バックエンドサーバーを持ち、パブリッククライアントとコンフィデンシャルクライアントの両方で構成されるアプリケーション

これらのフローの内、最もユースケースが多く、広く利用されている「**認可コードフロー認可コードフロー**」について、もう少し詳しく説明していきます。
![OIDC 認可コードフロー](/images/what_is_oidc/oidc_authorization_code_flow.png)

### 認可コードの取得

1. **OIDC 開始**
   エンドユーザーが「Google アカウントでログイン」ボタンを押下することにより、OIDC が開始されます。 リライング・パーティーはこのリクエストを受け付けると、 HTTP 302 リダイレクトを用いて、ID プロバイダが提供する**認可エンドポイント**へエンドユーザーをリダイレクトさせます。これを**認証リクエスト**と呼び、以下のようなクエリパラメータが付加されます。

   ```text
   GET /authorize?
       response_type=code
       &client_id=YOUR_CLIENT_ID
       &redirect_uri=YOUR_REDIRECT_URL
       &scope=openid email profile

   HTTP/1.1
   Host: accounts.google.com
   ```

   | パラメータ    | 説明                                                                                                      |
   | ------------- | --------------------------------------------------------------------------------------------------------- |
   | response_type | ここでは認可コード（`code`）を要求していることを示します。                                                |
   | client_id     | OIDC プロバイダから発行されるアプリケーションの識別子。                                                   |
   | redirect_uri  | 認証成功後にユーザーがリダイレクトされるURI。                                                             |
   | scope         | 要求するスコープ。OIDC の `openid`、`email`、`profile` に加え、その他のリソースへのアクセス権を指定する。 |

   ::: details response_type によるフローの切り替え
   認証リクエストに含まれる `response_type` の値によって、使用される認証フローと認証レスポンスに含まれるトークンが変化します。 `response_type` とフローの対応関係は以下です。

   | response_type       | フロー               |
   | ------------------- | -------------------- |
   | code                | 認可コードフロー     |
   | id_token            | インプリシットフロー |
   | id_token token      | インプリシットフロー |
   | code id_token       | ハイブリッドフロー   |
   | code token          | ハイブリッドフロー   |
   | code id_token token | ハイブリッドフロー   |

   :::

<!-- textlint-disable -->

2. **ユーザー認証**
エンドユーザーがリダイレクトされた後、ID プロバイダの認証エンドポイントでユーザー認証を行います。ユーザーは自身の認証情報（例：ユーザー名とパスワード、または多要素認証）を入力し認証が成功すると、ID プロバイダはユーザーに対して承認画面を表示し、リライング・パーティーが要求している情報とアクセス権について確認します。
<!-- textlint-enable -->

3. **ユーザー情報提供に同意**
   ユーザーが承認画面でリライング・パーティーの要求に同意すると、ID プロバイダは認可コードを生成し、ユーザーを指定された `redirect_uri` にリダイレクトします。このリダイレクト先の URI には、以下のようなクエリパラメータが付加されます。

   ```
   HTTP/1.1 302 Found
   Location: YOUR_REDIRECT_URL?code=AUTHORIZATION_CODE&state=XXXXXXXXXXXXXXXXXXX
   ```

   | パラメータ | 説明                                                                                      |
   | ---------- | ----------------------------------------------------------------------------------------- |
   | code       | 認可コード。リライング・パーティーはこれを用いてアクセストークン、ID トークンを取得する。 |
   | state      | リクエスト時に送信された state パラメータの値。CSRF 対策のために利用される。              |

### アクセストークンと ID トークンの取得

認可コードを取得したリライング・パーティーは、ID プロバイダのトークンエンドポイントにリクエストを送り、アクセストークンと ID トークンを取得します。これを**トークンリクエスト**と呼び、以下のようなクエリパラメータが付加されます。

```text
POST /token HTTP/1.1
    Authorization: Basic base64_encode(CLIENT_ID:CLIENT_SECRET)
    Content-Type: application/json
    Host: accounts.google.com

grant_type: authorization_code
&code=ZZZZZZZZZZZZZZZZZZZ
&redirect_uri=YOUR_REDIRECT_URL
```

<!-- textlint-disable -->

ID プロバイダは認可コードの検証を行い、検証が成功するとアクセストークンと ID トークンを生成します。**トークンレスポンス**は以下の形式で返されます。

<!-- textlint-enable -->

```text
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

| パラメータ    | 説明                                                            |
| ------------- | --------------------------------------------------------------- |
| access_token  | リソースサーバーへのアクセスに使用されるトークン。              |
| expires_in    | アクセストークンの有効期限（秒単位）。                          |
| id_token      | 認証されたユーザーに関する情報を含む JSON Web トークン（JWT）。 |
| token_type    | トークンのタイプ（通常は "Bearer"）。                           |
| refresh_token | アクセストークンの更新に使用されるトークン。                    |

### ID トークンの検証

ID トークンは JWT 形式で提供され、署名によりトークンの改ざんを防ぎます。署名検証には、ID プロバイダが提供する公開鍵を使用します。公開鍵は JWK（JSON Web Key）エンドポイントから取得可能です。さらに、ID トークンに含まれるクレームを検証し、トークンの正当性を確認します。

- iss が期待する値と一致するか
- aud が リライング・パーティーの client_id と一致するか
- exp が現在時刻よりも未来であるか

### プロフィールへのアクセス

リライングパーティーは、アクセストークンを用いて UserInfo エンドポイントにリクエストを送り、ユーザーのプロフィール情報を取得します。この情報には、ユーザーの基本情報（名前、メールアドレス、プロフィール写真など）が含まれます。UserInfo レスポンスは以下の形式で返されます。

```text
HTTP/1.1 200 OK
Content-Type: application/json

{
  "sub": "1234567890",
  "name": "John Doe",
  "email": "john.doe@example.com",
  "picture": "https://example.com/johndoe.jpg"
}
```

:::message
上記の sub クレームが ID トークンと同じであることを確認しなければなりません。

:::

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

state パラメータは、認証リクエスト時にリライング・パーティーで生成するランダムな文字列で、認可サーバーに送信されます。認可サーバーは、認可コードと共に state パラメータをレスポンスとして返し、リライングパーティーはこの state が、認可リクエストを送信した際にセッションに保存されたものと一致するか（= state がセッションに保存された値と一致するか）を確認します。

![OIDC CSRF 攻撃失敗](/images/what_is_oidc/oidc_csrf_attack_state.png)

:::details 【コラム】認可コードフローにおいて、リプレイ攻撃は脆弱性にならないのか？
インプリシットフローやハイブリッドフローにおいては、 nonce パラメータはリプレイ攻撃に対する対策として使用されます。これらのフローでは、ID トークンが直接クライアントに返されるため、リプレイ攻撃のリスクが存在します。リプレイ攻撃とは、攻撃者が有効なトークンを傍受し、後に再利用することで正当なユーザーとして不正に認証を受ける攻撃です。このリスクを軽減するために、nonce パラメータが利用されます。

一方、認可コードフローでは、ブラウザ経由でクライアントに直接送信されるのは認可コードのみです。そして、認可コードは [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1.2) に記載されている通り、一度しか使用できないため、認可コード自体がリプレイ攻撃に対する防御策となっています。

> The authorization code MUST expire shortly after it is issued to mitigate the risk of leaks. A maximum authorization code lifetime of 10 minutes is RECOMMENDED. The client MUST NOT use the authorization code more than once. If an authorization code is used more than once, the authorization server MUST deny the request and SHOULD revoke (when possible) all tokens previously issued based on that authorization code. The authorization code is bound to the client identifier and redirection URI.

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

   1. **OAuth Client ID の選択**
      ![Google OAuth Consent Screen Test User](/images/what_is_oidc/gc_authentication_information.png)

   2. **OAuth Client ID の作成**
      ![Google OAuth Consent Screen Test User](/images/what_is_oidc/gc_authentication_information_oauth_client_id.png)

<!-- textlint-disable -->

以上の設定で、 Client ID と Client Secret が作成できたと思います。

<!-- textlint-enable -->

この後のアプリケーション側の設定で必要になるので、捨てずに保存しておいてください。

## リライング・パーティーの実装

今回、リライング・パーティーの実装は、**Hono** × **Bun** を使用します。

- [Hono](https://hono.dev/): 高速で軽量なウェブフレームワーク
- [Bun](https://bun.sh/): JavaScript Runtime、超高速でスクリプトの実行が可能

:::message
以降では、 Bun がホストマシンにインストールされていて、bun コマンドが実行できることを前提として進めます。

:::

今回は、再現を容易にするため、リライング・パーティーを実装したリポジトリを用意しました。実際の挙動を再現したい方は、是非以下のリポジトリを手元にクローンしていただき、README の手順通りに実行してみてください。
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

2. **ユーザー情報提供を同意**

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

# 参考記事

https://authya.booth.pm/items/1296585
https://authya.booth.pm/items/1550861
https://authya.booth.pm/items/1877818
https://www.sakimura.org/2012/02/1487/
https://qiita.com/TakahikoKawasaki/items/4ee9b55db9f7ef352b47
