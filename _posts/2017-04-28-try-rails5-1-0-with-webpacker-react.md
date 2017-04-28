---
layout: blog
title: Rails5.1.0で--webpack=reactを試してみた
category: blog
tags: [Rails,React,Webpack]
summary: 12th RailsConf in Phoenix に合わせてリリースされたRails5.1.0を試してみました
author: yamagata
---

mofmofでRailsエンジニアをしている山形です。
好きな領域はフロントエンドで、もともとWebデザインをやっていたバックグラウンドを活かしてUIデザインを含めた開発をやっていたりします。

さて本日（US時間2017.4.27）Railsの新しいバージョン5.1.0がリリースされました。
Phoenixで行われているRailsConfに合わせて公開されたようです。

[Rails 5.1: Loving JavaScript, System Tests, Encrypted Secrets, and more](http://weblog.rubyonrails.org/2017/4/27/Rails-5-1-final/)

主な新機能としては

- __Loving JavaScript__
- System tests
- Encrypted secrets
- Parameterized mailers
- Direct & resolved routes
- Unify form_tag/form_for with form_with
- Everything else (hundreds of other fixes and improvements)

などが挙げられていますが、特に注目のやっと最近のJavaScript技術が取り入れられたwebpackerを試してみました。

## [rails new --webpack=react] vs [rails webpacker:install]

webpackerを使い始める方法は２つあるようです

１つめは`rails new my-app --webpack=react` newする時に指定する方法。
こうすると一連のディレクトリ作成 〜 `bundle install` 〜 `yarn install` までが一気に行われました。

２つめは既に作成済みアプリのGemfileに `gem 'webpacker', github: 'rails/webpacker'` を追加してinstall後、
`bin/rails webpacker:install:react` というコマンドでセットアップする方法です。

`bin/rails webpacker` のコマンド一覧はこちらです

```
Available webpacker tasks are:
webpacker:install             Installs and setup webpack with yarn
webpacker:compile             Compiles webpack bundles based on environment
webpacker:check_node          Verifies if Node.js is installed
webpacker:check_yarn          Verifies if yarn is installed
webpacker:verify_install      Verifies if webpacker is installed
webpacker:yarn_install        Support for older Rails versions. Install all JavaScript dependencies as specified via Yarn
webpacker:install:react       Installs and setup example react component
webpacker:install:vue         Installs and setup example vue component
webpacker:install:angular     Installs and setup example angular2 component
```

うっかりnewする時にオプションを忘れても安心ですね。


## bin/webpack-dev-serverで開発

ブラウザで確認する時は `bin/rails s` の他にもう一つ別のプロセスで `bin/webpack-dev-server` が必要なようです。
僕は今までもこれと同じように `npm run watch` を叩いてファイルの変更を監視していたので変更点はコマンドが変わったくらいですね

## app/javascript/packs が追加された

これ地味に嬉しいですね。
今まではes5なjsと分けるために `app/assets/javascripts/es2015` の中にさらに `components` とか `api` 等のフォルダを用意していたので頻繁に作業する割に階層が深いのがストレスでした。

## javascript_pack_tagヘルパー

で肝心のviewに表示する方法ですが `javascript_pack_tag` を使うようです。
デフォルトではlayoutにもこれは書かれていないので自分で追加する必要があります。

このヘルパーで読み込めるjsは必ず `app/javascript/packs` 以下に置く必要がありました。
`app/javascript` に置いたものはエラーしてしまいました。

ただし `app/javascript/packs` に置いたファイルから相対パスで親ディレクトリを参照しimportするのは可能でした。

```js
// app/javascript/test.js を配置して置いて
// app/javascript/packs/hello.js から以下を実行するのは可能
import test from "../test";
```

なのでpacks以下にはエントリーポイントのみを配置して、コンポーネントなどは `app/javascript/components` に入れるのもいいかもしれないです

## ライブリロードされる

webpack-dev-serverを起動した状態で、使用しているjsファイルをsaveすると自動でブラウザがリロードされました。
consoleに何かログが出ているんですが、すぐにリフレッシュされちゃうのでなんて書いてあるのかは読み取れず…

## ライブラリの使用

普通に `yarn add lodash` とかでいけます。
しかもwebpack-dev-serverを再起動する必要はありませんでした。

## Build

どうやら `assets:precompile` のhookで `webpacker:compile` が行われるようです。
以下ログです

```
bin/rake assets:precompile
Running via Spring preloader in process 17360
yarn install v0.17.9
[1/4] 🔍  Resolving packages...
success Already up-to-date.
✨  Done in 0.45s.
I, [2017-04-28T11:42:49.233684 #17360]  INFO -- : Writing /Users/kozo/tmp/rails510-test/public/assets/application-2928582a8241efa0ad750427fec796f54b00727b9b43b91472f2e0ff8722876f.js
I, [2017-04-28T11:42:49.243980 #17360]  INFO -- : Writing /Users/kozo/tmp/rails510-test/public/assets/application-2928582a8241efa0ad750427fec796f54b00727b9b43b91472f2e0ff8722876f.js.gz
I, [2017-04-28T11:42:49.247712 #17360]  INFO -- : Writing /Users/kozo/tmp/rails510-test/public/assets/application-f0d704deea029cf000697e2c0181ec173a1b474645466ed843eb5ee7bb215794.css
I, [2017-04-28T11:42:49.247948 #17360]  INFO -- : Writing /Users/kozo/tmp/rails510-test/public/assets/application-f0d704deea029cf000697e2c0181ec173a1b474645466ed843eb5ee7bb215794.css.gz
I, [2017-04-28T11:42:49.586327 #17360]  INFO -- : Writing /Users/kozo/tmp/rails510-test/public/assets/express/lib/application-29aaa7b87a3a289ad011fd55b0e54f83a6f929e6b4cefefd52d7e29484408a66.js
I, [2017-04-28T11:42:49.586541 #17360]  INFO -- : Writing /Users/kozo/tmp/rails510-test/public/assets/express/lib/application-29aaa7b87a3a289ad011fd55b0e54f83a6f929e6b4cefefd52d7e29484408a66.js.gz
Webpacker is installed 🎉 🍰
Using /Users/kozo/tmp/rails510-test/config/webpack/paths.yml file for setting up webpack paths
Compiling webpacker assets 🎉
Compiled digests for all packs in /Users/kozo/tmp/rails510-test/public/packs:
{"application.js"=>"http://localhost:8080/packs/application.js", "application.js.map"=>"http://localhost:8080/packs/application.js.map", "hello_react.js"=>"http://localhost:8080/packs/hello_react.js", "hello_react.js.map"=>"http://localhost:8080/packs/hello_react.js.map"}
```

`public/packs` 以下に作られるんですねー

## 画像とかcssも扱えるっぽい

今回試していないですが、画像とかcssをjs内にimport出来るっぽいです。
webpackなので当たり前なのですが、事前情報ではjsのみと聞いていたような気がするのでいつの間に対応したんですかね。

## まとめ

いままでのrailsのフロントエンドの苦労は一体何だったのか、俺達の努力は・・・という気持ちになるくらい一気に最新の環境にジャンプしましたね。
そして巡り巡ってRailsのフロントエンドにemberを統合させたかった(?)Yehuda先生もyarnというプロダクトで今回のアップデートに貢献しているとこが胸アツですね。

既存のrailsアプリにもwebpacker入れてみたくなりました。
