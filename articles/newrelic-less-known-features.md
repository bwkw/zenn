---
title: "NewRelicのあまり知られていないであろう機能に迫る"
emoji: "🔭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["newrelic"]
published: false
---

# はじめに
こんにちは、NewRelicのゆるキャラ「ニャリック」のステッカーを会社のPCに貼っている内藤です。

先日、弊社の [tomo_nxn](https://zenn.dev/tomo_nxn) さんと一緒に、~~「New Relic 実践入門 第2版」(翔泳社、3410円)を無料でもらう目的で~~ 担当プロジェクトでNew Relicを活用していく目的で、「**New Relic ハンズオン for beginners**」というイベントに参加してきましたw

この記事では、NewRelicの基本概念と重要性、なぜ最近これほどまでに注目されているのかを掘り下げつつ、基本的な機能およびこのイベントで知った魅力的な機能、さらには後から自分で調べて見つけた興味深い機能について紹介できたらと思います〜

「イベント内容について特に触れないんかい！」と思ったそこのあなた、イベント内容については、[tomo_nxn](https://zenn.dev/tomo_nxn) さんがまとめてくださっているので、是非こちらを覗いてみてください！（宣伝）
https://zenn.dev/levtech/articles/908463b504ed75

# そもそもNewRelicって？
NewRelicは、アプリケーションとインフラのパフォーマンスをリアルタイムで監視するためのオールインワンオブザーバビリティプラットフォームです。

オブザーバビリティとは、「**アウトプットからシステムをどれだけよく理解できるか？**」という能力を指し、システムのパフォーマンスや問題点をリアルタイムで可視化し、問題の早期発見と迅速な解決を可能にします。

# はいはい、つまり監視ツールね
「はいはい、つまり監視ツールね」と思ったそこのあなた、少しお待ちを🖐️

なぜなら、NewRelicはただシステムを監視するためのものではなく、そのデータを基にして、もっと深くシステムを理解し、改善していくための洞察を提供してくれるものだからです。

従来の監視ツールが「**何が起きたか？**」を提供するのに対し、オブザーバビリティツールは「**なぜそれが起きたのか？**」を解明することに注力しています。

つまり、NewRelicは、ただ見守ってくれるだけの監視ツールではなく、システムをよりよく理解し、改善していくための強力なパートナー、私はそんな存在だと思っています！

:::message
弊社は、NewRelicよりお金をいただいてるわけではございませんww
:::

# なぜ急にこんなにも注目を集めてるの？
弊社でも最近NewRelicを導入する運びとなり、私はその時初めて「**オブザーバビリティ**」という概念を知ったのですが、正直「なんで急にこんなにも注目を集めているんだろう？」って疑問が湧きましたw 

調べてみると、オブザーバビリティが注目を集めている理由は、主に以下の2点であるとされています。

**1. システムの複雑性の増大**
技術の進化とともにアーキテクチャが複雑化し、障害の原因を迅速に特定する必要が生まれています。

**2. 開発者の業務範囲の拡大**
DevOpsの普及により、開発者はコードの開発だけでなく運用にも関与するようになりました。これにより、システムの継続的な監視と迅速な問題解決が求められています。

上記2点の理由からも、「**オブザーバビリティ**」がビジネスとテクノロジーが成長する中で、今後も重要な役割を果たすことが伺えますね！

# よく知られている機能たち
## [APM](https://docs.newrelic.com/docs/apm/new-relic-apm/getting-started/introduction-apm/)
個人的には、NewRelicの代名詞です。

## [ブラウザ監視](https://docs.newrelic.com/docs/browser/browser-monitoring/getting-started/introduction-browser-monitoring/)

## [インフラストラクチャ監視](https://docs.newrelic.com/docs/infrastructure/infrastructure-monitoring/get-started/get-started-infrastructure-monitoring/0)

## [ログ管理](https://docs.newrelic.com/docs/logs/get-started/get-started-log-management/)

# 実はこんなことも出来る！
## [Synthetic Monitors](https://docs.newrelic.com/docs/synthetics/synthetic-monitoring/using-monitors/intro-synthetic-monitoring/)

## [Error Inbox](https://docs.newrelic.com/docs/apm/errors-inbox/errors-inbox-ui/)

## [IAST](https://docs.newrelic.com/docs/iast/introduction/)

## [CodeStream](https://docs.newrelic.com/docs/codestream/start-here/what-is-codestream/)

## [AI Monitoring](https://docs.newrelic.com/docs/ai-monitoring/intro-to-ai-monitoring/)

# おわりに
