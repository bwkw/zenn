---
title: "OIDC完全に理解した"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["認証", "認可", "OIDC"]
published: false
---

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

# 実践編
## ALBでOIDCを使用して、ユーザを認証する方法
## GitHubActionsでOIDCを使用して、AWS認証を行う方法

# おわりに
