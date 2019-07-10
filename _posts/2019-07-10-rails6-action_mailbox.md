---
layout: blog
title:  Action Mailbox
category: blog
tags: [Rails, Action Mailbox, ブログ]
summary: Action Mailbox の基本的な実装方法など
author: sakashita
image: /images/blog/2019-07-10-action-mailbox/card-header.png
---

[前回の Action text](/blog/rails6-actiontext.html) に続き、今回は Action Mailbox を使ってなにか試してみようと思います。

[Action Mailbox](https://weblog.rubyonrails.org/2018/12/13/introducing-action-mailbox-for-rails-6/) も Action text 同様、 Basecamp から移植した機能のようです。

かなりざっくり内容をまとめると、Action mailer では手間がかかったメールの受信を容易に実装できるようにし、
更にメールの受信をトリガーとして Controller のような処理ができるようにする。
ということだそうなのですが、どういうことなのかよくわからなかったのでユースケースを調べてみました。

- Discussion などのページに、メールでコメントを残す
  - 記事でこの内容を書いていきます
- 問い合わせフォームから送られたメールを自動返信する
  - 例えば在庫状況に関する問い合わせメールを、商品番号などから DB に問い合わせて自動返信する、など

ユースケース自体は理解できるのですが、そもそもメールを返信する機会自体が少なくなってきているし、
開発にしても今はメールにURLを添付してブラウザやアプリ上から操作や確認させるのが一般的だと思うので、
やっぱりイマイチメリットが分からない、というのが正直な感想です。

Rails経験、メール機能に関する開発経験が少ないためイメージが沸かないのかもしれませんが、
Rails 側で受信したメールを保存できるのは使い道がありそうな気がします。

ですがとりあえず試してみようということで、ここから実装していきます。
今回のやること・やらないことは以下です。

### やること
- メールの返信で Rails アプリ上にコメントを追加していく機能を実装
- 以下のモデルを作成
  - User (Email, Name)
  - Discussion (Title, Content)
  - Comment (Body)
- Comment は User, Discussion に参照を持ち、誰がどの議論へコメントしたかがわかるようにしたい    

### やらないこと
- Mailgun, SendGrid などとのつなぎ込み（別で機会があれば記事に書きます）
- Test

ちょっとこの段階では何ができるのかイメージしづらい感じになってしまいすみません。
実装が進むにつれて、やりたいことが見えてくる感じかと思います。

# Rails 6 インストールとセットアップ
こちらの内容は、[前回の記事](/blog/rails6-actiontext.html) とほとんど同じですので、詳しい説明はそちらを参照ください。

コマンドのみ記載していきます。

```shell
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

$ rails new action_mailbox --skip-coffee --skip-turbolinks --database=postgresql
...
✨  Done in 3.85s.
Webpacker successfully installed 🎉 🍰

$ cd action_mailbox
$ rails db:create db:migrate
Created database 'action_mailbox_development'
Created database 'action_mailbox_test
```

今回は ```erb``` のまま実装してきます。

### 各モデルの作成
###  User
Action Mailbox 以外の内容が多くならないようにするため、devise は使わず簡単にします。

```shell
$ rails g scaffold User email name
```

### Discussion
コメントの対象となるディスカッション（お題）を設定するモデル。title と content をもたせます。

```shell
$ rails g scaffold Discussion title:string content:text
```

### Comment
メールの返信によって追加される、ディスカッションへのコメント。ディスカッションとユーザに属するよう参照を設定します。

コメントはディスカッションの show に表示させるだけになるので、model だけの作成となります。

```shell
$ rails g model Comment body:string discussion:references user:references
```

作成したモデルをDBに反映します。

```shell
$ rails db:migrate
```

### リレーション追加
Discussion モデルに以下の記述を追加します。

```ruby
class Discussion < ApplicationRecord
  has_many :comments
end
```

以上で下準備完了です！

# Action Mailbox の実装
## 1. インストール
ここから Action Mailbox の実装に移ります。

まずはAction Mailbox をインストールします。

```shell
$ rails action_mailbox:install
rails db:migrateCopying application_mailbox.rb to app/mailboxes
      create  app/mailboxes/application_mailbox.rb
Copied migration 20190707075445_create_active_storage_tables.active_storage.rb from active_storage
Copied migration 20190707075446_create_action_mailbox_tables.action_mailbox.rb from action_mailbox

$ rails db:migrate
```

Action text 同様、action_mailbox コマンドが追加されています。

色々と Schema に追加されたようなので、```schema.rb``` を確認してみます。

```ruby
ActiveRecord::Schema.define(version: 2019_07_07_075446) do

  # These are extensions that must be enabled in order to support this database
  enable_extension "plpgsql"

  create_table "action_mailbox_inbound_emails", force: :cascade do |t|
    t.integer "status", default: 0, null: false
    t.string "message_id", null: false
    t.string "message_checksum", null: false
    t.datetime "created_at", precision: 6, null: false
    t.datetime "updated_at", precision: 6, null: false
    t.index ["message_id", "message_checksum"], name: "index_action_mailbox_inbound_emails_uniqueness", unique: true
  end

  create_table "active_storage_attachments", force: :cascade do |t|
    t.string "name", null: false
    t.string "record_type", null: false
    t.bigint "record_id", null: false
    t.bigint "blob_id", null: false
    t.datetime "created_at", null: false
    t.index ["blob_id"], name: "index_active_storage_attachments_on_blob_id"
    t.index ["record_type", "record_id", "name", "blob_id"], name: "index_active_storage_attachments_uniqueness", unique: true
  end

  create_table "active_storage_blobs", force: :cascade do |t|
    t.string "key", null: false
    t.string "filename", null: false
    t.string "content_type"
    t.text "metadata"
    t.bigint "byte_size", null: false
    t.string "checksum", null: false
    t.datetime "created_at", null: false
    t.index ["key"], name: "index_active_storage_blobs_on_key", unique: true
  end
...
```

この Rails アプリケーション宛に送信されたメールは、action_mailbox_inbound_emails レコードに記録されます。

メール本体は ActiveStorage によって S3 などに保持することができます。
Active Storage が同時に自動で追加されたのはこのためのようです。

しかし、メール本体は設定された日数(デフォルトで30日)が経過すると、自動的に削除(Incinerate = 焼却)されます。
メール自体は基本的に保持され続けるべきではなく、適切なドメインモデルで保持されるべきだとのことです。

ちなみにメールの保持日数は ```config.action_mailbox.incinerate_after``` で設定可能なようです。

Schema 以外に追加されたのは ```application_mailbox.rb``` ですね。

```ruby
class ApplicationMailbox < ActionMailbox::Base
  # routing /something/i => :somewhere
end
```

これは ```controller``` のようなもので、メールアドレスに応じて
どの Mailbox（後述します）に処理を任せるのか、をここに定義します。
コメントアウトにあるように、基本的には正規表現で書いていきます。

今回は、````replay-[discussion_id]@example.com```` をアドレスにし、
Mailbox 名は CommentReplies として、このアドレスに来た内容を
```discussion_id``` に応じたディスカッションページへのコメントとして追加することにしたいです。

上記のアドレスからメールが来たら CommentReplies で上述の処理を実行したいので、
```application_mailbox.rb``` には、下記のように定義しておきます。

```ruby
class ApplicationMailbox < ActionMailbox::Base
  routing /reply-(.+)@example.com/i => :comment_replies
end
```

## 2. Mailbox の作成と振り分け
次に、CommentReplies Mailbox を作成します。

```shell
$ rails g mailbox CommentReplies
      create  app/mailboxes/comment_replies_mailbox.rb
      invoke  test_unit
      create    test/mailboxes/comment_replies_mailbox_test.rb
```

```ruby
class CommentRepliesMailbox < ApplicationMailbox
  def process
  end
end
```
```ApplicationMailbox``` の ```routing``` にマッチしたメールアドレスへのメールを受信したら、上記の ```process``` メソッドが呼び出されます。

このメソッド内で、送信 User と、対象の Discussion を取得し、メールの body をコメントとして追加する処理を実装していきます。

送信元メールアドレスから User を取得できなかった(= 未登録ユーザからのメールを受信した)場合は
処理を終了させたいので、まずその実装を書いていきます。


```ruby
class CommentRepliesMailbox < ApplicationMailbox
  def process
    return unless user.present?
  end

  def user
    @user ||= User.find_by(email: mail.from)
  end
end
```

ここでいきなり登場した `mail` オブジェクトは、
[こちらの gem](https://github.com/mikel/mail) でオブジェクト化した
```InboundMail```（ActionMailbox で受信したメール）です。(ActionMailbox 左記の gem をラップしている。)

Action Mailbox の実装を確認すると、[このあたり](https://github.com/rails/rails/blob/master/actionmailbox/app/models/action_mailbox/inbound_email.rb) で InboundMail をオブジェクト化しているように見えます。

このオブジェクトからは to, from などの基本的な情報に加え、cc, date なども取得可能です。
今回は触れないですが、添付ファイルや Multipart の扱いも可能となっているので、HTMLメールも比較的容易に操作できるようです。

デバッガなどで mail オブジェクトの中身を確認してみるのも良いかもしれません。

## 3. Mailbox 内で discussion への comment 追加処理を実装

次に実装したいことは Discussion の取得と、メールのBodyをコメント化することです。

```ruby
class CommentRepliesMailbox < ApplicationMailbox
  REGEX = /reply-(.+)@example.com/i

  def process
    return unless user.present?

    discusstion.comments.create(
      user: user,
      body: mail.decoded
    )
  end

  def user
    @user ||= User.find_by(email: mail.from)
  end

  def discussion
    @discussion ||= Discussion.find(discussion_id)
  end

  def discussion_id
    recepient = mail.recipients.find{ |r| REGEX.match?(r) }
    recepient[REGEX, 1]
  end
end 
```

一気にいきましたが、これで完了です。

```mail.recipients``` には、送信先メールアドレスの配列が格納されています。
(送信先が配列になるユースケースはどんなものがあるんだろう...)

とりあえず今回は ```reply-[discussion_id]@example.com``` だけが入っている想定です。
これを配列から正規表現で取得します。

取得したアドレスに対しても同じ正規表現を利用して ```discussion_id``` を取得します。
正規表現の詳しい説明は省略しますが、要は1番目にヒットする ```(.+)``` の文字列を抜き出しています。

(もしアドレスが ```reply-5@example.com``` であれば、抜き出す文字列は "5" になるので、
```discussion_id``` として "5" が返却されます。)

あとは抜き出した id から discussion を取得し、そのコメントとして mail の本文をセットしているだけです。
件名は ```mail.subject``` で取得できますが、本文は ```mail.decoded``` で取得します。
その理由はちょっと複雑なようですが、[こちら](https://github.com/mikel/mail#encodings) で説明されているため、割愛します。

正規表現を ```REGEX``` で定数化したので、```application_mailbox.rb``` 内の正規表現も置き換えておきます。

```ruby
class ApplicationMailbox < ActionMailbox::Base
-  routing /reply-(.+)@example.com/i => :comment_replies
+  routing CommentRepliesMailbox::REGEX => :comment_replies
end
```

## 4. メール送信とコメントの確認

実際に動かして様子を見てみます。

ですがその前にユーザとディスカッションを作成しておく必要があるので、root を users#new にしておきます。

```ruby
Rails.application.routes.draw do
  resources :discussions
  resources :users
  root 'users#new'
end
```

メールによって追加されたコメントをディスカッションの show ページに表示する実装を忘れていたので、
こちらも追加しておきます。

```ruby
--- app/views/discussions/show.html.erb
<p id="notice"><%= notice %></p>

<p>
  <strong>Title:</strong>
  <%= @discussion.title %>
</p>

<p>
  <strong>Content:</strong>
  <%= @discussion.content %>
</p>

+ <% if @discussion.comments.any? %>
+   <h4>Comments</h4>
+   <% @discussion.comments.each do |comment| %>
+     <p>
+       <strong><%= comment.user.name %>: </strong>
+       <%= comment.body %>
+     <p>
+   <% end %>
+ <% end %>


<%= link_to 'Edit', edit_discussion_path(@discussion) %> |
<%= link_to 'Back', discussions_path %>
```

サーバを起動し、http://localhost:3000 をブラウザで開いてユーザ登録を行います。

![User 1 created](/images/blog/2019-07-10-action-mailbox/user-1-created.png)

少し寂しいので Mario を追加します。

![users-index](/images/blog/2019-07-10-action-mailbox/users-index.png)

続いてディスカッションを追加します。議題は何でも良いので、「好きなゲーム」にしておきます。

![create discussion](/images/blog/2019-07-10-action-mailbox/create-discussion.png)

メールの返信によって、このページに Comments(今回の場合はゲームのタイトル) が追加されていくイメージです。

ということで ```reply-1@example.com``` 宛に、各ユーザからメールを送信する必要があります。
Rails 6 では開発用にメール作成画面を提供してくれているので、そこで作業して送信します。

http://localhost:3000/rails/conductor/action_mailbox/inbound_emails/new

From, To, Body だけ入力して送信してみます。

![mail form](/images/blog/2019-07-10-action-mailbox/mail-form.png)

送信すると、送信したメールの詳細画面へ遷移します。Full email source をクリックすると、
送られたメールの詳細を確認する事ができます。

![mail show](/images/blog/2019-07-10-action-mailbox/mail-show.png)

Back to all inbound emails を押下すると、メール一覧画面が表示されます。
delivered になっているので、無事に送信されたようです。

![mail index](/images/blog/2019-07-10-action-mailbox/mail-index.png)

ちなみにメール送信は ```ActiveJob``` で実行されます。

Mario からも適当に好きなゲームをコメントさせ、これら２つのコメントが /discussions/1 に表示されていることを確認します。

![comments](/images/blog/2019-07-10-action-mailbox/comments.png)

# 所感
文章は少し長くなってしまいましたが、実装自体は比較的容易でした。

しかしやはり、使い所がイマイチピンと来てないところがあるので、今後の案件や
何かの記事で良いユースケースを見つけたら良いなーと思っています。

でも Basecamp で試してある程度の成果というか、メリットがあったから rails に移行したのだろうとは思うので、
　やっぱりそういう良いところは何かしらあるのだとは思います、自分が知らないだけで。

今回触れなかった、HTMLメールや添付ファイルの扱いについては良い使い道がありそうな気がするので、
今後機会があれば試しておきたいと思います。

### Source
https://github.com/Lynns0416/action_mailbox_demo
