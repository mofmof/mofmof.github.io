---
layout: blog
title: Vue.js + Firestoreでデータを取得してラーメン一覧を表示する
category: blog
tags: [vue.js, firestore, firebase, vue-router]
summary: まずはVue.jsのお作法が全く分からないので、GitHubでVue.jsのプロジェクトを探してみる。
author: aharada
image: /images/blog/2018-11-13-vue-firestore-ramens/screenshot.png
---

まずはVue.jsのお作法が全く分からないので、GitHubでVue.jsのプロジェクトを探してみる。スターの多い順で見ればより良いリポジトリが見つけられるという仮説で検索。

[Search · App.vue · GitHub](https://github.com/search?o=desc&p=2&q=App.vue&s=stars&type=Repositories)

[https://github.com/prograhammer/vue-pizza/](https://github.com/prograhammer/vue-pizza/)

このリポジトリでは`src/features`配下に画面ごとのディレクトリをきって、`main.vue`で画面自体の実装をしていることが読み取れる。

[GitHub - gothinkster/vue-realworld-example-app: An exemplary real-world application built with Vue.js, Vuex, axios and different other technologies. This is a good example to discover Vue for beginners.](https://github.com/gothinkster/vue-realworld-example-app)

こっちのリポジトリでは画面を`src/views`配下の`*.vue`ファイルで実装している。こっちの方が直感的だな。こちらの構成を踏襲するが、日本語の情報の多くは`src/views`よりも`src/pages`で作っている例が多かった。

[https://public-constructor.com/how-to-create-spa-with-vue/#toc5](https://public-constructor.com/how-to-create-spa-with-vue/#toc5)

このソースコードを読む限り`vue-router`が必要っぽい。確かコマンドで導入できたはずなのでやってみる。

```
$ yarn add vue-router
```

基本的に上記のURLのコードをコピペして進めていたのだけど、途中ハマった。

原因は`main.js`の`new Vue()`をしている箇所。ここを以下のように実装したら動くようになった。

main.js

```
import Vue from 'vue'
import App from './App.vue'
import router from './router.js'

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

## Firestoreの構成
趣味でいじってたFirebaseプロジェクトがあるので、それを転用して実装する。Firestoreはこんな感じのデータになっている。

```
- ramens
  - <unique_key>
    - body: "高田馬場のさんぽいちっていうラーメン"
    - imageUrl: "https://image/to/url.jpg"
  - <unique_key>
    - body: "渋谷の吉虎おいしいよ吉虎"
    - imageUrl: "https://image/to/url.jpg"
```


## その他のソースコード

全てのコードを載せるのも冗長なので、主にいじったソースコードのみ載せる。他はGitHubを参照してくだしあ。

[https://github.com/harada4atsushi/todo-vue-cli3-](https://github.com/harada4atsushi/todo-vue-cli3-)

router.js

```
import Vue from 'vue';
import VueRouter from 'vue-router';

import HelloWorld from '@/pages/HelloWorld.vue';
import Home from '@/pages/Home.vue';
import SearchIp from '@/pages/SearchIp.vue';
import Ramens from '@/pages/Ramens.vue';

Vue.use(VueRouter);

var router = new VueRouter({
  routes: [
    {
      path: '/',
      component: HelloWorld
    },
    {
      path: '/search_ip',
      component: SearchIp
    },
    {
      path: '/ramens',
      component: Ramens
    }
  ]
});

export default router;
```

前回は`Ramens.vue`にfirebaseの初期化処理を実装してましたが、それだと他のコンポーネントから呼び出せないので`firestore.js`に切り出した。

firestore.js

```
import firebase from 'firebase'
import 'firebase/firestore'

const firebaseApp = firebase.initializeApp({
  apiKey: process.env.VUE_APP_FIREBASE_API_KEY,
  authDomain: process.env.VUE_APP_FIREBASE_AUTH_DOMAIN,
  databaseURL: process.env.VUE_APP_FIREBASE_DATABASE_URL,
  projectId: process.env.VUE_APP_FIREBASE_PROJECT_ID,
  storageBucket: process.env.VUE_APP_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.VUE_APP_FIREBASE_MESSAGING_SENDER_ID
});

const firestore = firebaseApp.firestore()
firestore.settings({ timestampsInSnapshots: true })

export default firestore
```

App.vue

```
<template>
  <div id="app">
    <router-link to="/">HelloWorld</router-link>&nbsp;
    <router-link to="/search_ip">Search IP</router-link>&nbsp;
    <router-link to="/ramens">Ramens</router-link>&nbsp;
    <router-view />
  </div>
</template>

<script>
export default {
  name: 'App',
  components: {
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

`HelloWorld.vue`は前回そのままだから省略。`SearchIp.vue`もどこかのコピペで今回はあまり意味はないから省略。

Ramens.vue

```
<template>
  <div id="ramens">
    <div v-for="ramen in ramens">
      <img :src="ramen.imageUrl" />
      <p>{{ramen.body}}</p>
    </div>
  </div>
</template>

<script>
import db from '../firestore.js'

export default {
  name: 'ramens',
  data() {
    return {
      ramens: []
    }
  },
  mounted() {
    db.collection('ramens').get().then(snap => {
      const array = [];
      snap.forEach(doc => {
        array.push(doc.data());
      });
      this.ramens = array
    });
  }
}

</script>

<style>
img {
  width: 200px;
}
</style>
```

これでこんな感じの画面が出来上がる。

![ラーメン一覧](/images/blog/2018-11-13-vue-firestore-ramens/screenshot.png)
