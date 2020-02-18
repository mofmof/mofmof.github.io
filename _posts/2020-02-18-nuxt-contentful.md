---
layout: blog
title: Nuxt.js + Contentful でサクッとブログのベース的なものをつくる ローカルで動作するところまで
category: blog
tags: [Vue.js, Nuxt.js, Contentful]
summary: オウンドメディアを立ち上げるにあたっていい感じの技術を選定したく、試しました。ほほうという感じ。
author: iwai
image: /images/blog/2020-02-18-nuxt-contentful/nuxt-contentful.png
---

VuePressを使ってみようかと思ったんですが、まだちょっと整備されていないっぽい？のでもう実績のあるNuxtとContentfulの組み合わせでやろうかと。ちょっと触ってみたので、そのログです。

それぞれ公式はこちら！

contentfulのnuxt integrationガイド: https://www.contentful.com/developers/docs/javascript/tutorials/integrate-contentful-with-vue-and-nuxt/

nuxt公式: https://ja.nuxtjs.org/

参考にした記事: https://qiita.com/isihigameKoudai/items/3e45ade7c438176a4cc9

## TL;DR
- プロジェクトつくります
- contentfulにコンテンツをおきます
- コンテンツを埋め込みます
- ローカルで結果を確認します

10分くらいあればできそう。結果は下記に置いてあります。

https://github.com/yubachiri/nuxt-contentful

## プロジェクト作成
環境構築はNuxtの公式を参考にしつつ頑張りましょう。

なんか聞かれたらだいたいenter。SPAモードでOKです。一点、 `dotenv` の導入だけはチェックしておいてください。

```
$ npx create-nuxt-app nuxt-contentful-sample
$ cd nuxt-contentful-sample
```

一旦動かしてみましょう。

```
$ cd nuxt-contentful-sample
$ npm run build
$ npm run start
```

http://localhost:3000で動いているはずです。動いたら次へ。

## contentfulにコンテンツ作成

https://www.contentful.com/

- アカウント作成
- 適当にspace作成
- headerの"Content model"より、Add content typeする
    - Name: sample_post
- Add fieldする
    - Textを選択
    - nameと入力
    - createをクリック
    - saveをクリック
- headerの"Content"より、コンテンツを作成
    - Add sample_postをクリック
    - nameを適当に入力
    - Publishをクリック

とても雑ですがこれでコンテンツ作成までOKです。

## Nuxtからコンテンツの取得

Contentfulのcliを入れます。

```
$ npm install --save contentful
```

ちょっとコードを触っていきます。

nuxt.config.js

```
require('dotenv').config();
const { CTF_SPACE_ID, CTF_CDA_ACCESS_TOKEN } = process.env;

export default {
...
  env: {
    CTF_SPACE_ID,
    CTF_CDA_ACCESS_TOKEN
  }
}
```

plugins/contentful.js(新規作成)

```
const contentful = require('contentful');

const config = {
  space: process.env.CTF_SPACE_ID,
  accessToken: process.env.CTF_CDA_ACCESS_TOKEN
}

const client = contentful.createClient(config);

export default client;
```

.env

```
BASE_URL=http://localhost:3000

CTF_SPACE_ID=<あとで埋めます>
CTF_CDA_ACCESS_TOKEN=<あとで埋めます>
```

pages/index.vue

```
<template>
  <div>
    <ul>
      <li v-for="post in posts">
        {%raw%}{{ post.fields.name }}{%endraw%}
      </li>
    </ul>
  </div>
</template>

<script>
  import client from '~/plugins/contentful.js'

  export default {
    asyncData() {
      return client.getEntries()
        .then(entries => {
          return {
            posts: entries.items
          }
        })
    }
  }
</script>
```

です！

.envの<あとで埋めます>の部分も埋めていきましょう。

さっき作ったcontentfulのプロジェクトのページにいき、 header > Settings > API key をクリックしてください。

Example Keyがあるはずなので、それを選択。なければAdd API keyから作成しましょう。

その中に記載されている `Space ID` と `Content Delivery API - access token` が欲しいものです。それぞれ `CTF_SPACE_ID` と `CTF_CDA_ACCESS_TOKEN` に書きましょう。

```
CTF_SPACE_ID=hogehuga123
CTF_CDA_ACCESS_TOKEN=foobar4567
```

これで万事OKなはず。

```
$ npm run build
$ npm run dev
```

すると、localhost:3000にコンテンツの `name` のリストが表示されるはずです。

以上！
