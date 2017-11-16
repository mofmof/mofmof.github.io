---
layout: blog
title: Rails + Vue.jsでチャットUIっぽい吹き出しをコンポーネントにする
category: blog
tags: [Vue, Vue.js, Rails]
summary: 吹き出しのようなチャットUIを実装し、コンポーネント化してみます。
author: aharada
image: /images/blog/2017-11-16-rails-vue-component/chat.png
---

前回は「[Rails + Vue.js + Ajaxでチャット機能っぽいサンプルを実装する](/blog/rails-vue-chat-sample.html)」というのをやってみましたが、これに加えて吹き出しのようなチャットUIを実装し、コンポーネント化してみます。

このあたりが使いこなせるようになると、いままでバラバラのファイルに書かれていたjs,html,cssを小さい単一のコンポーネントに分割して管理できるのでVue.jsのありがたみがヒシヒシと感じられるようになってきます。

最終的にこんな感じの画面になります。

![チャットUI](/images/blog/2017-11-16-rails-vue-component/chat.png)

## 実装

まずは、`views/messages/index.html.erb`の`table`タグで実装していたメッセージ部分を、`ul`タグに変更し、`li`部分を`balloon`としてコンポーネントに置き換えます。

```html
<script>
    rails = window.rails = {};
    rails.messages = <%= raw @messages.to_json %>;
</script>

<p id="notice"><%= notice %></p>

<h1>メッセージ</h1>

<ul id="message-table">
  <balloon v-for="m in messages" :username="m.username" :body="m.body"></balloon>
</ul>
<br>


<%= form_for @message, html: { id: 'message-form' } do |f| %>
  <div class="field">
    <%= f.label :username %>
    <%= @message.username %>
    <%= f.hidden_field :username, 'v-model': 'message.username' %>
  </div>
  <div class="field">
    <%= f.text_area :body, cols: 100, rows: 10, 'v-model': 'message.body' %>
  </div>
  <button type="button" v-on:click="addNewMessage">送信</button><br />
  <span>username: {{ message.username }}</span><br />
  <span>body: {{ message.body }}</span><br />
<% end %>

<%= javascript_pack_tag 'messages/index' %>
```

`app/javascript/packs/messages/index.js`にコンポーネントをインポートして、Vueのインスタンス作成時に`components`を指定します。

```js
import Vue from 'vue/dist/vue.esm.js'
import App from '../app.vue'
import axios from 'axios';
import Balloon from '../components/balloon.vue'


document.addEventListener("DOMContentLoaded", () => {
  new Vue({
    el: '#message-table',
    components: {
      Balloon,
    },
    data: {
      messages: rails.messages
    },
  })

  new Vue({
    el: '#message-form',
    data: {
      message: {
        username: document.querySelector("[v-model='message.username']").value,
        body: document.querySelector("[v-model='message.body']").value,
      },
      messages: rails.messages
    },
    methods: {
      addNewMessage: function() {
        console.log('addNewMessage')
        axios.post('/messages.json', {
          message: this.message
        })
        .then(res => {
          // console.log(res.data)
          this.messages.push({
            username: this.message.username,
            body: this.message.body
          })
        })

      },
    }
  })

})
```

`app/javascript/packs/components/balloon.vue`に実際のコンポーネントを実装をします。

```html
<template>
  <li>
    <p class="username">
      {{ username }}
    </p>
    <div class="balloon1-left">
      <p>
        {{ body }}
      </p>
    </div>
  </li>
</template>

<script>
export default {
    props: ['username', 'body']
}
</script>

<style scoped>
li {
  list-style: none;
}

.username {
  float: left;
}

.balloon1-left {
  position: relative;
  margin: 1.5em 0 1.5em 15px;
  padding: 7px 10px;
  min-width: 120px;
  max-width: 500px;
  color: #555;
  font-size: 16px;
  background: #e0edff;
  margin-left: 70px;
}

.balloon1-left:before{
  content: "";
  position: absolute;
  top: 50%;
  left: -30px;
  margin-top: -15px;
  border: 15px solid transparent;
  border-right: 15px solid #e0edff;
}

.balloon1-left p {
  margin: 0;
  padding: 0;
}
</style>
```

Railsでは単一ファイルコンポーネントをしたいときにcssだけ別ファイルで生成されてしまうらしいので、設定を追加する。

`config/webpack/environment.js`

```
const { environment } = require('@rails/webpacker')

module.exports = environment
environment.loaders.get('vue').options.extractCSS = false
```

はい、これで出来上がり！

なんと簡単。素敵ですね、単一ファイルコンポーネント。
