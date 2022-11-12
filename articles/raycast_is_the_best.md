---
title: "Raycastを使って誰よりも早く帰りたいあなたへ"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raycast"]
published: false
---

# TL;DR
- Raycastは、Macで使用可能なランチャーツール
- Raycastは、アプリ/ファイル検索・過去のコピー履歴の呼び出し/検索・Google Chromeのタブ検索・Githubの操作等が行える、とにかく多機能なツール
- Raycastを使いこなせたら、かっこいいし生産性は上がるしで悪いことが見つからない

https://www.raycast.com/

# はじめに
Raycastは、Macで使用可能なランチャーツールです。

Macのランチャーツールといえば、[Alfred](https://www.alfredapp.com/) が有名どころでしょうか。
https://www.alfredapp.com/

読者の方には、この記事を通して、Raycastの凄まじさを感じてもらい、有効活用することで、仕事において**鬼の生産性**を実現していただけると幸いです。

# Alfredより優れている所
既に有名なAlfredより、Raycastはどこが優れているのでしょうか？

それは、**無料で多くの機能を使える点**、これに限ります。なんだ、大したことなさそうと思った方、もう少しお付き合いくださいませ。

ここから、詳しくRaycastで使える機能を見ていきましょう！

# Raycastで何が出来るのか？
## デフォルト機能
### アプリ/ファイル検索
ランチャーツールで一番標準に備わっている、アプリ/ファイル検索機能です。

Raycastの窓から、検索したいアプリ名/ファイル名を入力し、すぐに開くことが出来ます。
![slack](/images/raycast_is_the_best/slack.png)

### 過去のコピー履歴の呼び出し/検索
過去のコピー履歴の呼び出し、検索が出来る機能です。

過去のコピー履歴の呼び出しは、[Clipy](https://clipy.softonic.jp/mac)を使っている方が多いのではないでしょうか？
Clipyとの違いはなんといっても、コピー履歴から検索まで出来ることです。
![コピー履歴](/images/raycast_is_the_best/copy_history.png)

### スニペット作成・検索
頻繁に使用する文字をスニペットとして作成、またそのスニペットを検索出来る機能です。

先程と同様に、Clipyを使えばスニペットの作成は出来ますが、スニペットの検索は出来ません。
特にスニペットは、「あれなんて登録したっけ？」となりがちなので、スニペットの検索機能は非常にありがたいですね！

筆者は、よく使う「よろしくお願い致します🙇‍♂️」を `yoro` でスニペット登録しています笑（怒られそう
![スニペット](/images/raycast_is_the_best/snippet.png)

### ウィンドウのリサイズ・画面分割
現在アクティブなウィンドウをリサイズして配置出来る機能です。
![window_resize](/images/raycast_is_the_best/window_resize.gif)
筆者は、[こちらの記事](https://dev.classmethod.jp/articles/eetann-used-raycast/) を参考に「3分割」なら1/3、左側ならl、中央ならc、右側ならrのような法則でエイリアスをつけています。
| Name	               |  Alias  | 
|:--------------------|:-------:|
| First Third         | 	1/3 l  |
| Center Third        | 	1/3 c  |
| Last Third          | 	1/3 r  |
| Bottom Center Sixth | 	1/6 bc |
| Bottom Half         | 	1/2 b  |

### Floating Notesで軽いメモを取る
他のアプリの使用中に常に表示されるメモを取ることが出来る機能です。

画面遷移の影響も受けず、常にメモを取れるので、会議中にさっとメモを取る時などに非常に便利です。
![floating_notes](/images/raycast_is_the_best/floating_notes.gif)

### カレンダーの予定確認
カレンダーの予定の確認が出来ます。

さらに、予定にGoogle Meetのリンクがあった場合、**Enterキー** でMeetに入ることが出来ます。

些細な機能に思えますが、この便利さは使うと抜け出せません笑

### 言語処理能力
Raycastは、窓の言語処理脳力も非常に高いのが特徴です。

簡単なドル円計算はもちろん、複雑なUTC(協定世界時)からJST(日本標準時)への計算なども行ってくれます。
![UTCからJST](/images/raycast_is_the_best/utc_to_jst.png)

### 電卓
Alfredと同様に電卓機能も備えています。
![電卓](/images/raycast_is_the_best/calculator.png)

## 拡張機能
### Bookmark検索
https://www.raycast.com/raycast/browser-bookmarks
Raycastは、Bookmark検索を拡張機能として備えています。

よく使うサイトは、Bookmarkに入れておいて、これで即座に起動する事ができます。
![Bookmark](/images/raycast_is_the_best/bookmark.png)

### Google Chromeのタブ検索
https://www.raycast.com/Codely/google-chrome

開いているGoogle Chromeのタブを検索できる機能です。

皆さん、Macを使ってる時にタブを開きすぎて、「あのタブどこいっちゃったっけ？」ってなることよくありませんか？（筆者だけかも笑）

筆者と同じような方は、これを使えば間違いなく普段の仕事の生産性が上がります。
![Google Chromeのタブ検索](/images/raycast_is_the_best/chrome_tab.png)

### Githubへの操作
https://www.raycast.com/raycast/github

連携したGithubアカウントへ、様々な操作を行える機能です。

筆者は、リポジトリ検索・Pull Request検索を特に使っています。
![Githubのリポジトリ検索](/images/raycast_is_the_best/github_repository.png)

### Notionのページ検索
https://www.raycast.com/reckoning-dev/search-notion

[数日前に日本語版を正式リリースした](https://twitter.com/NotionJP/status/1590283053594603520?s=20&t=pIundfDkS6Kw9nfsHPfiuQ)、Notionのページ検索もRaycastで行えます。

Notionは、アプリ内でページ検索が行えるので、
1. Raycastで、Notionを立ち上げる
2. Notionのアプリでページ検索

でも良いのですが、筆者はそれすらも手間なのでコマンド一発で検索できるようにしています。
![Notionのページ検索](/images/raycast_is_the_best/notion.png)

### Jet Brains製品で最近起動したプロジェクトの検索
Raycastは、Jet Brains製品にも対応しています。

これを使えば、Jet Brains製品ですぐにプロジェクトを開く事が出来るので、Jet Brains製品で開発しているエンジニアには必須の機能です。

噂では、VsCode版も存在するようなので、興味ある方は是非調べてみてください。
噂では、VsCode版も存在するようなので、興味ある方は是非調べてみてください。

https://www.raycast.com/gdsmith/jetbrains

# 筆者のホットキー
ランチャーツールで何人かの人が陥りがちなのが、 大して使わない機能にもホットキーを設定することです。 これではせっかく、生産性を上げるためにツールを導入しているにも関わらず、考えること（覚えること）が多くなり、かえって逆効果です。

ランチャーツールで重要なのは、**過不足なく頭を使わないように** 機能に対してホットキーを設定することです。

以下に、筆者の手元の例を載せます。

## 検索機能
検索関連のものは、**⌥** を使うことをベースにホットキーを設定していきます。 これにより、検索の時は **⌥** を使うという意識を最初だけすれば良く、これはすぐに無意識で実行できるようになります。

| 機能                      | ホットキー |
|:------------------------|:-----:|
| クリップボードの履歴呼び出し・検索       |  ⌥V   |
| スニペット検索                 |  ⌥S   |
| Bookmark検索              |  ⌥B   |
| Google Chromeのタブ検索      |  ⌥C   |
| Githubのリポジトリ検索          |  ⌥R   |
| GithubのPullRequest検索    |  ⌥P   |
| Notion検索                |  ⌥N   |
| JetBrains製品で開いたプロジェクト検索 |  ⌥J   |

## 作成関連
作成関連のものは、**^⌥** を使うことをベースにホットキーを設定していきます。これにより、作成の時は **^⌥** を使うという意識を最初だけすれば良く、これもすぐに無意識で実行できるようになります。

| 機能               | ホットキー |
|:-----------------|:-----:|
| スニペット作成          |  ^⌥S  |
| リマインダー作成         |  ^⌥R  |
| Floating Notes作成 |  ^⌥N  |

# その他の便利設定
## Auto-switch Input Source
Raycast起動時に、デフォルトでONになる入力ソースを選択出来ます。

これを `Romaji` にすることで、Raycastの起動時に必ずローマ字入力になります。（地味に便利）

![JSTからUTC](/images/raycast_is_the_best/auto_switch_input_source.png =600x)

# 最後に
最後までお読みいただきありがとうございます。

Raycastはいかがでしたでしょうか？
少しでもその魅力は伝わりましたでしょうか？

もし、少しでも興味を持っていただけたら、実際にインストールして触ってみてください。
必ずや、あなたの生産性の向上の手助けとなってくれるハズです。

それでは、良いRaycastライフを！