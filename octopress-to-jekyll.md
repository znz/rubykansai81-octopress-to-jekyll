# ブログを Octopress 2 + GitHub Pages から Jekyll 3 + AMP + Netlify に移行した話

author
:   Kazuhiro NISHIYAMA

content-source
:   第81回 Ruby関西 勉強会

date
:   2018/03/17

allotted-time
:   15m

theme
:   lightning-simple

# 自己紹介

- Ruby コミッターなど
- Twitter, GitHub: `@znz`

# Jekyll とは?

- markdown などから静的なサイトを生成するもの
- GitHub Pages などでホスティングできる
- 動的なコンテンツは JavaScript や外部サイトを使う
- <https://jekyllrb.com/>

# Octopress とは?

- Jekyll 2.x ベースのブログ生成ソフト
- Octopress 3.0 も開発されたが止まっている
  - そのため Octopress のバージョンアップではなく Jekyll 直接に移行
- <https://github.com/imathis/octopress>

# GitHub Pages とは?

- GitHub の静的ファイルホスティングサービス
- git push するだけで反映される
- jekyll などの自動ビルドも可能
  - プラグインなどが制限されている
  - 手元や CI で生成して生成物を push も可能

# GitHub Pages とは?

- 独自ドメインはそのままだと https 対応できない
  - Cloudflare などの CDN と組み合わせる例が多い

# Netlify とは?

- 静的なサイトのホスティングサービス
  - 動的なもの (問い合わせフォームなど) も少し対応しているらしい (未使用)
- Git 連携あり
- CDN あり
- 独自ドメインでも https 対応可能

# Amplify for Jekyll とは?

- Jekyll の AMP (Accelerated Mobile Pages) 対応テーマ
- <https://github.com/ageitgey/amplify>
- 基本的に AMP のみのサイト用
- amp-jekyll gem (プラグイン) とは別
  - <https://github.com/juusaw/amp-jekyll>

# AMP とは?

- Accelerated Mobile Pages の略
- 高速なモバイル対応サイトを作れる仕組み
- 制限された AMP HTML で書く
  - 動的なものは AMP 側で対応しているもののみ
  - amp-iframe である程度は独自の javascript も可能
- <https://www.ampproject.org/ja/>

# なぜ amplify?

- ほぼ文字コンテンツだけなので AMP 対応だけで良い
  - 別対応は面倒
- amp-jekyll gem は AMP 対応は別 URL で生成

# なぜ amplify?

- 自前でテーマを作るのは大変そうだった
  - AMP ベースだけだとマージンがなくなるとか素の HTML よりみづらくなる
  - コードの highlight の CSS とか欲しかった

# 移行措置

# URL 維持の変更

- `categories` を `tags` にして `category: blog` を追加
  - カテゴリが URL の一部になるため
- permalink 設定も互換性があるように変更
- カテゴリ別ページはリダイレクトで対応
- pagination は直接ブックマークやリンクなどはないだろうと思って何もせず

# 画像対応

- 少しだけ使っていたので対応
- amp-img (width, height 必須) に書き換え
  - amp-jekyll から `amp_filter.rb` だけ利用
    - サイズ自動埋め込み
    - ファイル名のミスが検知できる
    - nokogiri と fastimage を Gemfile に追加

# スライド埋め込み

- <https://slide.rabbit-shocker.org/> から埋め込み
- iframe から amp-iframe に変更
  - これも width と height 必須だが固定値で良い
  - https 必須だったので古い http の URL は書き換え

# アマゾンの書影

- これも iframe から amp-iframe に変更
- これも width と height 必須だが固定値で良い
- allow-popups も必要だった
  - ないと表示だけでクリックしても開けない

# Google Analytics

- amp-analytics に変更
- AMP 側で対応があるので、普通のページに埋め込むより楽な点もある

# Google AdSense

- amp-ad に変更
- なぜか空白しか表示されていない…
- `jekyll.environment` で分岐してローカルでは非表示

# google カスタム検索

- サイト内検索
- 終了予定なので削除
- 使われてなさそうだったので代替も設置せず

# social share

- Twitter は amplify でリンクがあった
- 他も含めて zenback から amp-social-share に変更
- amp-social-share が組み込み対応していないものも data-share-endpoint 指定で対応可能

# feed.xml

- octopress デフォルトの atom.xml から amplify デフォルトに名前変更
- リダイレクトを設定
  - フィードリーダーに影響はないはず

# GitHub Pages から Netlify

- 主に https 対応のため
- github-pages gem (177) は jekyll のバージョンが古かった (3.6.2)
  - 最新は jekyll 3.7.3
  - octopress の頃と同様に手元で生成する手もある

# JEKYLL_ENV 設定

- Netlify の Web 画面からでは staging と production を分けられず
- netlify.toml で設定

```toml
[context.production.environment]
JEKYLL_ENV = "production"
[context.deploy-preview.environment]
JEKYLL_ENV = "preview"
[context.branch-deploy.environment]
JEKYLL_ENV = "staging"
```

# 使ったプラグイン (1/2)

- jekyll-paginate
  - jekyll-paginate-v2 は互換性がなくなる予定だったので不採用
- jekyll-compose
  - amplify の Gemfile に入っていたので試し中
- jekyll-tagging
  - タグごとのページ生成用

# 使ったプラグイン (2/2)

- jekyll-minifier
  - カスタム CSS が全ページに埋め込みなのに無駄に大きい気がしたので
  - style amp-boilerplate が変わらないのは確認済み
- jekyll-sitemap
  - 必要性はよくわからないが一応追加

# まとめ

- 開発が止まっている Octopress 2 から Jekyll 3 に移行
- ついでに AMP 対応と https 対応
  - https は AMP に必須だった
- GitHub Pages から Netlify に移行
  - 手元で生成をやめた
