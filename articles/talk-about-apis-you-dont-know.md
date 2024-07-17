---
title: "君たちの知らないAPIデザインパターンの話をしよう"
emoji: "👺"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["API"]
published: true
published_at: 2024-04-11 11:00
publication_name: "levtech"
---

:::message
この記事では、**Web API** にフォーカスして話を進めていきます。
一部「**API**」と表現している部分がありますが、それらは全て「**Web API**」として解釈いただけますと幸いです。
:::

# TL;DR
- API 設計の学習に、「API デザインパターン」を読んだ
- 良い API は、実行可能であり、表現力があり、シンプルで予測可能なもの
- API 設計は、REST の原則に沿って直感的にできることが多いが、デザインパターンを学ぶことで複雑な要件に対応することが可能
- 今回は、数あるデザインパターンの中で、カスタムメソッド、ロングランオペレーション、再実行可能ジョブ、シングルトンサブリソースの 4 つを紹介

# はじめに
こんにちは、エンジニア歴 2 年目に入り、後輩の入社に怯えている内藤です。

先日、API 設計について体系的に学べる本がないかな〜って探していたところ、先輩エンジニアに「API デザインパターン」を紹介いただきました。

https://techplay.jp/book/373

この本は、名前の通り API のデザインパターンを紹介している本なのですが、この本を読んだ正直な感想としては「めちゃくちゃ使うってほどでもないな。..w」でした。ただ、特定の複雑な要件や特殊な状況に対応する際には、改めて読み返す価値のある一冊だなとも思いました。

そこで、この記事では、API の基本からいくつかのデザインパターンに触れ、その有用性について整理できたらな〜と思います！

# そもそも良いAPIとは？
良い API を設計するためには、以下の 4 つの特性を網羅している必要があります。

### 実行可能であること
良い API は、システムがユーザーの要求を迅速かつ正確に満たす能力を持つべきです。これは、ユーザーが求める操作を効率的に、かつ適切に実行できることを意味します。

### 表現力があること
表現力豊かな API は、ユーザーが望む機能をシンプルかつ直感的に表現でき、ユーザーは少ない努力で目的を達成できます。

### シンプルであること
シンプルさは、API の要素数を単純に減らすことではありません。むしろ、ユーザーが必要とする機能を最も直接的かつ簡単な方法で利用できるようにすることが求められます。

### 予測可能であること
API は、その動作が一貫しており、ユーザーが直感的に理解できるようなものでなければなりません。これにより、使用時の不意打ちがなく、信頼性の高いインターフェイスを提供します。

# よく見るAPI
Web サービスで一番馴染みのある API の設計思想は REST かと思います。

REST の設計は以下の 4 つの基本原則に基づいています。
- アドレス可能性 : 各リソースは一意の URI で識別される
- ステートレス性 : サーバーとクライアント間の通信はステートレスである
- 接続性 : リソース間の関係をリンクを通じて表現できる
- 統一インターフェース : HTTP メソッド（GET、POST、PUT、DELETE など）によって、一貫したインターフェースでさまざまな操作を実現できる

この REST の原則に沿った典型的な API のエンドポイント設計の例を以下に示します。

| 目的          | エンドポイント                            | メソッド      |
|-------------|------------------------------------|-----------|
| ユーザーの一覧取得    | http://api.example.com/users       | GET       |
| ユーザーの新規登録    | http://api.example.com/users       | POST      |
| 特定のユーザーの情報取得 | http://api.example.com/users/:id	  | GET       |
| ユーザーの情報更新    | http://api.example.com/users/:id   | PUT/PATCH |
| ユーザーの情報の削除   | 	http://api.example.com/users/:id	 | DELETE    |

このように、REST の設計原則に従って API を構築することで、ほとんどの API 設計は直感的に、かつ問題なく行うことができます。

# デザインパターンの紹介
ここからが本題です。大抵の場合、上の例で示したような API 設計で十分です。

ただ、複雑な要件では、上のような典型的な API 設計のみでは[良いAPIを設計するための4つの特性](https://zenn.dev/levtech/articles/talk-about-apis-you-dont-know#%E3%81%9D%E3%82%82%E3%81%9D%E3%82%82%E8%89%AF%E3%81%84api%E3%81%A8%E3%81%AF%EF%BC%9F)を満たせないことがあり、そのような場合のためにデザインパターンが有効です。

## カスタムメソッド
### 概要
カスタムメソッドは、標準的な CRUD 操作（作成、読み取り、更新、削除）では対応できない特定の操作が必要になる場合に便利です。例えば、メールの送信や即時の文書翻訳など、通常の `create` や `update` メソッドでは処理が難しい操作がこれに該当します。

参考までに、以下に Google が出しているカスタムメソッドの記事を示します。
https://cloud.google.com/apis/design/custom_methods?hl=ja

### 実装例
以下は、カスタムメソッドを扱う API の実装例です。`EmailApi`抽象クラスを介して、特定のユーザーにメールを送信する方法を示しています。 この例の注目点は、エンドポイントの定義における`:send`, `:unsend`の使用です。ここでの`:send`, `:unsend`は、カスタムメソッドの命名規則を示しており、標準的な HTTP メソッド（GET、POST など）に対する補足的な操作を表します。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class EmailApi {
>   @post("/{id=users/*emails/*}:send")  // ここ
>   abstract SendEmail(req: SendEmailRequest): Email;  // メールを送信する 
>   
>   @post("/{id=users/*/emails/*}:unsend")  // ここ
>   abstract UnsendEmail(req: UnsendEmailRequest): Email;  // メール送信を中断する
> }
> 
> interface Email {   // メール
>   id: string;       // ID
>   subject: string;  // 件名
>   content: string;  // 本文
>   state: string;    // 状態（draft(下書き), sent(送信), unsend(未送信)）  
>   // ...
> }
> ```

### いつ使えそうか？
ユーザーのアカウントを一時停止したいって時に、アカウントの状態変更、ユーザーセッションの無効化、外部サービスへの通知などのいくつかのステップを`suspendAccount`っていうカスタムメソッド一発でできるようにする、みたいな時に使えそうだな〜と思いました！

## ロングランオペレーション (Long Running Operations, LRO)
### 概要
LRO は、完了までに時間がかかる処理を指し、デザインパターンではこれを非同期動作を通じてサポートします。すべての操作を典型的な API 設計で処理すると、速い処理と遅い処理の区別がなくなり、API が提供すべき予測可能な振る舞いを損なうことになります。

時間がかかる処理を行う場合、非同期処理を利用して、作業の開始時にプレースホルダーを返し、作業完了時には結果と共にプレースホルダーを「解決」するか、エラー発生時に「拒否」します。

### 実装例
以下は、LRO を扱う API の実装例です。この例では、`ChatRoomApi`抽象クラスを利用して、進行中の操作の状態を取得し、操作が完了するまで待機し、また必要に応じて操作をキャンセルする方法を紹介します。

操作が完了し結果が利用可能になると、`Operation`オブジェクトの`done`フラグは`true`に設定され、`result`フィールドには結果が格納されます。この時点で、`Operation`オブジェクトは「解決された」と見なされ、クライアントは結果を取得し利用できます。
一方で、操作がエラーにより失敗すると、`done`フラグは同様に true に設定されますが、`result`フィールドの代わりに`OperationError`オブジェクトが返されます。このオブジェクトにはエラーコード、エラーメッセージ、および追加の詳細情報が含まれ、「拒否された」と見なされます。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class ChatRoomApi {
>   @get("/{id=operations/*}")
>   abstract GetOperation<ResultT, MetadataT>(req: GetOperationRequest):
>     Operation<ResultT, MetadataT>;  // 指定されたIDを持つ操作の現在の状態を取得
> 
>   @get("/{id=operations/*}:wait")
>   abstract WaitOperation<ResultT, MetadataT>(req: WaitOperationRequest):
>     Operation<ResultT, MetadataT>;  // 指定されたIDを持つ操作が完了するまで待機する
> 
>   @post("/{id=operations/*}:cancel")
>   abstract CancelOperation<ResultT, MetadataT>(req: CancelOperationRequest):
>     Operation<ResultT, MetadataT>;  // 指定されたIDを持つ進行中の操作をキャンセルする
> }
> 
> interface Operation<ResultT, MetadataT> {  // プレースホルダー
>   id: string;                              // ID 
>   done: boolean;                           // 操作の完了を表すフラグ
>   expireTime: Date;                        // 操作が期限切れになる時刻
>   result?: ResultT | OperationError;       // 操作の結果（未完了の場合は設定されない）
>   metadata?: MetadataT;                    // 操作に関する追加情報
> }
> 
> interface OperationError {  // エラー
>   code: string;             // ステータスコード
>   message: string;          // エラーメッセージ
>   details?: any;            // エラー詳細
> }
> 
> interface GetOperationRequest {  // 指定されたIDを持つ操作の現在の状態を取得するリクエスト
>   id: string;                    // ID
> }
> 
> interface WaitOperationRequest {  // 指定されたIDを持つ操作が完了するまで待機するリクエスト
>   id: string;                     // ID
> }
> 
> interface CancelOperationRequest {  // 指定されたIDを持つ進行中の操作をキャンセルするリクエスト
>   id: string;                       // ID
> }
> ```

### いつ使えそうか？
大量の画像や動画をクラウドストレージにバッチでアップロードし、その処理状況を追跡するみたいな時に使えそうだな〜と思いました。

API を介してアップロードジョブを開始し、クライアント側でジョブの ID を利用して定期的にアップロードの進捗を確認できるようにする。進捗状況は非同期に更新され、完了したら最終的なレポートが生成される。もしアップロード中に問題が発生した場合は、そのジョブを特定して対処し、必要に応じて再開や再実行ができるようになっている。みたいな感じになっていると便利そうだな〜と！

## 再実行可能ジョブ
### 概要
再実行可能ジョブは、ジョブの設定と実行を明確に分離します。このアプローチにより、開発者は API を介してタスクの実行条件を事前に定義でき、これらの設定を基にして後にタスクを実行することが可能になります。

この設計は、API 自体でタスクの即時実行や定期実行を管理し、外部スケジューラーに頼らない柔軟性を提供します。また、ジョブ設定と実行の権限分離により、開発者はジョブの設定を担当し、運用チームは実際のジョブの実行を管理するといった、責任範囲の明確化ができます。

### 実装例
以下は、再実行可能ジョブの API 実装例です。この例では、メッセージ分析を目的としたジョブの設定（`CreateAnalyzeChatRoomJob`)と、非同期にそのジョブを実行するためのカスタム run メソッド（`RunAnalyzeChatRoomJob`)を利用しています。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class ChatRoomApi {
>   @post("/analyzeChatRoomJobs")
>   abstract CreateAnalyzeChatRoomJob(req: CreateAnalyzeChatRoomJobRequest):
>     AnalyzeChatRoomJob;  // チャットルーム分析ジョブを作成する 
> 
>   @post("/{id=analyzeChatRoomJobs/*}:run")
>   abstract RunAnalyzeChatRoomJob(req: RunAnalyzeChatRoomJobRequest):
>     Operation<AnalyzeChatRoomJobExecution, RunAnalyzeChatRoomJobMetadata>;  // 指定されたIDのチャットルームの分析ジョブを実行する
> }
> 
> interface CreateAnalyzeChatRoomJobRequest {  // チャットルーム分析ジョブを作成するリクエスト
>   resource: AnalyzeChatRoomJob;              // チャットルーム分析ジョブの設定
> }
> 
> interface AnalyzeChatRoomJob {  // チャットルーム分析ジョブの設定
>   id: string;                   // ID
>   chatRoom: string;             // 分析するチャットルーム
>   destination: string;          // 結果の保存先
>   compressionFormat: string;    // 圧縮形式
> }
> 
> interface RunAnalyzeChatRoomJobRequest {  // 指定されたIDのチャットルームの分析ジョブを実行するリクエスト
>   id: string;                             // ID
> }
> 
> interface AnalyzeChatRoomJobExecution {  // チャットルーム分析ジョブの実行結果
>   id: string;                            // ID
>   job: AnalyzeChatRoomJob;               // チャットルーム分析ジョブの設定
>   sentenceComplexity: number;            // 文章の複雑さ
>   sentiment: number;                     // 感情分析のスコア
>   abuseScore: number;                    // 虐待スコア
> }
> 
> interface RunAnalyzeChatRoomJobMetadata {  // チャットルーム分析ジョブの実行結果の追加情報
>   messagesProcessed: number;               // 処理されたメッセージの総数
>   messagesCounted: number;                 // 分析対象のメッセージの総数
> }
> ```

### いつ使えそうか？
システムのセキュリティチェックのジョブを事前にセットアップしておいて、それが自動でバックグラウンドで走るように API に仕込むっていう使い方が良さそうだな〜と思いました。

毎月とか 3 ヶ月に 1 回とか、決まったスケジュールでシステムの安全性をチェックして、問題があったらレポートを自動生成してくれる。で、もし何か緊急のセキュリティリスクが出たときとかには、slack で通知飛んでくるようにしておいて、その修正後にそのジョブを手動でさっと再実行できるようにしておくと便利そうだな〜と！

## シングルトンサブリソース
### 概要
API 設計において、リソースの特定のプロパティがそのサイズや複雑さ、セキュリティー要件、または頻繁な更新の必要性により、通常のプロパティと同様に扱うことが難しい場合があります。シングルトンサブリソースを利用することで、これらのプロパティを独立したエンティティとして設計し、それぞれに適した管理方法を適用することが可能になります。

### 実装例
以下は、シングルトンサブリソースを扱う API の実装例です。`RideSharingApi`抽象クラスは、ドライバーの基本情報の取得・更新メソッドと、位置情報の取得・更新メソッドを別々に提供します。これにより、ドライバーの位置情報などの頻繁に変更されるデータを独立して管理し、更新することが可能になります。

※ デザインパターンのコードを引用しています。
> ```typescript
> abstract class RideSharingApi {
>   @get("/{id=drivers/*}")
>   abstract GetDriver(req: GetDriverRequest): Driver;  // ドライバーの情報を取得する
> 
>   @patch("/{resource.id=drivers/*}")
>   abstract UpdateDriver(req: UpdateDriverRequest): Driver;  // ドライバーの情報を更新する
> 
>   @get("/{id=drivers/*/location}")
>   abstract GetDriverLocation(req: GetDriverLocationRequest): DriverLocation;  // ドライバーの位置情報を取得する
> 
>   @patch("/{resource.id=drivers/*/location}")
>   abstract UpdateDriverLocation(req: UpdateDriverLocationRequest): DriverLocation;  // ドライバーの位置情報を更新する
> 
> }
> 
> interface Driver {       // ドライバー 
>   id: string;            // ID
>   name: string;          // 名前
>   licensePlate: string;  // ナンバープレート
> }
> 
> interface DriverLocation {  // ドライバーの位置情報
>   id: string;               // ID
>   lat: number;              // 緯度 
>   long: number;             // 経度
>   updateTime: Date;         // 位置情報の最終更新時間
> }
> 
> interface GetDriverLocationRequest {  // ドライバーの位置情報を取得するリクエスト
>   id: string;                         // ID
> }
> 
> interface UpdateDriverLocationRequest {  // ドライバーの位置情報を更新するリクエスト 
>   resource: DriverLocation;              // 更新するドライバーの位置情報
>   fieldMask: FieldMask;                  // 更新するフィールドを指定するマスク
> }
> ```

### いつ使えそうか？
ショッピングサイトなどで商品ごとにめちゃくちゃたくさんの画像があって、それが頻繁に更新されるって時に使えそうだな〜と思いました。

商品画像を商品情報とは独立したサブリソースとして設計すれば、商品画像の更新とか追加がすごくやりやすくなるだろうな〜と！

# おわりに
これまで、良い API の本質とその実現に役立つデザインパターンについて紹介してきました。

私自身、デザインパターンを知らないあまり、強引に API を設計していたことも多々あったな。.。と考えさせられましたし、API 設計を考える上で 1 つのひっかかりが出来たという意味で、読んで価値ある本だったなと思いました。

まだまだ紹介出来てないデザインパターンが多くあるので、興味が湧いた方は是非手に取ってみてください👋
