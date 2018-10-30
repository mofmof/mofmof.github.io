---
layout: blog
title: Google Cloud Vision APIで写真がラーメンかどうかを判定できそうか調べてみた
category: blog
tags: [機械学習,machine learning,画像分類,Cloud Vision]
summary: とりあえず手っ取り早くラーメン画像を識別出来そうか試してみたかった。
author: aharada
image: /images/blog/2018-10-30-google-cloud-vision/ramen1.png
---

とりあえず手っ取り早くラーメン画像を識別出来そうか試してみたかった。どうやら画面から画像アップしたらAPIの出力結果が見えるようになっている親切な仕組みがあったので、まずはこれであたりをつける。

[Vision API - 画像コンテンツ分析](https://cloud.google.com/vision/)

`Labels`よりも`Web`の方がはっきりラーメンであるという結果を返してくれるのでこっちを使ったほうが良さそうだ。

![ラーメン画像1](/images/blog/2018-10-30-google-cloud-vision/ramen1.png)

二郎系のラーメンではどうだろうか？こちらも問題なく`Ramen 1.089`とトップのスコアで返ってきた。

![ラーメン画像2](/images/blog/2018-10-30-google-cloud-vision/ramen2.png)

うどんはどうだろうか？`Udon 0.7214`と微妙なスコアを返してはいるが、`Ramen`とは認識されていないので問題なさそう。

![うどん画像](/images/blog/2018-10-30-google-cloud-vision/udon.png)

念のためスパゲッティも試してみる。`Spaghetti alle vongole 4.26471`とボンゴレ・ビアンコであることも当てた。素晴らしい。こちらも`Ramen`は入っていないのでOK。

![スパゲッティ画像](/images/blog/2018-10-30-google-cloud-vision/spa.png)

最後にソーキそば。`Soki 1.0794`と微妙なスコアだが`Ramen`は入っていないので問題なさそう。ところで`Nakamura Soba`ってなんだろう。

![ソーキそば画像](/images/blog/2018-10-30-google-cloud-vision/soki.png)

結論としては、Cloud Vision APIのWEBエンティティを使えばラーメン画像であるか否かは判別出来そう。

## APIから叩く(Ruby)

GCPのコンソール画面から、プロジェクトを作成し、Cloud Vision APIを有効にする。

![Cloud Visionを有効に](/images/blog/2018-10-30-google-cloud-vision/enable.png)

Cloud Vision APIも他APIと同様にリクエスト割当を制限出来るので、無料の範囲で試すのも問題なさそう。

![割当](/images/blog/2018-10-30-google-cloud-vision/wariate.png)

下記URLのドキュメントを読んでいけばとりあえずは使えるようになるだろう。なんかGoogle Cloud Storageと統合出来るって記述があった。Storageにいれておけば勝手にラベリングしてくれるとかかな？だったらAPI叩かなくて良いので助かるがどうなんだろ。

[Cloud Vision API ドキュメント](https://cloud.google.com/vision/docs/)

「入門ガイド」の「他の特徴を検出」にWEBエンティティの取得が書いてあるのでここを参照する。

[入門ガイド](https://cloud.google.com/vision/docs/how-to)

めんどいのでcurlでやりたかったけど、例がなかったのでRubyでやってみることにする。まずは例によってgemを入れる。

```
$ gem install google-cloud-vision
```

参考までにこのgemのGitHubにURLを。

[googleapis/google-cloud-ruby](https://github.com/googleapis/google-cloud-ruby/tree/master/google-cloud-vision)

サンプルコードをちょっといじってスコアが出力されるように変更。

marin.rb

```
require "google/cloud/vision"

# WEB上においてある画像へのリンク
image_path = "http://path/to/image.jpg"
# GCPで作成したプロジェクトのID
project_id = "cloud-vision-sample-000000"

vision = Google::Cloud::Vision.new project: project_id
image  = vision.image image_path

web = image.web

web.entities.each do |entity|
  puts "#{entity.description}: #{entity.score}"
end
```

実行。

```
$ ruby main.rb
Traceback (most recent call last):
        5: from main.rb:6:in `<main>'
        4: from /Users/harada/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/google-cloud-vision-0.31.0/lib/google/cloud/vision.rb:501:in `new'
        3: from /Users/harada/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/google-cloud-vision-0.31.0/lib/google/cloud/vision.rb:566:in `default_credentials'
        2: from /Users/harada/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/googleauth-0.6.7/lib/googleauth/credentials.rb:91:in `default'
        1: from /Users/harada/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/googleauth-0.6.7/lib/googleauth/credentials.rb:133:in `from_application_default'
/Users/harada/.rbenv/versions/2.5.1/lib/ruby/gems/2.5.0/gems/googleauth-0.6.7/lib/googleauth/application_default.rb:64:in `get_application_default': Could not load the default credentials. Browse to (RuntimeError)
https://developers.google.com/accounts/docs/application-default-credentials
for more information
```

デフォルトクレデンシャルを使うらしいので設定をする必要があるらし。

コマンドでGCPをごにょごにょ出来そうなSDK？を入れて、デフォルトクレデンシャルの設定を済ませます。

```
$ brew cask install google-cloud-sdk

$ gcloud beta auth application-default login
```

再度実行すると以下のような結果が出力された。

```
久留米ラーメン清陽軒 本店: 1.0929930210113525
Ramen: 1.0777499675750732
久留米ラーメン: 0.6992999911308289
: 0.650600016117096
Pork Bones: 0.6037999987602234
Japanese Cuisine: 0.5972999930381775
Taiho: 0.5503000020980835
Noodle: 0.4499000012874603
Soup: 0.41499999165534973
Taihou Ramen KITTE Hakata: 0.37685999274253845
久留米ラーメン清陽軒 文化街店: 0.3738899827003479
: 0.33869999647140503
: 0.2946999967098236
Kurume: 0.0490993857383728
Fukuoka Prefecture: 0.00014083199494052678
```

画面から試した場合と違う結果になった理由がわからないが、ラーメンかそうでないかの判断は出来そうなのでOKとする。
