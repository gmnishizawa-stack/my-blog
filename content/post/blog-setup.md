+++
title = 'Cloudflare Pages + Hugoでブログを構築した記録'
date = 2025-10-28
description = 'このブログをどうやって作ったか、手順をまとめました'
summary = '今後の情報発信をするのに、ノートとかASPサービスも検討したのですが、Cloudflareをメインに扱うなら、Cloudflareのテクノロジーが使えるものでやったほうがいいかな、と思い「Cloudflare Pages」を使ってみることにしました。  ...'
+++

## はじめに

今後の情報発信をするのに、ノートとかASPサービスも検討したのですが、    
Cloudflareをメインに扱うなら、Cloudflareのテクノロジーが使えるものでやったほうがいいかな、と思い
「Cloudflare Pages」を使ってみることにしました。  
Hugoにはあまりこだわりがなく、Claudeのオススメに従いました。
本来ならそこそこ難しい作業かと思いますが、Claudeがあるのでいけるだろう、と思ってやったら出来ました。 
ということでこのブログを構築するまでの作業記録をまとめました。  
AIと一緒に進めるのがよいかと思いますので細かい手順までは書きませんがCloudflareアカウントをお持ちでしたら30分あればできると思います。

## 使用した技術

- **静的サイトジェネレーター**: Hugo
- **テーマ**: Clarity
- **バージョン管理**: Git / GitHub
- **ホスティング**: Cloudflare Pages
- **URL**: https://nishizawa.pages.dev/
- **アシスタント**: Claude (Anthropic) ← **重要!**

## 構築手順

### 1. 環境構築
- Hugoのインストール (winget)
- Gitのインストール

### 2. ブログサイト作成
- `hugo new site my-blog` でプロジェクト作成
- テーマを3回変更 (Ananke → PaperMod → Clarity)
- 設定ファイル (`hugo.toml`) を編集

### 3. GitHubにアップロード
- ローカルリポジトリ初期化
- GitHubにリモートリポジトリ作成
- プッシュ

### 4. Cloudflare Pagesでデプロイ
- GitHubと連携
- ビルド設定 (Hugo)
- 自動デプロイ設定

### 5. URL確保
- 最初は `my-blog-dz3.pages.dev`
- 後で `nishizawa.pages.dev` に変更

## 所感

Claudeと一緒だったからすべて順調でした。  

テーマ（ブログデザイン）を何回か変更したので余計に時間掛かりましたがそれでも30分くらいでした。  
pagesもHugoもGitもほぼ初めてでしたが、ただ言われたとおりに操作をしていくうちに「ああ、こういうものか」という感じでわかっていきます。  
普段慣れないコマンドラインの操作もコピペだけでどんどん作業が進みました。  
ドメイン名は「pages.dev」のままでいこうと思います、CloudflarePagesで作りました感をアピールして。  
CloudflarePagesで構築したので、なんといっても無料でできるのがイイところ（もちろん広告もありません）。  

今日時点（2025年10月27日）ではシンプルな構造ですが今後はサイドメニューは増築していきたいです。

## 参考リンク

- [Hugo公式サイト](https://gohugo.io/)
- [Cloudflare Pages](https://pages.cloudflare.com/)
- [Clarityテーマ](https://github.com/chipzoller/hugo-clarity)
- [Claude](https://claude.ai/)