---
layout: blog
title: RubyをJavaScriptにしてしまうOpalを触ってみる
category: blog
tags: [Ruby on Rails, JavaScript, Opal]
summary: 何もかもRubyで書きたいあなたへ。RubyをJSにコンパイルしてくれるOpal入門します。
author: iwai
image: /images/blog/2019-04-24-opal-introduction/opal_logo.png
---

[公式](https://opalrb.com/)

## TL;DR

全部Rubyでかけるのがうま味？ Rubyに慣れ親しんだJS初心者は嬉しいかも？ なOpal入門してみます。

<br>

## Opal導入する

適当なプロジェクトを作成して、準備します。

GemfileにOpalを追加
```
gem 'opal-rails'
```

app/assets/javascripts/application.js を app/assets/javascripts/application.js**.rb**に変更して中身も変える

```
require 'opal'
require 'opal_ujs'
require 'turbolinks'
require_tree '.'
```

したら、コントローラを作ってみます。

```
$ rails g controller Sample index
```

結果
```
create  app/controllers/sample_controller.rb
  route  get 'sample/index'
invoke  erb
create    app/views/sample
create    app/views/sample/index.html.erb
invoke  test_unit
create    test/controllers/sample_controller_test.rb
invoke  helper
create    app/helpers/sample_helper.rb
invoke    test_unit
invoke  assets
invoke    opal
create      app/assets/javascripts/sample.js.rb
invoke    scss
create      app/assets/stylesheets/sample.scss
```

Opalがこんなものを作ってくれましたね。

```
create      app/assets/javascripts/sample.js.rb
```

```
# Place all the behaviors and hooks related to the matching controller here.
# All this logic will automatically be available in application.js.
# You can use Opal in this file: http://opalrb.org/
#
#
# Here's an example view class for your controller:
#
class SampleView
  # We should have <body class="controller-<%= controller_name %>"> in layouts
  def initialize(selector = 'body.controller-sample')
    @selector = selector
  end

  def setup
    on(:click, 'a', &method(:link_clicked))
  end

  def link_clicked(event)
    event.prevent
    puts "Hello! (You just clicked on a link: #{event.current_target.text})"
  end


  private

  attr_reader :selector, :element

  # Uncomment the following method to look for elements in the scope of the
  # base selector:
  #
  # def find(selector)
  #   Element.find("#{@selector} #{selector}")
  # end

  # Register events on document to save memory and be friends to Turbolinks
  def on(event, selector = nil, &block)
    Element[`document`].on(event, selector, &block)
  end
end

SampleView.new.setup
```

おお、なんとなく読めます。動作確認してみましょう。

自動生成のコードでは、尻の`setup`メソッドでaタグに`link_clicked`を呼ぶクリックイベントを定義してくれていそうなので、aタグ設置したりしてみようと思います。クラス宣言の外に下記を追加して、/sample/indexに行きましょう。

```
Document.ready? do
  Element.find('body').html = '<h1>Hello World from Opal!</h1> <a href="#">Hello</a>'
end
```

htmlがHelloみたいな雰囲気になり、ひとつリンクが設置されているはずです。リンクをクリックすると`link_clicked`が呼ばれてコンソールにputsしてくれますね。ほへー。

自動生成されたコードを参考にしつつ、さっき追記したものを消してもうちょっと触ってみる。

/app/views/sample/index.html.erb
```
<h2>sample</h2>
<input type="text" id="sample_input" data-target="#sample_text">
<input type="button" id="change_color" data-target="#sample_text" value="色変えるボタン">

<div id="sample_text"></div>
```

/app/assets/javascripts/sample.js.rb
```
略

  def setup
    on(:click, 'a', &method(:link_clicked))
    on(:input, '#sample_input', &method(:value_changed))
    on(:click, '#change_color', &method(:change_color))
  end

  def link_clicked(event)
    event.prevent
    puts "Hello! (You just clicked on a link: #{event.current_target.text})"
  end

  def value_changed(event)
    Element.find(event.target.data['target']).text = event.target.value
  end

  def change_color(event)
    Element.find(event.target.data['target']).css("color", "red")
  end

略
```

こんなんになる。

![opal_result](/images/blog/2019-04-24-opal-introduction/opal_result.png)

jQueryより見た目はスッキリするし、諸々をRubyで統一できるのはもしかしたらハッピーかもしれないですね。今回はシンプルにサクッとなのでerbだったけど他のテンプレートエンジンに乗せかえるのもそんな問題なさそうです。
