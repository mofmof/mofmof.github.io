---
layout: blog
title: モダンなheadless CMS Contentfulを試してみる
category: blog
tags: [headless CMS, contentful, CMS]
summary: オウンドメディアでも立ち上げたいなと思いまして、今だとCMS何がイケてるのかなーと調べてたらContentfulがイケてるのではと思ったので試してみます。

author: aharada
image: /images/blog/2019-05-28-contentful/post.png
---

オウンドメディアでも立ち上げたいなと思いまして、今だとCMS何がイケてるのかなーと調べてたらContentfulがイケてるのではと思ったので試してみます。

Contentfulは最近トレンドのheadless CMSというカテゴリのCMSで、コンテンツ管理画面はあるけど、コンテンツを表示する部分は含まれていなくて、APIでコンテンツデータを公開するもの。

フロントの自由度が高く、アプリにも使えるし、WEBにも使えるし、スマートにコンテンツ配信出来るわけ。逆に言うとWordpressのような従来のCMSに比べると、自分で表示部分を作り込まなければならないので大変手間がかかるのがデメリット。

Contentfulは無料プランで5,000レコードまでコンテンツを保存出来る模様。下記エントリでのざっくりとした目安では800記事くらいは書けるみたい。ありがた過ぎるやろ。

[Contentful　料金　無料枠　メモ - Qiita](https://qiita.com/belowt/items/82b9f47e5e97688edadd)

## 最初から入っている The example projectをいじってみる

サインアップするとなにやら二択でてきた。シンプルにやるには左の`I create content`の方が良さそう。とりあえず。左を選んで見る。

![二択](/images/blog/2019-05-28-contentful/first.png)

Space home 画面にチュートリアルのようなものが表示されるので、まずはView contentを押して見る。

![Space home](/images/blog/2019-05-28-contentful/space-home.png)

どうやらサンプルのスペース(プロジェクトみたいな区分け)があって、コンテンツが予め登録されている模様。

![content](/images/blog/2019-05-28-contentful/content.png)

適当にエントリを登録してみようとすると、Titleにはドイツ語を入れなければならない？みたいな状況になった。

![エントリを登録](/images/blog/2019-05-28-contentful/add-entry.png)

Localeを変更しようとすると、デフォルトの言語を日本語に変えられないみたい。どうやらスペース作成時にしか指定出来ないからっぽい。

## Postを公開する

というわけでまずはスペースを自分で登録してみる。左上のスペース名を選択するとCreate space出来るリンクがでてくる。

![Spaces](/images/blog/2019-05-28-contentful/spaces.png)

Content typeを登録せよと言われるので`Post`で登録。続いて`Post`のフィールドを定義せよと言われるので、`Title`と`Body`を登録する。

なるほど、コンテンツのフィールドはタイトルや本文以外にも自由に定義出来る。CategoryやAuthorなど、追加のフィールドも自在だ。

jsonのフィールドとかも定義出来るみたい。CMSというから文章メインかと思ったけど、いろんな形のデータを配信出来る。

![Title](/images/blog/2019-05-28-contentful/title.png)

![Body](/images/blog/2019-05-28-contentful/title.png)

次に、Settings -> Localeから、デフォルトの言語をEnglishになっているのでJPに変更する。日本語は二種類あるけど、違いが分からんので適当に選択。

Content -> Add postして適当なポストを作っておく。

![Post](/images/blog/2019-05-28-contentful/post.png)

Space homeの最後のチュートリアルで、`Fetch your content`というのが残っているので、次はAPIで取得してみる。

![Space home](/images/blog/2019-05-28-contentful/space-home2.png)


## APIから取得してみる

Rubyが得意なので`ruby`でのやり方をやってみる。

```
$ gem install contentful
```

画面から適当にAPIキー作っておく。

![APIキー](/images/blog/2019-05-28-contentful/apikey.png)

contentful.rb

```
require 'contentful'

client = Contentful::Client.new(
  space: '<PROJECT_ID>',  # This is the space ID. A space is like a project folder in Contentful terms
  access_token: '<ACCESS_TOKEN>' # This is the access token for this space. Normally you get both ID and the token in the Contentful web app
)

entries = client.entries(content_type: 'post')
puts(entries.first.body)
```

```
$ ruby contentful.rb
Postをテストしてます。
contentfulいいよcontentful
```

出来たー！

ちょっと管理画面は戸惑うところがいくつかあったけど、概念的には難しいことはないのですぐに慣れそう。
