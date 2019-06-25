---
layout: blog
title: Rails 6 と Action text を使ってみる
category: blog
tags: [Rails, Action text, ブログ]
summary: Action text を試してみる。
author: 　sakashita
image: /images/blog/2019-06-24-rails6-actiontext/card-header.png
---

Rails 6 は Beta版に続き、[rc1 が リリース](https://weblog.rubyonrails.org/2019/4/24/Rails-6-0-rc1-released/)されました。Rails 6 では action mailbox, multiple databases, parallel testing... など様々な新機能が実装されていますが、今回は action text を触ってみようと思います。

[Action text](https://weblog.rubyonrails.org/2018/10/3/introducing-action-text-for-rails-6/) は、ざっくり説明すると Basecamp で使用されているリッチテキストエディタ [Trix](https://trix-editor.org/)と Active Strorage & Image processing （と text-processing）を組み合わせた [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) です。

記事を書くために Action text を触ってみた感じになっていますが、WYSIWYG を実装したいと思ったことが事の発端でした。はじめは [Floara](https://www.froala.com/wysiwyg-editor) を試していて、かなりいい感じだったんですが、有料なのでやっぱりなあと思い、せっかくなので Action text を試そうと思った次第です。

## Rails 6 インストールとセットアップ
前置きが長くなりましたがここから実装していきます。
まずはセットアップから。（```Ruby >= 2.5.0``` なので、必要に応じてインストールと変更を。）

```
$ ruby -v
ruby 2.6.3p62 (2019-04-16 revision 67580) [x86_64-darwin18]

$  rails -v
Rails 5.2.3

$ gem install rails --pre
Fetching zeitwerk-2.1.6.gem
Fetching activesupport-6.0.0.rc1.gem
...
14 gems installed

$ rails -v
Rails 6.0.0.rc1
```
（ちなみにですが、rc は Release Candidate の略だということを初めて知りました。）

続いて ```rails new``` します。オプションは必要に応じて変更してください。

```
$ rails new action_text --skip-coffee --skip-turbolinks --database=postgresql
...
✨  Done in 3.85s.
Webpacker successfully installed 🎉 🍰
```

最後の二行だけしか表示していませんが、ご存知の通り、Rails 6 では デフォルトで Webpacker がイントールされるようになっていることがわかります。
migrate して サーバーを起動して画面を確認します。

```
$ cd action_text
$ rails db:create db:migrate
Created database 'action_text_development'
Created database 'action_text_test'

$ rails s
```

![Yay! You're on Rails!](/images/blog/2019-06-24-rails6-actiontext/yay-youre-on-rails.png)

担当するプロジェクトでは ```slim``` を使うことが多いので、ここでも ```slim``` を使用してみます。

```rb:Gemfile
...
gem 'slim-rails'
```

```
$ bundle install
```

ここから action text を実装していきます。
まずは scaffold から。

```
$ rails g scaffold Blog title:string body:text
$ rails db:migrate
```

root を ```blog#index``` に変更します。

```rb:config/routes.rb
Rails.application.routes.draw do
+  root 'blogs#index'
  resources :blogs
  # For details on the DSL available within this file, see https://guides.rubyonrails.org/routing.html
end
```

## Action text の実装
ここから Action text の実装に移ります。

まずはAction text をインストールし、

```
$ rails action_text:install
$ rails db:migrate
```

Blog の body を has_rich_text して、

```rb:app/models/blog.rb
class Blog < ApplicationRecord
  has_rich_text :body
end
```

form の body を rich_text_area に変更します。

```rb:app/views/blogs/_form.html.slim
...
  .field
    = f.label :body
-    = f.text_area :body
+    = f.rich_text_area :body
...
```

実装はこれだけです。（あとで分かりますがこれだけでは無かった。）実際に少し画面を触ってみます。

![Action text -- Form](/images/blog/2019-06-24-rails6-actiontext/action-text-form.png)

Drag and drop による画像アップロードが良い感じです。何も実装していないですが、application.js で require している js と active storage がよしなにやってくれています。
アンダーラインや文字色選択など、もうちょっと機能ほしいかなという感覚もありますが、最低限のことはできるようになっています。なお、`rich_text_area` で編集された内容はHTMLで保存されます。
では、登録して詳細画面へ遷移して、内容を確認してみます。

![Action text -- Form](/images/blog/2019-06-24-rails6-actiontext/raw-html-on-show-page.png)

おっとこれはまずいですね... 登録されたHTMLコードがそのまま表示されてしまっています。```html_safe``` すれば行けるだろうと思って試したのですが、```Blog.body``` は ```ActionText::RichText``` オブジェクトなので、html_safe できません。```Blog.body.body``` で登録されたHTMLを取得することができるので html_safe による編集内容の反映が可能ですが、画像が取得できなくなってしまっています。。

ここで少し [action_text](https://github.com/rails/actiontext/blob/archive/app/models/action_text/rich_text.rb) の実装を覗いてみます。

```rb:actiontext/app/models/action_text/rich_text.rb
class ActionText::RichText < ActiveRecord::Base
  self.table_name = "action_text_rich_texts"

  serialize :body, ActionText::Content
  delegate :to_s, :nil?, to: :body

  belongs_to :record, polymorphic: true, touch: true
  has_many_attached :embeds

  before_save do
    self.embeds = body.attachments.map(&:attachable) if body.present?
  end

  def to_plain_text
    body&.to_plain_text.to_s
  end

  delegate :blank?, :empty?, :present?, to: :to_plain_text
end
```

画像は ```:embeds``` として has_many_attached されていますね。とすれば、```ActiveStorage::Blob.service.path_for(Blog.first.body.embeds.first.key)``` で画像のパスを取得することはわかりますが、やりたいことから遠ざかってしまうので別の回避策を考えます。

とりあえず "action text tutorial" で検索してトップに出てきた [Introducing Action Text for Raisl 6](https://weblog.rubyonrails.org/2018/10/3/introducing-action-text-for-rails-6/) を見てみると、 (DHHの動画)[https://www.youtube.com/watch?v=HJZ9TnKrt7Q&feature=youtu.be] を見るようにとのことだったので見てみます。

9:30 ぐらいで紹介される ```show.html.erb``` の実装は、何もひねることなく下記のようになっていました。

```rb
<%= @post.content %>
```

んー、では erb なら行けるのか。

```rb:app/views/blogs/show.html
p
  strong Body:
-  = @blog.body.body.html_safe
+  = render partial: 'blog_body', locals: { body: @blog.body }
```

```rb:app/views/blogs/_blog_body.html.erb
<%= body %>
```

![Action text show with erb](/images/blog/2019-06-24-rails6-actiontext/show-partial-erb.png)

あ、うまく行ってますね。画像は表示されてないですが...。やはり slim が未対応だっただけのようです。[確認](https://rubygems.org/gems/slim-rails/versions/3.2.0)したら slimは2018年10月が最終更新になっていたので、正式リリースまでには対応されることを期待し、今回は rich_text の箇所だけ erb で行きます。

画像が表示されない理由はログに出ていたので、指示通り ```image_processing``` を追加して ```bundle install``` します。

```rb:Gemfile
...
gem 'slim-rails'
gem 'image_processing', '~> 1.2'
```

サーバーを再起動し、show ページを更新してみると、

![Show page sucess](/images/blog/2019-06-24-rails6-actiontext/show-page-success.png)

無事、LGTMのトムが表示されました。

## 所感
slim と image_processing の未インストールを除けば、かなり容易に実装できました。スタイル当てればそれなりの形にはなりそうです。個人的には markdown で問題ないのですが、設定・実装不要で画像アップロードまでやってくれるのは良いなと思いました。
（今回はローカルでの動作確認までを対象とするので、s3の設定等については ActiveStorage 関連の記事を参照ください。）

次回は action mailbox を触ってみようと思います。
