---
title: "Cloudflareのマネージドルールとは？仕組みを公式ドキュメントから理解する"
date: 2025-12-19
tags: ["Cloudflare", "WAF", "セキュリティ", "マネージドルール", "公式ドキュメント"]
summary: "CloudflareのWAF機能を使う上で必ず目にする「マネージドルール」。一体どんな仕組みで、どう更新されているのか？日々の実装・運用業務で必要になる正確な理解を、公式ドキュメントを参照しながら整理しました。自動更新のサイクルから、カスタマイズの自由度、デフォルト設定の意図まで、実務で役立つポイントを解説します。"
---

今回はCloudflareのWAF機能の中核である「マネージドルール」について、公式ドキュメントに基づいて解説します。

## マネージドルールとは

Cloudflareの公式ドキュメントによると、マネージドルールは**「Cloudflareが事前に設定・提供するルールセット」**です。

> "Managed rulesets are preconfigured rulesets provided by Cloudflare that you can deploy. Only Cloudflare can modify these rulesets."
> 
> (出典: [Cloudflare Ruleset Engine - Work with managed rulesets](https://developers.cloudflare.com/ruleset-engine/managed-rulesets/))

つまり、ユーザーが一からWAFルールを作る必要はなく、Cloudflareが用意した「攻撃パターン集」を有効化するだけで、Webアプリケーションを保護できる仕組みです。

## 主なマネージドルールセット

Cloudflareは以下の主要なマネージドルールセットを提供しています(公式ドキュメントより):

### 1. **Cloudflare Managed Ruleset**
> "Created by the Cloudflare security team, this ruleset provides fast and effective protection for all of your applications. The ruleset is updated frequently to cover new vulnerabilities and reduce false positives."
> 
> (出典: [Managed Rules - Cloudflare WAF](https://developers.cloudflare.com/waf/managed-rules/))

Cloudflareのセキュリティチームが作成した汎用的な攻撃対策ルールセットです。CVE(共通脆弱性識別子)や既知の攻撃手法に対する保護を提供します。

### 2. **Cloudflare OWASP Core Ruleset**
> "Cloudflare's implementation of the Open Web Application Security Project, or OWASP ModSecurity Core Rule Set. Cloudflare routinely monitors for updates from OWASP based on the latest version available from the official code repository."
> 
> (出典: [Managed Rules - Cloudflare WAF](https://developers.cloudflare.com/waf/managed-rules/))

OWASP Top 10に基づいた攻撃対策を提供するルールセットです。

### 3. **Cloudflare Exposed Credentials Check**
漏洩した認証情報データベースと照合し、盗まれた認証情報の使用を検知します。

## 自動更新の仕組み

マネージドルールの大きな特徴は、**Cloudflareが自動的にルールを更新してくれる**点です。

公式の変更履歴(Changelog)ページには以下の記載があります:

> "The release cycle for new rules happens on a 7-day cycle, typically every Monday or Tuesday depending on public holidays."
> 
> (出典: [Changelog for managed rulesets](https://developers.cloudflare.com/waf/change-log/))

つまり、通常は**7日サイクル(毎週月曜日または火曜日)**でルールが更新されます。

また、緊急の脆弱性対応時には:

> "Cloudflare may also add rules at any time during emergency releases for high profile zero-day protection."
> 
> (出典: [Get started - Cloudflare WAF](https://developers.cloudflare.com/waf/get-started/))

このように、重大なゼロデイ脆弱性に対しては、定期サイクル外でも緊急リリースが行われます。

## ルールの動作フロー

公式ドキュメントによると、マネージドルールは以下のように動作します:

1. **リクエストの受信**: Cloudflareがリクエストを受け取る
2. **ルールの照合**: マネージドルールセット内の各ルールとリクエストを照合
3. **アクションの実行**: マッチした場合、設定されたアクション(Block、Managed Challenge、Logなど)を実行

## カスタマイズ性

「マネージド(管理された)」という名前ですが、完全な「お任せ」ではありません。公式ドキュメントでは以下のカスタマイズ方法が説明されています:

> "To customize the behavior of managed rulesets, do one of the following:
> - Create exceptions to skip the execution of managed rulesets or some of their rules under certain conditions.
> - Configure overrides to change the rule action or disable one or more rules of managed rulesets."
> 
> (出典: [Managed Rules - Cloudflare WAF](https://developers.cloudflare.com/waf/managed-rules/))

具体的には:
- **例外設定(Exceptions)**: 特定条件下でルールをスキップ
- **オーバーライド(Overrides)**: 個別ルールのアクション変更や無効化
- **ルールセット単位の設定**: ルールセット全体のアクション変更
- **タグによる一括設定**: WordPressやJoomlaなど、特定の技術スタックに関連するルールをまとめて有効化

## デフォルト設定の考え方

興味深いことに、Cloudflare Managed Rulesetは**一部のルールがデフォルトで無効**になっています。

> "Some rules in the Cloudflare Managed Ruleset are disabled by default, intending to strike a balance between providing the right protection and reducing the number of false positives."
> 
> (出典: [Cloudflare Managed Ruleset](https://developers.cloudflare.com/waf/managed-rules/reference/cloudflare-managed-ruleset/))

これは**保護と誤検知のバランス**を考慮した設計です。すべてのルールを有効化すると、正当なトラフィックまでブロックしてしまう可能性があるため、最初は誤検知率の低いルールだけを有効化し、ユーザーが段階的にチューニングできるようにしています。

## 新ルールのロールアウトプロセス

Cloudflareは新しいルールを追加する際、慎重なプロセスを踏んでいます:

> "Cloudflare will deploy the updated or new rules into logging only mode (Log action), with beta and new tags."
> 
> (出典: [Changelog for managed rulesets](https://developers.cloudflare.com/waf/change-log/))

1. **最初はLog(記録のみ)モード**で新ルールをデプロイ
2. ユーザーが誤検知をチェックできる期間を設ける
3. 問題がなければ、後のリリースサイクルでBlock等の実際のアクションに変更

この段階的なアプローチにより、新ルールによる影響を最小限に抑えています。

## まとめ

Cloudflareのマネージドルールは:

- Cloudflareが提供・管理する事前設定済みのWAFルールセット
- 7日サイクル(通常毎週月・火曜日)で自動更新される
- 緊急時にはゼロデイ対応として即座にルールが追加される
- デフォルトでは誤検知を減らすため一部ルールが無効化されている
- 例外設定やオーバーライドで柔軟にカスタマイズ可能

「自分でルールを書かなくても基本的な保護が得られる」という手軽さと、「必要に応じて細かく調整できる」という柔軟性を両立した仕組みです。

**参考情報**
- [Cloudflare WAF - Managed Rules 公式ドキュメント](https://developers.cloudflare.com/waf/managed-rules/)
- [WAF Changelog](https://developers.cloudflare.com/waf/change-log/)