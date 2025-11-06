+++
title = 'Cloudflareがどのようにトラフィックを評価するか'
date = 2025-11-06
summary = 'Cloudflareは、スコアリング、検出エンジン、ルールを組み合わせてトラフィックを評価します。Bot ScoreとWAF Attack Scoreの仕組み、Managed Rules・Custom Rules・Rate Limiting Rulesの役割、そして実際の判定プロセスを分かりやすく解説します。'
+++

# Cloudflareがどのようにトラフィックを評価するか

Cloudflareを使い始めると、「どうやってトラフィックを評価しているんだろう?」と疑問に思うことがあるかもしれません。実は、Cloudflareは複数の仕組みを組み合わせて、リクエストが自動化されたものか、攻撃的なものか、それとも正常なものかを判断しています。

この記事では、Cloudflareのトラフィック評価の仕組みについて解説します。

## スコアリング:リクエストに点数をつける

Cloudflareのトラフィック評価の中心にあるのが「スコアリング」という仕組みです。これは、やってくるリクエストそれぞれに点数をつけて、そのリクエストの性質を判断する方法です。

Cloudflareは目的に応じて2種類のスコアを使い分けています:
1. **Bot Score**: 自動化されているか、人間による操作かを判定
2. **WAF Attack Score**: 攻撃の可能性を判定

それぞれ詳しく見ていきましょう。

### Bot Score(ボットスコア)

Bot Scoreは、そのリクエストがボット(自動化されたプログラム)から来ているか、人間から来ているかを示す点数です。

- **スコアの範囲**: 1〜99
- **低いスコア(1に近い)**: 自動化されたプログラムの可能性が高い
- **高いスコア(99に近い)**: 人間がブラウザから操作している可能性が高い
- **一般的な閾値**: スコア30以下は自動化されたトラフィックと判断されることが多い

**重要な注意点**: Bot Scoreは「自動化されているかどうか」を判定するだけで、**そのリクエストが悪意のあるものかどうかは別問題**です。

たとえば、スコアが1のリクエストでも:
- **悪意のあるケース**: スクレイピングボット、ブルートフォース攻撃、DDoSボット
- **正当なケース**: 自社のAPI、監視ツール、パートナー企業のシステム、検索エンジンのクローラー

Bot Scoreを使う際は、正当な自動化トラフィックを誤ってブロックしないよう、適切な除外設定が必要です。

参考: [Bot scores - Cloudflare Docs](https://developers.cloudflare.com/bots/concepts/bot-score/)

### WAF Attack Score(WAF攻撃スコア)

WAF Attack Scoreは、そのリクエストがWebアプリケーションへの攻撃を含んでいる可能性を示す点数です。

- **スコアの範囲**: 1〜99
- **低いスコア(1に近い)**: 攻撃である可能性が高い
- **高いスコア(99に近い)**: 正常なリクエストの可能性が高い
- **推奨される閾値**: スコア20以下をブロックするのが初期設定として推奨される

このスコアは、SQLインジェクションやクロスサイトスクリプティング(XSS)など、従来のWAFマネージドルールでは捉えきれない攻撃のバリエーションを検出するために使われます。

参考: [WAF attack score - Cloudflare Docs](https://developers.cloudflare.com/waf/detections/attack-score/)

## スコアはどうやって計算されるのか?

Cloudflareは、複数の検出エンジンを組み合わせてスコアを算出しています。

### 主な検出エンジン

**1. ヒューリスティック検出**

既知の攻撃パターンや悪意のあるフィンガープリント(識別情報)のデータベースと照合する方法です。明らかに自動化されたリクエストと判断された場合、そのリクエストは**Bot Score 1**と判定されます。

**2. 機械学習(ML)**

Cloudflareのトラフィック評価の中核を担っています。

世界中で毎日何十億ものリクエストを処理するCloudflareは、この膨大なデータを使って機械学習モデルを訓練しています。このモデルがリクエストの様々な特徴を分析して、人間からのものか、ボットからのものか、攻撃的なものかを判断します。

- **継続的な学習**: 定期的にモデルが更新され、新しい脅威に対応
- **グローバルな知見**: 2600万以上のWebサイトからの情報を活用
- **高精度**: Bot ScoreとWAF Attack Scoreの大部分はこのエンジンで決定

参考: [Cloudflare Bot Management: machine learning and more](https://blog.cloudflare.com/cloudflare-bot-management-machine-learning-and-more/)

**3. その他の検出エンジン**

- **異常検知**: サイトごとの正常なトラフィックパターンを学習し、そこから外れたリクエストを検出(Enterprise Bot Management)
- **JavaScript検出**: クライアント側で軽量なJavaScriptを実行し、ヘッドレスブラウザなどを検出(Bot Management)

これらの検出エンジンが総合的に判断して、最終的なBot ScoreやWAF Attack Scoreが算出されます。

## ルールによる判定

スコアだけでなく、Cloudflareは「ルール」を使った判定も行います。ルールとは、「こういう条件のリクエストが来たら、こうする」という指示のことです。

### Managed Rules(マネージドルール)

Managed Rulesは、Cloudflareがあらかじめ用意した保護ルールのセットです。

**特徴:**
- SQLインジェクション、XSS、ゼロデイ脆弱性など、既知の攻撃パターンから保護
- Cloudflareが継続的に更新・メンテナンス
- ワンクリックで有効化できる
- OWASP Core Rulesetなど、業界標準のルールセットを含む

**主なマネージドルールセット:**
- **Cloudflare Managed Ruleset**: 広範な脆弱性から保護する基本セット
- **Cloudflare OWASP Core Ruleset**: OWASP Top 10の脆弱性に対応

Managed Rulesは、「既知の攻撃パターン」に対して効果的です。Cloudflareが攻撃手法を分析し、それに対応するルールを自動で更新してくれるため、利用者は最新の脅威に対して常に保護されます。

参考: [Managed Rules - Cloudflare Docs](https://developers.cloudflare.com/waf/managed-rules/)

### Custom Rules(カスタムルール)

Custom Rulesは、ユーザーが自分で作成する保護ルールです。

**特徴:**
- リクエストの様々な属性(URL、IPアドレス、国、ヘッダーなど)に基づいて条件を設定
- Bot ScoreやWAF Attack Scoreと組み合わせて使用可能
- サイト固有のニーズに合わせてカスタマイズできる

**カスタムルールでできること:**
- 特定の国からのアクセスをブロック
- ログインページへのアクセスに追加の保護を設定
- Bot Scoreが30以下のリクエストにチャレンジを表示(ただし正当な自動化トラフィックは除外)
- 特定のAPIエンドポイントを認証なしでアクセスできないようにする
- 自社のAPIや監視ツールなど、正当な自動化トラフィックを明示的に許可

参考: [Custom rules - Cloudflare Docs](https://developers.cloudflare.com/waf/custom-rules/)

**Managed RulesとCustom Rulesの使い分け:**

| 項目 | Managed Rules | Custom Rules |
|------|---------------|--------------|
| 設定方法 | ワンクリックで有効化 | 自分で条件とアクションを設定 |
| 保護対象 | 既知の一般的な攻撃 | サイト固有の要件 |
| メンテナンス | Cloudflareが自動更新 | ユーザーが管理 |
| 推奨される使い方 | 基本的な保護として必須 | 追加のカスタマイズ |

実際には、両方を組み合わせて使うのがベストプラクティスです。Managed Rulesで広範な攻撃から保護し、Custom Rulesでサイト特有のニーズに対応します。

### Rate Limiting Rules(レート制限ルール)

Rate Limiting Rulesは、一定期間内のリクエスト数を制限するルールです。

**基本的な仕組み:**
- 指定した期間内に、設定した回数以上のリクエストがあった場合に制限を発動
- 制限に引っかかったリクエストに対して、Block、Challenge、Logなどのアクションを実行
- IPアドレス、APIキー、ヘッダーなど、様々な特性でカウント可能

**レート制限の例:**
- ログインページへのアクセスを1分間に5回まで制限(ブルートフォース攻撃対策)
- API呼び出しを1時間に1000回まで制限(DDoS対策)
- 特定のリソースへのアクセスを10分間に100回まで制限

Rate Limiting Rulesは、自動化された攻撃、特にブルートフォース攻撃やDDoS攻撃に対して非常に効果的です。

参考: [Rate limiting rules - Cloudflare Docs](https://developers.cloudflare.com/waf/rate-limiting-rules/)

## 実際の判定プロセス

リクエストがCloudflareに到達すると、以下のような流れで評価されます。

### 1. スコアの算出

リクエストがCloudflareのエッジサーバーに到達した瞬間に、複数の検出エンジンが並行して動作してスコアを算出します: 

- ヒューリスティックチェックで既知の悪意のあるパターンと照合
- 機械学習モデルがリクエストの特徴を分析
- 異常検知エンジンが正常なトラフィックパターンとの差異を検出
- JavaScript検出(有効な場合)がクライアント側の振る舞いを確認

これらの検出エンジンの結果を総合して、**Bot Score**や**WAF Attack Score**が算出されます。このスコアはリクエストに付与され、後続のルール評価で使用できるようになります。

### 2. ルールの評価

算出されたスコアとリクエストの特性を使って、設定されているルールが**以下の順番**で評価されます:

**評価順序:**
1. **Custom Rules(カスタムルール)** - アカウントレベル
2. **Custom Rules(カスタムルール)** - ゾーンレベル
3. **Rate Limiting Rules(レート制限ルール)** - アカウントレベル
4. **Rate Limiting Rules(レート制限ルール)** - ゾーンレベル
5. **Managed Rules(マネージドルール)** - アカウントレベル
6. **Managed Rules(マネージドルール)** - ゾーンレベル

参考: [WAF phases - Cloudflare Docs](https://developers.cloudflare.com/waf/reference/phases/)

**重要なポイント:**
- **Custom Rulesが最初に評価される**ため、ここでBot ScoreやWAF Attack Scoreを使った判定ができます
- 各レベル内では、ルールは上から順番に評価されます
- BlockやChallengeなどの**終端アクション**が実行されると、それ以降のルールは評価されません
- Logアクションなどの**非終端アクション**は、後続のルールの評価を止めません

**注意:**
- **Bot Fight Mode / Super Bot Fight Mode**は、Custom Rulesよりも**前**に実行されます
- そのため、Bot Fight Modeが有効な場合、Custom Rulesでブロックしたいリクエストが先にManaged Challengeを受ける可能性があります
- Pro以上のプランでは、Custom RulesのSkipアクションを使ってSuper Bot Fight Modeを特定のパスで回避できます

参考: [Get started with Super Bot Fight Mode - Cloudflare Docs](https://developers.cloudflare.com/bots/get-started/super-bot-fight-mode/)

### 3. アクションの実行

ルールにマッチした場合、以下のようなアクションが実行されます:

- **Block**: リクエストをブロックし、エラーページを表示(終端アクション)
- **Managed Challenge**: 自動的にチャレンジを実行。ボットはブロック、人間は通過(終端アクション)
- **JS Challenge**: JavaScriptチャレンジを実行。ブラウザ以外はブロック(終端アクション)
- **Challenge (CAPTCHA)**: CAPTCHAを表示(終端アクション)
- **Log**: ログに記録するだけで、リクエストは通過(非終端アクション)
- **Skip**: 特定のセキュリティ機能をスキップ(非終端アクション)
- **Allow**: リクエストを許可し、同じphase内の以降のルールをスキップ(終端アクション)

### 4. オリジンサーバーへの転送

セキュリティチェックをすべて通過したリクエストだけが、最終的にオリジンサーバーに転送されます。

### 実行順序の具体例

例えば、以下のような設定がある場合:

1. Custom Rule: Bot Score < 30 かつ Verified Botでない かつ APIパスでない なら Block
2. Rate Limiting Rule: 1分間に100リクエスト以上なら Challenge
3. Managed Rule: SQLインジェクションパターンを検出したら Block

リクエストの処理順序(悪意のあるボットの場合):
1. まずBot Scoreが算出される(例: スコア5)
2. Custom Ruleが評価される → Bot Score 5 < 30、Verified Botではない、APIパスでもない → **Block**
3. この時点で終端アクションが実行されるため、Rate Limiting RuleやManaged Ruleは評価されない

リクエストの処理順序(自社APIの場合):
1. Bot Scoreが算出される(例: スコア5)
2. Custom Ruleが評価される → Bot Score 5 < 30 だが、APIパス(/api/)へのアクセス → **マッチせず通過**
3. Rate Limiting Ruleが評価される
4. Managed Ruleが評価される
5. 問題がなければオリジンへ転送

リクエストの処理順序(通常の人間の場合):
1. Bot Scoreが算出される(例: スコア85)
2. Custom Ruleは通過(Bot Score 85 > 30)
3. Rate Limiting Ruleが評価される
4. Managed Ruleが評価される
5. どこかでマッチすればそのアクションが実行され、マッチしなければオリジンへ転送

## 実務での活用方法

### ログでトラフィックを確認する

Cloudflareダッシュボードで、どのようなリクエストがブロックされたか、どう評価されたかを確認できます:

- **Security Events**: ブロックやチャレンジが発動したイベント
- **Bot Analytics**: ボットトラフィックの詳細な分析
- **Security Analytics**: 全体的なセキュリティイベントの概要

これらのログを見ることで、実際にどんなトラフィックが来ているのか、設定したルールが正しく機能しているかを確認できます。

## まとめ

Cloudflareのトラフィック評価は、以下の要素を組み合わせた多層的な仕組みです:

**スコアリング:**
- Bot Score: 自動化されているか、人間による操作かを判定(1〜99)
- WAF Attack Score: 攻撃の可能性を判定(1〜99)

**検出エンジン:**
- ヒューリスティック: 既知のパターンと照合
- 機械学習: 膨大なデータから学習したモデルで判定
- 異常検知: サイトごとの正常な振る舞いからの逸脱を検出
- JavaScript検出: クライアント側の振る舞いを分析

**ルールベースの判定:**
- Managed Rules: Cloudflareが用意した保護ルール
- Custom Rules: 独自の条件で作成するルール
- Rate Limiting Rules: リクエスト数に基づく制限

Cloudflareのトラフィック評価は、世界中のトラフィックから学習し、常に進化しています。これらの仕組みを理解することで、Webサイトやアプリケーションをより効果的に保護できます。

## 参考リンク

この記事の内容は、以下のCloudflare公式ドキュメントとブログ記事を参考にしています:

**Bot Management関連:**
- [Bot scores - Cloudflare Docs](https://developers.cloudflare.com/bots/concepts/bot-score/)
- [Bot Management - Cloudflare Docs](https://developers.cloudflare.com/bots/get-started/bm-subscription/)
- [Get started with Super Bot Fight Mode - Cloudflare Docs](https://developers.cloudflare.com/bots/get-started/super-bot-fight-mode/)
- [Cloudflare Bot Management: machine learning and more](https://blog.cloudflare.com/cloudflare-bot-management-machine-learning-and-more/)

**WAF Attack Score関連:**
- [WAF attack score - Cloudflare Docs](https://developers.cloudflare.com/waf/detections/attack-score/)

**Managed Rules関連:**
- [Managed Rules - Cloudflare Docs](https://developers.cloudflare.com/waf/managed-rules/)
- [Cloudflare Managed Ruleset - Cloudflare Docs](https://developers.cloudflare.com/waf/managed-rules/reference/cloudflare-managed-ruleset/)

**Custom Rules関連:**
- [Custom rules - Cloudflare Docs](https://developers.cloudflare.com/waf/custom-rules/)
- [Challenge bad bots - Cloudflare Docs](https://developers.cloudflare.com/firewall/recipes/challenge-bad-bots/)

**Rate Limiting関連:**
- [Rate limiting rules - Cloudflare Docs](https://developers.cloudflare.com/waf/rate-limiting-rules/)
- [Rate limiting parameters - Cloudflare Docs](https://developers.cloudflare.com/waf/rate-limiting-rules/parameters/)
- [Introducing Advanced Rate Limiting](https://blog.cloudflare.com/advanced-rate-limiting/)

**実行順序関連:**
- [WAF phases - Cloudflare Docs](https://developers.cloudflare.com/waf/reference/phases/)
- [Phases list - Cloudflare Docs](https://developers.cloudflare.com/ruleset-engine/reference/phases-list/)

より詳しい情報や最新のアップデートについては、[Cloudflare公式ドキュメント](https://developers.cloudflare.com/)をご確認ください。
