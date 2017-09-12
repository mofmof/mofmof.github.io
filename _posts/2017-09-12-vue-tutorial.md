---
layout: blog
title: ReactよりイケてるらしいのでVue.jsに入門してみた
category: blog
tags: [Vue.js, vue]
summary: Reactのコードをたまにレビューすることがあるのですが、いつも記述量多いなーと感じていました。そんなとき、
author: aharada
---

`React`のコードをたまにレビューすることがあるのですが、いつも記述量多いなーと感じていました。そんなとき、`Vue.js`はもっとシンプルでいいよ！という悪魔の囁きを聞いたので`React`ではなく`Vue.js`に入門してみることにしました。

## インストール

基本的に公式ページが最新なので、公式ページを参考にします。日本語化されている。先人の努力の結晶や。ありがとう。

[https://jp.vuejs.org/v2/guide/installation.html](https://jp.vuejs.org/v2/guide/installation.html)

devtoolを入れろと書いてあるので入れた。使い方はまだ知らん

[https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)

`npm`と使ってインストールする。ぼくの環境では昔にインストールしたやつが入っているのか、特に`npm`自体のセットアップは必要なかった。

```
$ npm install vue
```

cliツール入れろと書いてあるので言われるがままやってみる。

```
$ npm install --global vue-cli
# "webpack" ボイラープレートを使用した新しいプロジェクトを作成する
$ vue init webpack vue-tutorial
# 依存関係をインストールしてgo!
$ cd my-project
$ npm install
$ npm run dev
```

webpackとはなんぞや状態。

全部ちょいちょいなんか聞かれますが、面倒なのでそのままエンターキー連打でやり過ごす作戦。

```
? Project name vue-tutorial
? Project description A Vue.js project
? Author a.harada <redhornet96@gmail.com>
? Vue build standalone
? Install vue-router? Yes
? Use ESLint to lint your code? Yes
? Pick an ESLint preset Standard
? Setup unit tests with Karma + Mocha? Yes
? Setup e2e tests with Nightwatch? Yes
```

プロジェクトのテンプレートが出来上がったみたい。だがしかし動かし方が良くわからなかったので、そっと削除した。

### チュートリアル

気を取り直して、次のチュートリアル的なの進めます。

[https://jp.vuejs.org/v2/guide/index.html](https://jp.vuejs.org/v2/guide/index.html)

ここを見ると初心者は`vue-cli`を使ってくれるなと書いてある。インストール手順の方でインストールしろって書いてあったから入れたのに！くっ

`index.html`ファイルだけで完結して`Vue.js`のお試しが出来るみたい。それぞれのやってみたコードを貼っていく。

まず一つ目、Hello Vue!って表示された。divタグの中身のmessageがjs側でバインディングされた模様。

```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app">
      {{ message }}
    </div>
    <script type="text/javascript">
      var app = new Vue({
        el: '#app',
        data: {
          message: 'Hello Vue!'
        }
      })
    </script>
  </body>
</html>
```

続いてこれ。`Hover your mouse over me for a few seconds to see my dynamically bound title!`というテキストにマウスカーソルをあわせてしばらく待つと`You loaded〜`と表示される。

```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app-2">
      <span v-bind:title="message">
        Hover your mouse over me for a few seconds
        to see my dynamically bound title!
      </span>
    </div>
    <script type="text/javascript">
      var app2 = new Vue({
        el: '#app-2',
        data: {
          message: 'You loaded this page on ' + new Date().toLocaleString()
        }
      })
    </script>
  </body>
</html>
```

次は表示/非表示切り替えかな。seenをfalseにしたら非表示になった。

```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app-3">
      <p v-if="seen">Now you see me</p>
    </div>
    <script type="text/javascript">
      var app3 = new Vue({
        el: '#app-3',
        data: {
          seen: true
        }
      })
    </script>
  </body>
</html>

```

ループは`v-for`だけで表現出来るみたい。これはイケてる感やばい。

`app4.todos.push({text: 'hogehoge'})`と入力してみると新しく画面にも追加された。わーイケてる。


```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app-4">
      <ol>
        <li v-for="todo in todos">
          {{ todo.text }}
        </li>
      </ol>
    </div>
    <script type="text/javascript">
      var app4 = new Vue({
        el: '#app-4',
        data: {
          todos: [
            { text: 'Learn JavaScript' },
            { text: 'Learn Vue' },
            { text: 'Build something awesome' }
          ]
        }
      })
    </script>
  </body>
</html>
```

こんどはボタンを押すとテキストがリバースされる。イベントリスナーとか気にしないで実装出来るの楽だー。

```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app-5">
      <p>{{ message }}</p>
      <button v-on:click="reverseMessage">Reverse Message</button>
    </div>
    <script type="text/javascript">
      var app5 = new Vue({
        el: '#app-5',
        data: {
          message: 'Hello Vue.js!'
        },
        methods: {
          reverseMessage: function () {
            this.message = this.message.split('').reverse().join('')
          }
        }
      })
    </script>
  </body>
</html>
```

inputに入力したテキストがそのままpタグ内に表示される。こちらもイベントリスナーなしで実装できる。


```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app-6">
      <p>{{ message }}</p>
      <input v-model="message">
    </div>
    <script type="text/javascript">
      var app6 = new Vue({
        el: '#app-6',
        data: {
          message: 'Hello Vue!'
        }
      })
    </script>
  </body>
</html>
```

続いてコンポーネントについて。上の例ではliタグで直接TODOリストを表現していたのですが、liタグをtodo-itemというコンポーネントでラップする。コンポーネントに切り出すことで、一部品のなかでロジックを完結させられるようになる。

最後なのでこれだけ画像アップしとくわ。

```
<html>
  <head>
    <script src="https://unpkg.com/vue"></script>
  </head>
  <body>
    <div id="app-7">
      <ol>
        <!--
          todo オブジェクトによって各 todo-item を提供します。
          それは、内容を動的にできるように表します。
          また後述する "key" で各コンポーネントに提供する必要があります。
        -->
        <todo-item v-for="item in groceryList" v-bind:todo="item"></todo-item>
      </ol>
    </div>
    <script type="text/javascript">
      Vue.component('todo-item', {
        props: ['todo'],
        template: '<li>{{ todo.text }}</li>'
      })
      var app7 = new Vue({
        el: '#app-7',
        data: {
          groceryList: [
            { id: 0, text: 'Vegetables' },
            { id: 1, text: 'Cheese' },
            { id: 2, text: 'Whatever else humans are supposed to eat' }
          ]
        }
      })
    </script>
  </body>
</html>
```

![コンポーネント](/images/blog/2017-09-12-vue-tutorial/component.png)

ざっくりどういうことが出来るかわかった。次はRails上で使う方法を試してみたい。
