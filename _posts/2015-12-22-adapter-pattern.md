---
layout: blog
title: Gofのデザインパターン、Adapterパターンの使いどころ
category: blog
tags: [デザインパターン,Ruby,Adapterパターン]
summary: 今「増補改訂版Java言語で学ぶデザインパターン入門」というGoFのデザインパターンについて説明している入門書を読んでいます。
author: aharada
---

今「増補改訂版Java言語で学ぶデザインパターン入門」というGoFのデザインパターンについて説明している入門書を読んでいます。

<iframe src="https://rcm-fe.amazon-adsystem.com/e/cm?lt1=_blank&bc1=FFFFFF&IS2=1&bg1=FFFFFF&fc1=000000&lc1=0000FF&t=redhornet09-22&o=9&p=8&l=as1&m=amazon&f=ifr&ref=qf_sp_asin_til&asins=4797327030" style="width:120px;height:240px;" scrolling="no" marginwidth="0" marginheight="0" frameborder="0"></iframe>

昔ぼくがまだ若手の時に、Javaをメインにやっていて、そのときGoFのデザインパターンの勉強をしたことがあるのですが、当時はパッパラパーでした。

どうやら久々にまたちゃんとデザインパターンを勉強してみると、以前よりだいぶ理解が進むようになったようです。今回はAdapterパターンについて読んだのでちょっと書き残しておきます。


## 具体例

例えば、こんな感じに本文と宛先と送信元を引数に渡して、メッセージを送信するMessengerクラスのdeliverメソッドがあったとします。この時点はまあ問題なく動いているわけです。

```
class Messenger
  def deliver(body, to, from)
    puts "deliver! body: #{body}, to: #{to}, from: #{from}"
  end
end

Messenger.new.deliver('Hello!', 'bob', 'alice')
# => deliver! body: Hello!, to: bob, from: alice
```

次に、開発している中でdeliverした本文をMessengerのフィールドに持たせて後からでも参照出来るようにしたくなったとします。

この例で言えば、Messengerクラスにフィールドを持たせて、deliverメソッドの引数を変更すれば対応することも出来ます。ところがこのMessengerクラスのdeliverメソッドがアプリケーション全体で何箇所も使用されていたらどうでしょうか？

直すことは出来ても、あちこちを修正する必要があるので修正範囲が大きく、バグを埋め込むリスクが高くなってしまいます。そんなときに効くのがGoFのデザインパターンの1つ、Adapterパターンです。

下のようにMessengerクラスを継承したMessageSenderクラスを定義して、deliverメソッドの引数の差分を吸収することで、既存のMessengerクラスに一切手を加えずに、新しいインターフェースにadapt(適合させる)ことが出来ます。

```
class MessageSender < Messenger
  attr_accessor :body, :to, :from

  def initialize(body, to, from)
    @body = body
    @to = to
    @from = from
  end

  def deliver
    super(body, to, from)
  end
end

sender = MessageSender.new('Hello!', 'bob', 'alice')
sender.deliver
# => deliver! body: Hello!, to: bob, from: alice

puts sender.body
# => Hello!
```

## 使えそうなシーン

あるモジュールについて、最初にざっくりと作ってしまったため、後からメソッドの引数の設計があんまりイケてなかったことに気が付いた。だけど、既にそのモジュールはあちこちで使われてしまっていて、メソッドの引数を全ての使用箇所で修正するのは非常にリスキー。修正箇所を限定的にして、リスク少なく修正したいっていうケースに使えそうですね。

あとは、クラス名とかメソッド名とかをtypoしていたことに相当後になってから気づいたとか、そういうケースでもtypoを修正したクラスを別途作って、旧クラスをadaptして共存させるっていうやり方をすれば安全に1つずつ修正が出来るようになりそう。
