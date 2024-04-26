---
title: "我々はまだ知らなかった。NewRelicの真の姿を"
emoji: "🔭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["newrelic"]
published: true
published_at: 2024-04-30 11:00
publication_name: "levtech"
---

# TL;DR
- NewRelicは、ただの「監視ツール」じゃなく「**オールインワンオブザーバビリティプラットフォーム**」である
- 「APM」「ブラウザ監視」「インフラストラクチャ監視」「ログ管理」は既に弊チームに導入している
- 「Synthetic Monitoring」「Errors Inbox」「分散トレーシング」「CodeStream」は今後弊チームに導入していきたい

# はじめに
こんにちは、NewRelicのゆるキャラ「ニャリック」のステッカーを会社のPCに貼っている内藤です。

先日、弊社の [tomo_nxn](https://zenn.dev/tomo_nxn) さんと一緒に、~~[「New Relic 実践入門 第2版」(翔泳社、3410円)](https://www.amazon.co.jp/dp/4798184500)を無料でもらう目的で~~ 担当プロジェクトでNew Relicを活用していく目的で、New Relic日本法人初のハンズオンイベントに参加してきましたw

イベント内容については、[tomo_nxn](https://zenn.dev/tomo_nxn) さんがまとめてくださっているので、是非こちらを覗いてみてください！（ただの宣伝）
https://zenn.dev/levtech/articles/908463b504ed75

この記事ではイベント内容には特に触れず、まずNewRelicの基本概念とその重要性に焦点を当て、最近なぜこれほど注目されているのかを掘り下げます。次に、基本的な機能やイベントで知った魅力的な機能、そして後から自分で調べて見つけた興味深い機能について紹介します。最後に、実際弊チームが今後取り組んでいくことについて整理できたらと思います〜

:::message
「NewRelicの基本的なことなんて知ってるよ」って方は「[実はこんな機能もあるんやで](https://zenn.dev/levtech/articles/newrelic-less-known-features#%E5%AE%9F%E3%81%AF%E3%81%93%E3%82%93%E3%81%AA%E6%A9%9F%E8%83%BD%E3%82%82%E3%81%82%E3%82%8B%E3%82%93%E3%82%84%E3%81%A7)」に飛んでください！
:::

# そもそもNewRelicって？
NewRelicは、アプリケーションとインフラのパフォーマンスをリアルタイムで監視するための「**オールインワンオブザーバビリティプラットフォーム**」です。

オブザーバビリティとは、「**アウトプットからシステムをどれだけよく理解できるか？**」という能力を指します。この能力によって、システムのパフォーマンスや問題点をリアルタイムで可視化し、問題を早期に解決することが可能になります。

# はいはい、つまり監視ツールね
「はいはい、つまり監視ツールね」と思ったそこのあなた、少しお待ちを🖐️

なぜなら、NewRelicはただシステムを監視するためのものではなく、そのデータを基にして、もっと深くシステムを理解し、改善していくための洞察を提供してくれるものだからです。

従来の監視ツールが「**何が起きたか？**」を提供するのに対し、オブザーバビリティツールは「**なぜそれが起きたのか？を解明する機能**」を提供しています。

つまり、NewRelicは、ただ見守ってくれるだけの監視ツールではなく、システムをよりよく理解し、改善していくための強力なパートナー、私はそんな存在だと思っています！

:::message alert
**弊社は、NewRelicよりお金をいただいてるわけではございませんw**
:::

# なぜ急にこんなにも注目を集めてるの？
弊社でも最近NewRelicを導入する運びとなり、私はその時初めて「**オブザーバビリティ**」という概念を知ったのですが、同時に「なんで急にこんなにも注目を集めているんだろう？」って疑問も湧きました。 

調べてみると、オブザーバビリティが注目を集めている理由は、主に以下の2点であるとされているようです。

1. **システムの複雑性の増大**
技術の進化により、アーキテクチャが複雑化し、障害の原因特定が難しくなっています。これにより、復旧作業に多くの時間を要するため、迅速な問題解決が求められるようになりました。

2. **開発者の業務範囲の拡大** 
DevOpsの普及に伴い、開発者はコードの開発だけでなく、運用にも関与するようになりました。これにより、システムの継続的な監視と迅速な問題解決が一層重要になっています。

上記2点の理由からも、「**オブザーバビリティ**」がビジネスとテクノロジーが成長する中で、今後も重要な役割を果たすことが伺えますね〜！

# よく知られている機能たち
これまで、「**オブザーバビリティ**」についてお話ししてきましたが、このセクションでは、一般的によく知られている（であろう）機能をいくつか紹介したいと思います！

## [APM](https://docs.newrelic.com/jp/docs/apm/new-relic-apm/getting-started/introduction-apm/)
個人的には、NewRelicの代名詞で、「**オブザーバビリティ**」の真髄だと思っています。

APM機能を用いてメトリクス、イベント、ログ、トランザクション（MELT）を監視することで、アプリの健全性のリアルタイムな追跡と、エラー発生時の迅速な原因究明が可能になります。

![apm](/images/newrelic_less_known_features/apm.png)

## [ブラウザ監視](https://docs.newrelic.com/jp/docs/browser/browser-monitoring/getting-started/introduction-browser-monitoring/)
従来の監視ツールでもお馴染みのやつですね。

ブラウザ監視機能によって、フロントエンドのパフォーマンスを測定し、ユーザー体験を向上させるためのデータを取得できます。ページロード時間の分析からJavaScriptエラーの特定まで、フロントエンドの問題を詳細に把握できます。

![browser](/images/newrelic_less_known_features/browser_monitoring.png)

## [インフラストラクチャ監視](https://docs.newrelic.com/jp/docs/infrastructure/infrastructure-monitoring/get-started/get-started-infrastructure-monitoring/)
これも従来の監視ツールでもお馴染みのやつですね。

インフラストラクチャ監視によって、サーバー、仮想マシン、コンテナなどのリソースの健康状態を一元的に視覚化できます。リソースの使用状況やパフォーマンスの変動を追跡し、システム全体の最適化を行うことができます。

![infrastructure](/images/newrelic_less_known_features/infrastructure_monitoring.png)

## [ログ管理](https://docs.newrelic.com/jp/docs/logs/get-started/get-started-log-management/)
これも従来の監視ツールでもお馴染みのやつですね。

ログ管理機能によって、ログデータを一元的に集約し、検索や分析を容易に行うことができます。

![logs](/images/newrelic_less_known_features/logs_management.png)

# 実はこんな機能もあるんやで
NewRelicには前章で紹介したような基本的な機能に加えて、以下のような魅力的な機能が存在します！

これらの機能は従来の監視ツールではあまり見かけないもので、NewRelicの「**オブザーバビリティ**」のコンセプトをより強く感じる機能のように思います！

## [Synthetic Monitoring](https://docs.newrelic.com/jp/docs/synthetics/synthetic-monitoring/getting-started/get-started-synthetic-monitoring/)
Synthetic Monitoringによって、実際のユーザーの行動をシミュレートしてウェブサイトやアプリケーションのパフォーマンスをテストすることができます。これにより、ユーザーに影響を与える前に問題を特定し、修正することができます。

![synthetic monitoring](/images/newrelic_less_known_features/synthetic_monitoring.png)

## [Errors Inbox](https://docs.newrelic.com/jp/docs/errors-inbox/getting-started/)
Errors Inboxによって、アプリケーションのエラーを一箇所に集約し、効率的にエラーを解析し、対応することができます。これにより、エラーの優先順位付け、割り当て、追跡が簡単になり、エラー対応のプロセスがスムーズになります。

![errors_inbox](/images/newrelic_less_known_features/errors_inbox.png)

## [分散トレーシング](https://docs.newrelic.com/jp/docs/distributed-tracing/concepts/introduction-distributed-tracing/)
分散トレーシングによって、マイクロサービスや大規模分散システムのパフォーマンスを追跡・分析することができます。これにより、複数のサービスやコンポーネントを跨ぐリクエストのフローを視覚化し、それぞれのコンポーネントでの処理時間や通信の遅延をリアルタイムで監視することができます。

![分散トレーシング](/images/newrelic_less_known_features/distributed-tracing.png)

## [CodeStream](https://docs.newrelic.com/jp/docs/codestream/start-here/what-is-codestream/)
New Relic CodeStreamによって、パフォーマンスやエラーに関する問題をIDE内で特定することができます。CodeStreamを利用することで、開発者はエラー、パフォーマンスの低下、サービスレベル目標（SLO）の違反などの問題をIDE内で直接確認できます。

![code_stream](/images/newrelic_less_known_features/code_stream.png)

# 偉そうに言ってるけど、お前は何をやるんだ？
これまでNewRelicについて説明してきましたが、弊チームで今後何に取り組んでいくかを少しお話しできればと思います！

これまで、弊チームではNewRelicの「[よく知られている機能たち](https://zenn.dev/levtech/articles/newrelic-less-known-features#%E3%82%88%E3%81%8F%E7%9F%A5%E3%82%89%E3%82%8C%E3%81%A6%E3%81%84%E3%82%8B%E6%A9%9F%E8%83%BD%E3%81%9F%E3%81%A1)」の導入を進めてきており、あとは運用に乗せるだけです。
今後は、システムの問題を定期的に振り返りながら、発見した問題の改善に取り組む運用プロセスの構築に力を入れていきます〜！（これでやるしかなくなったｗ）

一方で、「[実はこんな機能もあるんやで](https://zenn.dev/levtech/articles/newrelic-less-known-features#%E5%AE%9F%E3%81%AF%E3%81%93%E3%82%93%E3%81%AA%E6%A9%9F%E8%83%BD%E3%82%82%E3%81%82%E3%82%8B%E3%82%93%E3%82%84%E3%81%A7)」で紹介した機能については、残念ながらまだ導入が進んでいません。

「Synthetic Monitoring」は、主にユーザーの体験を向上させるための機能で、より良いサービスを作るために欠かせないものだと思っているので、なるべく早く導入したいです。
しかし弊チームでは、現時点で特にエラーやアラート対応でしんどい思いをしてるので、今後は「Errors Inbox」「分散トレーシング」「CodeStream」といった機能の導入に積極的に動いていきます〜！（こっちもやるしかなくなったw）

# おわりに
NewRelicについて長らく偉そうに話してきましたが、弊社も導入したてで、僕もまだまだ知らないことばかりです。
ただ、NewRelicは、きっと弊社のシステムの運用コストを下げ、よりよいものにしていくためのパートナー、きっとそんな存在になってくれるはずです！

今後導入を進めていく中で、多くの成功と失敗があるかと思います。その辺りもいずれ記事に書こう(というより誰かが書いてくれるはずw)と思うので、その辺りが気になる方は是非弊社テックブログをウォッチしててくださいね〜〜

# 参考文献
https://newrelic.com/jp/blog/best-practices/what-is-observability
https://newrelic.com/jp/blog/best-practices/what-is-observability-difference-from-debugging
https://newrelic.com/jp/blog/best-practices/what-is-observability-difference-from-monitoring
