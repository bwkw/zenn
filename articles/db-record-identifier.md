---
title: "他のレコードと違う存在になりたいレコードへ"
emoji: "🤡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["DB", "レコード", "識別子"]
published: false
---

# TL;DR

# はじめに

データーベースにおいて、データを一意に識別する手段は、システムの信頼性と効率の基盤となります。多くのアプリケーションフレームワークでは、シンプルで実績のある「Auto Increment」方式をデフォルトの識別子として採用しています。これは、シーケンシャルに増加する数値を使い、データベースの各レコードにユニークなIDを割り振る方法です。この単純さが、特に単一のデータベースサーバーを用いる場合や小規模なアプリケーションにおいては、その強みとなっています。

しかし、現代のアプリケーションは、単純なブログやウェブショップを超えて、分散システム、マイクロサービスアーキテクチャ、クラウドベースのスケーラビリティを要求されることが増えています。このような複雑なシステムでは、Auto IncrementのIDが持つ限界が顕著になり始めます。スケールアウトや複数の書き込みノードを持つ環境では、IDの衝突を避けるための追加的な工夫が必要となり、その管理が煩雑になることがあります。また、IDを予測可能なシーケンスにすることは、セキュリティー上のリスクを招く可能性もあります。

これらの課題を解決するために、UUID、ULID、Snowflakeのような新しい形式の識別子が提案されてきました。これらの技術は、ユニークさ、スケーラビリティ、生成の予測不可能性など、特定の利点を提供し、より要求の厳しい現代のアプリケーションのニーズに応えることを目的としています。本記事では、これらの識別子がどのように機能し、それぞれがどのような状況で最適であるかを解き明かしていきます。

# データ識別子

## Auto Increment

| id  | name  |
| --- | ----- |
| 1   | Alice |
| 2   | Bob   |
| 3   | Peter |
| 4   | Tom   |

### 特徴

- 整数型で表され、通常1から始まり、新しいエントリーが追加されるごとに1ずつ増加します

### Pros

1. **簡単な実装**
   - フレームワークが提供する機能を活用し、テーブルに自動で連番IDを設定できます。これにより、開発者は手動でIDを管理する手間を省くことができます
2. **パフォーマンスの向上**
   - MySQL (InnoDB) を使用する際、整数型のIDを採用すると、UUIDと比べてデータサイズが小さくなります。これにより、B木の高さが低くなり、データベースの検索パフォーマンスが向上します。
   - 時系列でデータを取得する際には、セカンダリインデックスを介してPKを検索し、最終的にリーフノードを特定します。このプロセスでは、セカンダリインデックスの追加があっても一定のオーバーヘッドが発生し、パフォーマンスに数十パーセントの差が出ることがあります。
3. **外部公開時の利便性**
   - 連番IDはURLとして使用する際に便利です。外部に露出させる際に、直感的で分かりやすい形式を提供できます。

### Cons

## UUID v4

| id                                   | name  |
| ------------------------------------ | ----- |
| 550e8400-e29b-41d4-a716-446655440000 | Alice |
| 123e4567-e89b-12d3-a456-426614174000 | Bob   |
| 0c74f13f-fa83-4c48-9b33-689fed9a34b7 | Peter |
| 5f9c7df8-3d59-4846-99fe-c4518e82c86f | Tom   |

### 特徴

- 128ビット長で、8-4-4-4-12の形式に分かれた16進表記を用いて表されます
- 全世界でユニークな値である可能性が非常に高く、衝突するリスクは極めて低いです

### Pros

### Cons

## ULID

| id                         | name  |
| -------------------------- | ----- |
| 01ARZ3NDEKTSV4RRFFQ69G5FAV | Alice |
| 01ARYZ6S41TSV4RRFFQ69G5FAV | Bob   |
| 01AS3YPYXJTSV4RRFFQ69G5FAV | Peter |
| 01B2M2Y8JPTSV4RRFFQ69G5FAV | Tom   |

### 特徴

- 最初の48ビットはタイムスタンプで、残りの80ビットはランダムな値です
- Base32でエンコードされ、結果として26文字のアルファベットと数字のみで構成されます
- 時系列情報が組み込まれており、生成順にソートが可能です

### Pros

### Cons

## Snowflake

| id                  | name  |
| ------------------- | ----- |
| 1382971839180339201 | Alice |
| 1382971839180339202 | Bob   |
| 1382971839180339203 | Peter |
| 1382971839180339204 | Tom   |

### 特徴

- Twitterによって開発された方式（[参考](https://github.com/twitter-archive/snowflake)）で、64ビットの整数で表されます
- タイムスタンプ、データセンターまたはワーカーノードのID、シーケンス番号の情報が含まれています
- 時系列情報が組み込まれているため、生成された順番に沿ってソート可能です

### Pros

### Cons

# データ識別子の比較

# ユースケース

# おわりに

# 参考文献