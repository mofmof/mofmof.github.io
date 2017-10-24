---
layout: blog
title: Rails + Vue.js + Ajaxでチャット機能っぽいサンプルを実装する
category: blog
tags: [Vue.js, vue, Rails, Ruby on Rails, Ajax]
summary: 前回はRails + Vue.jsで動的に画面の値を取得するサンプルを組んだような覚えがあるので、
author: aharada
---

前回はRails + Vue.jsで動的に画面の値を取得するサンプルを組んだような覚えがあるので、今回はVue.jsを経由してデータを登録するサンプルを実装したい。

## とりあえず普通にRails単体で動くサンプルを実装

messagesをscaffold

```
$ r g scaffold messages username:string body:string
```

```
class CreateMessages < ActiveRecord::Migration[5.1]
  def change
    create_table :messages do |t|
      t.string :username
      t.string :body, limit: 1000

      t.timestamps
    end
  end
end
```

テーブルを作成

```
$ rake db:migrate
```

seedで適当なサンプルデータを作っておく

db/seeds.rb

```
Message.create(username: 'taro', body: 'こんにちは！太郎です、よろしくね。')
Message.create(username: 'hanako', body: 'はじめまして、私は花子と申します。太郎さんは彼女とかいるんですか？')
Message.create(username: 'taro', body: 'そうですね、特定の人はいませんが、花子が本命ですよ')
Message.create(username: 'hanako', body: '面白いやつだな、気に入った。殺すのは最後にしてやる')
```

```
$ rake db:seed
```

## 普通にRailsでindexを表示する

新規登録出来るフォームを追加し、単にRails一覧表示する画面。

messages/index.html.erb

```
<p id="notice"><%= notice %></p>

<h1>Messages</h1>

<table>
  <thead>
    <tr>
      <th>Username</th>
      <th>Body</th>
      <th colspan="3"></th>
    </tr>
  </thead>

  <tbody>
    <% @messages.each do |message| %>
      <tr>
        <td><%= message.username %></td>
        <td><%= message.body %></td>
        <td><%= link_to 'Show', message %></td>
        <td><%= link_to 'Edit', edit_message_path(message) %></td>
        <td><%= link_to 'Destroy', message, method: :delete, data: { confirm: 'Are you sure?' } %></td>
      </tr>
    <% end %>
  </tbody>
</table>

<br>

<%= form_for @message do |f| %>
  <div class="field">
    <%= f.label :username %>
    <%= @message.username %>
    <%= f.hidden_field :username %>
  </div>
  <div class="field">
    <%= f.text_area :body, cols: 100, rows: 10 %>
  </div>
  <%= f.submit '送信' %>
<% end %>
```

messages_controller.rb

```
class MessagesController < ApplicationController
  before_action :set_message, only: [:show, :edit, :update, :destroy]

  def index
    @messages = Message.all
    @message = Message.new(username: 'taro')
  end

  def create
    @message = Message.new(message_params)

    respond_to do |format|
      if @message.save
        format.html { redirect_to messages_url, notice: 'Message was successfully created.' }
        format.json { render :show, status: :created, location: @message }
      else
        format.html { render :new }
        format.json { render json: @message.errors, status: :unprocessable_entity }
      end
    end
  end

  def destroy
    @message.destroy
    respond_to do |format|
      format.html { redirect_to messages_url, notice: 'Message was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_message
      @message = Message.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def message_params
      params.require(:message).permit(:username, :body)
    end
end
```

http://localhost:3000/messages

とりあえず普通に動いた。

![スクリーンショット](/images/blog/2017-10-24-rails-vue-chat-sample/new-message.png)


## Vue.js経由でmessageを追加出来るようにする

テーブルを表示するために、Vue側にサーバサイドから取得したデータを渡す必要があった。こちらを参考にしてやってみたけど、こうじゃない感ががが。

[https://qiita.com/hkusu/items/64cfd8bf4b4c8391e454](https://qiita.com/hkusu/items/64cfd8bf4b4c8391e454)

RailsからJS側に変数の値を渡すためのgemで[gon](https://github.com/gazay/gon)というのがあるみたいだけど時間が足りなかったので触らなかった。

兵十「ゴン、お前だったのか…」

messages/index.html.erb

```
<script>
    // この辺こうじゃない感。お行儀が悪い印象。
    rails = window.rails = {};
    rails.messages = <%= raw @messages.to_json %>;
</script>

<p id="notice"><%= notice %></p>

<h1>Messages</h1>

<table id="message-table">
  <tr>
    <th>Username</th>
    <th>Body</th>
    <th colspan="3"></th>
  </tr>
  <tr v-for="m in messages">
    <td>{{ m.username }}</td>
    <td>{{ m.body }}</td>
  </tr>
</table>
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

app/javascript/packs/messages/index.js

```
import Vue from 'vue/dist/vue.esm.js'
import App from '../app.vue'
import axios from 'axios';

document.addEventListener("DOMContentLoaded", () => {
  new Vue({
    el: '#message-table',
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

なんとなくVueのインスタンスを2つ作っちゃったんだけどいいのかな。あんまりサンプルでは見たことない。

messages_controller.rb

```
class MessagesController < ApplicationController
  before_action :set_message, only: [:show, :edit, :update, :destroy]

  def index
    @messages = Message.all
    @message = Message.new(username: 'taro')
  end

  def create
    @message = Message.new(message_params)

    respond_to do |format|
      if @message.save
        format.html { redirect_to messages_url, notice: 'Message was successfully created.' }
        format.json { render :show, status: :created, location: @message }
      else
        format.html { render :new }
        format.json { render json: @message.errors, status: :unprocessable_entity }
      end
    end
  end

  def destroy
    @message.destroy
    respond_to do |format|
      format.html { redirect_to messages_url, notice: 'Message was successfully destroyed.' }
      format.json { head :no_content }
    end
  end

  private
    # Use callbacks to share common setup or constraints between actions.
    def set_message
      @message = Message.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def message_params
      params.require(:message).permit(:username, :body)
    end
end
```

画面から送信ボタンを押して見るとコンソールにエラーが出ている。

```
Can't verify CSRF token authenticity.Completed 422 Unprocessable Entity
```

RailsのCSRFトークンを渡していないからエラーになっている模様。時間がないのでコメントアウトしちゃおっと。プロダクションのコードでは絶対にやらないようにね。

application_controller.rb

```
# protect_from_forgery with: :exception
```

できたああああ！！

![Vue.jsで動的にメッセージを追加](/images/blog/2017-10-24-rails-vue-chat-sample/messages.gif)
