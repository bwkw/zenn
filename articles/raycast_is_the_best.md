---
title: "早く仕事を終わらせたいあなたに捧げるRaycastのススメ"
emoji: "🛠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raycast"]
published: true
---

# TL;DR

- Raycast は、Mac で使用可能なランチャーツール
- Raycast は、アプリ/ファイル検索・過去のコピー履歴の呼び出し/検索・Google Chrome のタブ検索・Github の操作等が行える、とにかく多機能なツール
- Raycast を使いこなせたら、かっこいいし生産性は上がるしで悪いことが見つからない

https://www.raycast.com/

# はじめに

Raycast は、Mac で使用可能なランチャーツールです。

Mac のランチャーツールといえば、[Alfred](https://www.alfredapp.com/) が有名どころでしょうか。
https://www.alfredapp.com/

読者の方には、この記事を通して Raycast の凄まじさを感じてもらい、有効活用することで、仕事において**鬼の生産性**を実現していただけると幸いです。

# Alfred より優れている所

既に有名な Alfred より、Raycast はどこが優れているのでしょうか？

それは、**無料で多くの機能を使える点**、これに限ります。
なんだ、大したことなさそうと思った方、もう少しお付き合いくださいませ。

ここから、詳しく Raycast で使える機能を見ていきます。

# Raycast で何ができるのか？

## デフォルト機能

### アプリ検索

ランチャーツールで一番標準に備わっている、アプリ検索機能です。

Raycast の窓から、検索したいアプリ名を入力し、すぐに開くことが出来ます。
![slack](/images/raycast_is_the_best/slack.png)

### ファイル検索

自分のパソコンにあるファイルを検索できる機能です。

あのファイルどこやったっけってなる人にとっては、めちゃくちゃ便利な機能ですね。
![file](/images/raycast_is_the_best/search_file.png)

### 過去のコピー履歴の呼び出し/検索

過去のコピー履歴の呼び出し、検索ができる機能です。

過去のコピー履歴の呼び出しは、[Clipy](https://clipy.softonic.jp/mac) を使っている方が多いのではないでしょうか？
Clipy との違いはなんといっても、コピー履歴から検索までできることです。
![コピー履歴](/images/raycast_is_the_best/copy_history.png)

### スニペット作成・検索

頻繁に使用する文字をスニペットとして作成、またそのスニペットを検索できる機能です。

先程と同様に、Clipy を使えばスニペットの作成は出来ますが、スニペットの検索は出来ません。
特にスニペットは、「あれなんて登録したっけ？」となりがちなので、スニペットの検索機能は非常にありがたいですね！

自分は、よく使う「ありがとうございます！️」を `ari` でスニペット登録しています笑（怒られそう）
![スニペット](/images/raycast_is_the_best/snippet.png)

### ウィンドウのリサイズ・画面分割

現在アクティブなウィンドウをリサイズして配置できる機能です。
![window_resize](/images/raycast_is_the_best/window_resize.gif)

<!-- textlint-disable -->

自分は、[こちらの記事](https://dev.classmethod.jp/articles/eetann-used-raycast/) を参考に「3分割」なら1/3、左側ならl、中央ならc、右側ならrのような法則でエイリアスをつけています。

<!-- textlint-enable -->

| Name                | Alias  |
| :------------------ | :----: |
| First Third         | 1/3 l  |
| Center Third        | 1/3 c  |
| Last Third          | 1/3 r  |
| Bottom Center Sixth | 1/6 bc |
| Bottom Half         | 1/2 b  |

### Floating Notes で軽いメモを取る

他のアプリの使用中に常に表示されるメモを取ることができる機能です。

画面遷移の影響も受けず、常にメモを取れるので、会議中にさっとメモを取る時などに非常に便利です。
![floating_notes](/images/raycast_is_the_best/floating_notes.gif)

### カレンダーの予定確認

カレンダーの予定の確認が出来ます。

さらに、予定に Google Meet のリンクがあった場合、**Enter キー** で Meet に入ることが出来ます。（下画像の `Meeting`の一番右にビデオのアイコンがあります。このアイコンは、Google Meet のリンクがあることを指します）
![スケジュール](/images/raycast_is_the_best/schedule.png)

### 言語処理能力

Raycast は、窓の言語処理脳力も非常に高いのが特徴です。

簡単なドル円計算はもちろん、複雑な UTC(協定世界時)から JST(日本標準時)への計算なども行ってくれます。
![UTCからJST](/images/raycast_is_the_best/utc_to_jst.png)

### 電卓

Alfred と同様に電卓機能も備えています。
![電卓](/images/raycast_is_the_best/calculator.png)

## 拡張機能

### Bookmark 検索

https://www.raycast.com/raycast/browser-bookmarks
Raycast は、Bookmark 検索を拡張機能として備えています。

よく使うサイトは、Bookmark に入れておいて、これで即座に起動できます。
![Bookmark](/images/raycast_is_the_best/bookmark.png)

### Google Chrome のタブ検索

https://www.raycast.com/Codely/google-chrome

開いている Google Chrome のタブを検索できる機能です。

皆さん、Mac を使ってる時にタブを開きすぎて、「あのタブどこいっちゃったっけ？」ってなることよくありませんか？

私と同じような方は、これを使えば間違いなく普段の仕事の生産性が上がります。
![Google Chromeのタブ検索](/images/raycast_is_the_best/chrome_tab.png)

### Github への操作

https://www.raycast.com/raycast/github

連携した Github アカウントへ、様々な操作を行える機能です。

私は、リポジトリー検索・Pull Request 検索を特に使っています。
![Githubのリポジトリ検索](/images/raycast_is_the_best/github_repository.png)

### Notion のページ検索

https://www.raycast.com/reckoning-dev/search-notion

[数日前に日本語版を正式リリースした](https://twitter.com/NotionJP/status/1590283053594603520?s=20&t=pIundfDkS6Kw9nfsHPfiuQ)、Notion のページ検索も Raycast で行えます。

<!-- textlint-disable -->

Notionは、アプリ内でページ検索が行えるので、

<!-- textlint-enable -->

1. Raycast で、Notion を立ち上げる
2. Notion のアプリでページ検索

でも良いのですが、私はそれすらも手間なのでコマンド一発で検索できるようにしています。
![Notionのページ検索](/images/raycast_is_the_best/notion.png)

### JetBrains 製品で開いたプロジェクト検索

https://www.raycast.com/gdsmith/jetbrains
Raycast は、JetBrains 製品にも対応しています。

これを使えば、JetBrains 製品ですぐにプロジェクトを開く事ができるので、JetBrains 製品で開発しているエンジニアには必須の機能です。

![JetBrains](/images/raycast_is_the_best/jetbrains.png)
噂では、VsCode 版も存在するようなので、興味ある方は是非調べてみてください。

## 自作拡張機能

Raycast で自分が使いたい拡張機能がなかった場合、自分で作ることも出来ます。

Developer 向けに、この拡張機能の作り方のリファレンスがあるので、これを参考に自作することをオススメします。

<!-- textlint-disable -->

言語は、React + TypeScriptなので比較的とっつきやすいと思います。

<!-- textlint-enable -->

https://developers.raycast.com/

# ホットキーの設定

ランチャーツールで何人かの人が陥りがちなのが、 大して使わない機能にもホットキーを設定することです。 これではせっかく、生産性を上げるためにツールを導入しているにも関わらず、考えること（覚えること）が多くなり、かえって逆効果です。

ランチャーツールで重要なのは、**過不足なく頭を使わないように** 機能に対してホットキーを設定することです。

以下に、手元の例を載せます。

## 検索関連

検索関連のものは、**⌥** を使うことをベースにホットキーを設定していきます。 これにより、検索の時は **⌥** を使うという意識を最初だけすれば良く、これはすぐに無意識で実行できるようになります。

| 機能                                   | ホットキー |
| :------------------------------------- | :--------: |
| ファイル検索                           |     ⌥F     |
| クリップボードの履歴呼び出し・検索     |     ⌥V     |
| スニペット検索                         |     ⌥S     |
| Bookmark 検索                          |     ⌥B     |
| Google Chrome のタブ検索               |     ⌥C     |
| Github のリポジトリー検索              |     ⌥R     |
| Github の Pull Request 検索            |     ⌥P     |
| Notion のページ検索                    |     ⌥N     |
| JetBrains 製品で開いたプロジェクト検索 |     ⌥J     |

## 作成関連

作成関連のものは、**⌥^** を使うことをベースにホットキーを設定していきます。これにより、作成の時は **⌥^** を使うという意識を最初だけすれば良く、これもすぐに無意識で実行できるようになります。

| 機能                | ホットキー |
| :------------------ | :--------: |
| スニペット作成      |    ⌥^S     |
| リマインダー作成    |    ⌥^R     |
| Floating Notes 作成 |    ⌥^N     |

## 開く関連

開く関連のものは、**⌥⇧** を使うことをベースにホットキーを設定していきます。 これにより、開く関連の時は **⌥⇧** を使うという意識を最初だけすれば良く、これはすぐに無意識で実行できるようになります。
| 開くアプリ(対象) | ホットキー |
|:------------------------|:-----:|
| Slack | ⌥⇧S |
| Warp | ⌥⇧W |
| カレンダーの予定 | ⌥⇧C |

# その他の便利設定

## Auto-switch Input Source

Raycast 起動時に、デフォルトで ON になる入力ソースを選択出来ます。

これを `Romaji` にすることで、Raycast の起動時に必ずローマ字入力になります。（地味に便利）

![JST から UTC](/images/raycast_is_the_best/auto_switch_input_source.png =600x)

# 最後に

最後までお読みいただきありがとうございます。

Raycast の魅力は伝わりましたでしょうか？
少しでも Raycast に興味を持っていただけたら、実際にインストールして触ってみてください。
必ずや、あなたの生産性の向上の手助けとなってくれるハズです。

それでは、良い Raycast ライフを！
