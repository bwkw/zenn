---
title: "「アーキテクチャカンファレンス 2024」で学んだアーキテクチャとの付き合い方"
emoji: "⚙️"
type: "tech"
topics: ["アーキテクチャ", "architecture"]
published: true
published_at: 2025-01-10 11:00
publication_name: "levtech"
---

# TL;DR

- アーキテクチャカンファレンス 2024 に参加した
- トレードオフ分析では、評価軸の重み付けや市場の変化に応じた継続的見直しが重要
- 組織とアーキテクチャは相互に影響を与える
- アーキテクチャはあくまでビジネス目標達成の手段であることに留意するべき
- 進化的アーキテクチャ実現のために、シンプルさと柔軟性を重視し、偶有的複雑性を最小化するべき

# はじめに

<!-- textlint-disable -->

こんにちは、直近はシステムのリプレースに着手している内藤です。
リプレースや大規模な設計変更を進めている方にとって、アーキテクチャ設計の悩みは尽きないですよね...

先日参加した「アーキテクチャカンファレンス 2024」では、こうした課題へのヒントとなるテーマが数多く取り上げられていました。本記事では、その中でも特に印象的だった「トレードオフ分析」「組織とアーキテクチャの相互作用」「進化的アーキテクチャ」に焦点を当て、私が得た知見を共有できればと思います👋

<!-- textlint-enable -->

# トレードオフ分析

<!-- textlint-disable -->

トレードオフとは、複数の要件や目標が競合する中で、最適な選択を行うために他の選択肢を犠牲にする必要がある状況を指します。アーキテクチャ設計では、性能、コスト、スケーラビリティ、保守性など相反する要求をどうバランスさせるかが常に問われます。ここでは、トレードオフ分析で重要なポイントをいくつか紹介します！

<!-- textlint-enable -->

## 評価軸の重み付けは均一ではない

キーノートセッションでは、「**全ての評価軸が同等の重みを持つわけではない**」という洞察が共有されました。たとえば、性能とスケーラビリティで優れる案と、コストや保守性など他の多くの軸で優れる案があった場合、性能が最優先されるプロジェクトでは前者が選ばれるべきです。

このように意思決定時には全ての項目を平等に扱うのではなく、**ビジネス目標やプロジェクト背景に応じて評価軸の重み付けを調整すること**が不可欠です。

https://speakerdeck.com/findyinc/modern-trade-off-analysis

## トレードオフ分析は変化する

トレードオフ分析は一度行って終わりではなく、市場環境やステークホルダーの要求、技術進化などによって条件が変化するため、継続的な見直しが必要です。たとえば、短期的にはコスト削減が最優先される設計でも、事業が成長するにつれてスケーラビリティや保守性が重視されるケースは珍しくありません。

このようにトレードオフ分析は「**プロジェクトの成長とともに変化するべきプロセス**」です。

## 意思決定プロセスの透明性

トレードオフ分析を成功させるためには、意思決定プロセスを透明に保ち、関係者全員が評価基準や選択理由を理解できるようにすることが大切です。そのための有効な手法として 「**ADR（Architectural Decision Records）**」 があります。ADR は意思決定を体系的に記録し、その背景や理由を明確化するツールです。

たとえば ZOZO では ADR を活用し、過去の意思決定履歴を共有しています。これにより、新たな課題への迅速な対応や透明性向上につながっています。

https://docs.google.com/presentation/d/1ziV5yqWkqUZl6pCTmjGveB-QfgoSC_8BhYmpWy7mE6A/edit#slide=id.p1

## フィードバックループによる継続的改善

キーノートセッションでは、トレードオフ分析を支える仕組みとして「**フィードバックループの構築も重要である**」とも述べられていました。設計上の選択肢が運用にどう影響したかを振り返り、継続的に評価することで意思決定精度が向上します。

たとえばエラーレートやレスポンスタイムなど運用データを定期的に分析し、設計変更が実際の運用結果にどのような影響を与えたか数値で可視化することが推奨されていました。

# 組織とアーキテクチャの相互作用

組織とアーキテクチャは密接に関連しています。「**アーキテクチャはあくまでビジネス目標を達成するための手段**」であり、その目標に適合した設計を追求する必要があります。

## 組織目標との調和

<!-- textlint-disable -->

SanSan では、「**適応度関数（Fitness Function）**」を導入し、システム設計がビジネス要求に迅速に応えられるかを継続的に評価しています。たとえば、名刺管理システム「Bill One」の開発では、サイクルタイムに影響する保守性、テスト容易性、疎結合性といった品質特性を適応度関数で定量的に評価し、それに基づいて設計改善を行っていました。

<!-- textlint-enable -->

https://speakerdeck.com/sansantech/20241126

また、オープンロジでは、物流システムのリファクタリングとアーキテクチャ再構築において「**アーキテクチャそのものの良し悪しではなく、目的を達成できるかどうか**」に焦点を当てた設計方針を採用しています。具体的には、アクセスログを活用して未使用コードを特定・削除することでデッドコードを排除し、モジュール間の役割分担を明確化することで依存関係を整理していました。

https://speakerdeck.com/deeprain/wu-liu-sisutemuniokerurihuakutaringutoakitekutiyanozai-gou-zhu-yi-cun-guan-xi-tomoziyurufen-ge-nozhong-yao-xing

## コンウェイの法則とその適用

「**コンウェイの法則（システム設計は組織構造を反映する）**」は、多くの企業で実践的な指針として活用されています。

Bitkey では、イベントストーミングやモブプログラミングなどチーム全体が設計プロセスに参加する手法を採用し、組織構造と一致した効率的なアーキテクチャ設計を実現していました。

https://speakerdeck.com/bitkey/co-creating-architecture-a-sustainable-development-ecosystem-built-by-the-entire-team

また、ナレッジワークでは、「コンパウンド戦略」を推進する中で、複数プロダクトを同時展開可能なアーキテクチャへの移行を目的としたリアーキテクチャを実施しました。この過程で、各プロダクトチームが独立して進化できるようにアーキテクチャを分離しつつ、必要な部分では共通基盤を活用することで効率性を確保していました。

https://speakerdeck.com/kworkdev/technology-selection-and-rearchitecture-for-compounding-strategy

# 進化的アーキテクチャ

進化的アーキテクチャとは、システムが変化する要求や環境に柔軟に対応できるよう設計されるべきだという考え方です。このセクションでは、「シンプルさと柔軟性」「偶有的複雑性の最小化」に焦点を当てて解説します。

## シンプルさと柔軟性

オープンロジでは、10 年の歴史を持つ物流システムのリファクタリングとアーキテクチャ再構築において、変更容易性と拡張性を重視したシンプルな設計を採用しました。具体的には、アクセスログを活用して未使用コードを特定・削除することでデッドコードを排除し、システム全体の軽量化を図ったり、適切なモジュール化によって依存関係を整理していました。

https://speakerdeck.com/deeprain/wu-liu-sisutemuniokerurihuakutaringutoakitekutiyanozai-gou-zhu-yi-cun-guan-xi-tomoziyurufen-ge-nozhong-yao-xing

## 偶有的複雑性の最小化

<!-- textlint-disable -->

偶有的複雑性とは、本質的課題以外から発生する不要な複雑さ（例：冗長な依存関係や非効率な設計）を指します。ログラスでは、この偶有的複雑性排除に注力し、本質的課題解決への集中環境を整備しています。同社は「Enabling & Platform」部門を設置し、技術負債削減や運用コスト軽減など横断的課題への対応体制を構築していました。

<!-- textlint-enable -->

https://speakerdeck.com/knih/architectures-and-topologies

# おわりに

アーキテクチャカンファレンス 2024 では、「トレードオフ分析」「組織とアーキテクチャの相互作用」「進化的アーキテクチャ」といったテーマが多くのセッションで語られていました。直近でシステムのリプレースを進めている私にとって、多くの知見を得られる非常に有意義な機会となりました。

読者の皆さんも、本記事を通じてアーキテクチャに関する新たな視点やヒントを得ていただけていたら幸いです！

P.S. 来年もアーキテクチャカンファレンスが開催されるそうなので、是非参加してみようと思います！（記事を書くのが遅れ、今年になってしまいましたが…w）
