---
layout: blog
title: "[Rails6]actioncableとvueを使って30分でチャットしてみる[Vue]"
category: blog
tags: [Rails, vue, actioncable]
summary: vueとrails6とactioncableでチャット機能を爆速で仕上げる
author: akito-n
#image: none
---


弊社もリモートワークになって早くも２ヶ月が経とうとしています。リモートワークと言ってもいいことばかりではなく、集中力のマネジメントに気を使う必要が有ります。

幸い弊社では集中力を維持するために~~スマートドラッグ~~ポモドーロテクニックが流行っており、特に弊社のつよつよエンジニアの手掛ける[g4](https://www.g-g-g-g.games/)というツールのおかげで
ゲーム感覚で集中力を維持できています。

とはいえ、いつまでもルーティーンのように同じことを繰り返していくと飽きてくるのもまた事実です。そこでポモドーロでいう１サイクル25分（＋5分）で何か１機能追加するという爆速機能追加をやっていきたいと思います。


## 言語、ライブラリのバージョン

- Ruby 2.6.3
- Rails 6.0.2.2
- vue@^2.6.11
- テンプレートはslimを使用

# 目次

- [プロジェクト立ち上げ](#anc1)
- [チャンネルを作る](#anc2)
- [トップページを作る](#anc3)
- [Vueをいれる](#anc4)
- [Vueをマウントする](#anc5)


<a id="anc1"></a>

# プロジェクト立ち上げ

今回作っていくのはRails６ + Vue + Actioncableによるチャット機能です。ただ、今回雛形でいいのがなかったのでそこから作成していきます。

```
$ rails new chat-sample --database=postgresql
```
何はともあれrails newです。databaseオプションはなんでもいいんですが使い慣れているpostgresを指定しておきます。



<a id="anc2"></a>

# セットアップ
まず、actioncableをいい感じに使うためにchannelを作成しておきます。

```
$ rails g chennel message
```
上記コマンドでmessage_channel.rbというのが作成されます。

```
class MessageChannel < ApplicationCable::Channel
  def subscribed
    # stream_from "some_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end
end

```

まずここでチャンネルの設定をしておきます。subscribeの中の

```
# stream_from "some_channel"
```
のコメントアウトを外し、message_channelとかにしておきます。
また、送信時にメッセージを受け取って返す連絡口となるメソッドとして、speakというものを用意しておきましょう。
いろいろ記載すると下記になりました。

```
# frozen_string_literal: true

class MessageChannel < ApplicationCable::Channel
  def subscribed
    stream_from 'message_channel'
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def speak(data)
    ActionCable.server.broadcast 'message_channel', message: data['message']
  end
end

```


<a id="anc3"></a>

# トップページを作る
今回はとりあえずトップページにチャットの機能を置くことにします。

```
$ rails g controller home index
```
homeコントローラーを作って、そこのindexをトップにしておきます。

routes.rbはこんな感じ

```
# frozen_string_literal: true

Rails.application.routes.draw do
  root to: 'home#index'
end

```
なお、`mount ActionCable.server => '/cable'` という一行を入れているactioncableの記事も有りましたが、試したところローカルに限ってはなくても動いたので一旦ここではつけてないです。

<a id="anc4"></a>

## Vueをいれる

ここまでで一旦railsは落ち着いたのでフロントのVueを入れていきます。

```
$ rails webpacker:install:vue
```

このコマンドでvueを入れていきましょう。

### actioncableも入れる

actuoncableも入れておきます。

```
$ npm install actioncable
```

home#indexに置くjsファイルはこんな感じにします

chat.js
```
import Vue from 'vue';
import ActionCable from 'actioncable';
import Chat from '../TheChat';

const cable = ActionCable.createConsumer('ws:localhost:3000/cable');
Vue.prototype.$cable = cable;

document.addEventListener('DOMContentLoaded', () => {
  const app = new Vue({
    el: '#chat',
    render: h => h(Chat)
  })
})

```

```
const cable = ActionCable.createConsumer('ws:localhost:3000/cable');
Vue.prototype.$cable = cable;
```

ここでactioncableをつなげています。

続いてTheChat.vueです。（名前適当につけました。）

```
<template>
  <div>
    <input v-model="msgText" placeholder="ここにメッセージ" />
    <button v-if="messageChannel" @click="speak">送信</button>
    <li v-for="(message, index) in messages" :key="index">{{ message.message || "中身ないよ" }}</li>
  </div>
</template>

<script>
export default {
  data() {
    return {
      msgText: "",
      messages: [],
      messageChannel: null
    };
  },
  created() {
    this.messageChannel = this.$cable.subscriptions.create("MessageChannel", {
      received: data => {
        console.log("dataきた");
        this.messages.push(data);
      }
    });
  },
  methods: {
    speak() {
      console.log("msg送った", this.msgBox);
      this.messageChannel.perform("speak", {
        message: this.msgText
      });
    }
  }
};
</script>

```

createdでMessageChannelを取得しています。speakした時に

```
this.messageChannel.perform("speak", {
        message: this.msgText
      });
```

で送られ、message_channel.rbの中のspeakメソッドに入ります。

もし保存とかする場合はmessage_channel.rb内のspeakメソッド内で保存するといいと思います。

ブロードキャストされたデータは

```
received: data => {
        this.messages.push(data);
      }
    });
```

ここでmessagesにpushされます。

template側では

```
<li v-for="(message, index) in messages" :key="index">{{ message.message || "中身ないよ" }}</li>
```
この部分で表示が行われます（message.messageってなんかアレですね）


<a id="anc5"></a>

## Vueをマウントする

最後に、home#indexにVueをマウントします。

home/index.html.slim
```
= javascript_pack_tag "chat.js"

#chat
```
chat.jsをpack_tagで置いてidとなっている#chatでidも書いておきます。

最後に動作確認！

![image-01](/images/blog/2020-04-24-vue-actioncable-rails/image01.gif)

あら早い！ここまで１ポモでさくっと作成することができましたね。


# まとめ
いやーポモドーロテクニックってすごいなー

## g4使いましょう！
[g4](https://www.g-g-g-g.games/)

