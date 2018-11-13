---
layout: blog
title: Vue.jsでFirebaseのFirestoreにアクセスしてデータを取得する
category: blog
tags: [vue.js, firestore, firebase]
summary: 前回はvue-cli3で環境変数をどう扱うか試したので、続きをやります。
author: aharada
image: /images/blog/2018-11-13-vue-firestore/console.png
---

前回はvue-cli3で環境変数をどう扱うか試したので、続きをやります。

[vue-cli3系を使ってvue.jsアプリケーションで環境変数を扱う](/blog/vue-cli3-env.html)

## 今回参考にしたページ
[Vue.js + FirebaseでTodoアプリを作る - Qiita](https://qiita.com/magaya0403/items/e292cd250184ea3fe7b0)
[Vue.js + Firebaseでポートフォリオを作ろう！ - Qiita](https://qiita.com/sosumii/items/f097bf8b2209b9d75d1a)
[firebase  -  npm](https://www.npmjs.com/package/firebase)

## とりあえずFirestoreにアクセス出来るところまで

思い出す意味で、まずは画面を表示させてみる。`localhost:8080`で画面が表示されることを確認。

```
$ yarn serve
```

`firebase`のパッケージが必要なので`yarn add`します。

```
$ yarn add firebase
```

main.js

```
import Vue from 'vue'
import App from './App.vue'
import firebase from 'firebase'
import 'firebase/firestore'

require('firebase/database');

Vue.config.productionTip = false

var app = firebase.initializeApp({
  apiKey: process.env.VUE_APP_FIREBASE_API_KEY,
  authDomain: process.env.VUE_APP_FIREBASE_AUTH_DOMAIN,
  databaseURL: process.env.VUE_APP_FIREBASE_DATABASE_URL,
  projectId: process.env.VUE_APP_FIREBASE_PROJECT_ID,
  storageBucket: process.env.VUE_APP_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.VUE_APP_FIREBASE_MESSAGING_SENDER_ID
});

const firestore = app.firestore()
firestore.settings({ timestampsInSnapshots: true })

export default firestore

var db = firebase.firestore();
db.collection("ramens").get().then((querySnapshot) => {
    querySnapshot.forEach((doc) => {
        console.log(`${doc.id} => ${doc.data()}`);
    });
});

new Vue({
  render: h => h(App)
}).$mount('#app')
```

.env.local

```
VUE_APP_FIREBASE_API_KEY=xxxxxxxxxxx
VUE_APP_FIREBASE_AUTH_DOMAIN=xxxxxxxxxxx
VUE_APP_FIREBASE_DATABASE_URL=xxxxxxxxxxx
VUE_APP_FIREBASE_PROJECT_ID=xxxxxxxxxxx
VUE_APP_FIREBASE_STORAGE_BUCKET=xxxxxxxxxxx
VUE_APP_FIREBASE_MESSAGING_SENDER_ID=xxxxxxxxxxx
```

.env.localに入力する情報は`firebase`のコンソールから取得することが出来る。

![環境変数](/images/blog/2018-11-13-vue-firestore/env.png)

とりあえずfirestoreからデータを取得してコンソールに表示することは出来た。

![コンソール](/images/blog/2018-11-13-vue-firestore/console.png)

## コンポーネントにしてみる

全部`main.js`に書くと当然つらいので、適当にコンポーネントにしてみる

main.js

```
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  render: h => h(App)
}).$mount('#app')
```

components/Ramens.vue

```
<template>
  <div id="ramens">
    ramenramenramen
  </div>
</template>

<script>
import firebase from 'firebase'
import 'firebase/firestore'

require('firebase/database');

export default {
  name: 'ramens',
    components: {}
}

var app = firebase.initializeApp({
  apiKey: process.env.VUE_APP_FIREBASE_API_KEY,
  authDomain: process.env.VUE_APP_FIREBASE_AUTH_DOMAIN,
  databaseURL: process.env.VUE_APP_FIREBASE_DATABASE_URL,
  projectId: process.env.VUE_APP_FIREBASE_PROJECT_ID,
  storageBucket: process.env.VUE_APP_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.VUE_APP_FIREBASE_MESSAGING_SENDER_ID
});

const firestore = app.firestore()
firestore.settings({ timestampsInSnapshots: true })

// export default firestore

var db = firebase.firestore();
db.collection("ramens").get().then((querySnapshot) => {
    querySnapshot.forEach((doc) => {
        console.log(`${doc.id} => ${doc.data()}`);
    });
});

</script>

<style>
</style>
```

App.vue

```
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <Ramens />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'
import Ramens from './components/Ramens.vue'

export default {
  name: 'app',
  components: {
    HelloWorld,
    Ramens
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

同様にconsoleに出力された。OK。画面最下部にもコンポーネントにした分が表示されている。

![コンポーネント](/images/blog/2018-11-13-vue-firestore/component.png)

次は取得したデータを画面に表示させてみたい。
