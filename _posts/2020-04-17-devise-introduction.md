---
layout: blog
title: "[Rails] devise の使い方（導入からログアウトまで）"
category: blog
tags: [Rails, devise, gem, 入門, 最小構成]
summary: 今回はRailsで認証系のデファクトスタンダートとも言えるdevise を実践的に解説していきます。この記事を最後まで読み終わった時、あなたはdeviseの導入から実装、ログアウトまでを完全に理解し、自分のプロジェクトにdeviseを実装することができるようになるでしょう。
author: akito-n
#image: none
---
## 言語、ライブラリのバージョン

- Ruby 2.6.3
- Rails 6.0.2.2
- devise 4.7.1
- テンプレートはslimを使用

# 目次

- [deviseの概要](#anc1)
- [セットアップ](#anc2)
- deviseをmodelに適用する
    - [新規作成の場合](#anc3)
    - [既存モデルに適用する場合](#anc4)
- コントローラーをカスタマイズする
    - [新規登録処理](#anc5)
    - [ログアウト処理](#anc6)
- [ログインに伴う制御を行う](#anc7)

<a id="anc1"></a>

# deviseの概要

- Webアプリケーションを作成する際にほぼ必ずと言っていいほど必要になる認証系の機能をごそっと用意してくれるGem。アカウントの登録、ログイン、パスワードの再設定、ログイン情報のトラッキング、ログイン情報の保持、タイムアウト設定、ロック機能などのおおよそWebアプリで必要な機能のほとんどを提供してくれる。またこれらはモジュールに分解されており、全部の機能を使うのではなく自分のアプリケーションに必要な機能を選んで導入することが可能です。

- また、複数モデルの同時ログインにも対応しています。完全に影響を分割することができるので管理ユーザーモデルと一般ユーザーモデルが１つのアプリに存在する場合でも両方にdeviseを導入することができます。

## 始める前に
## erb→slimにする
erbでいく方はここは読み飛ばしてしまって構いません。slimでいく方はここでslimの設定をしておきましょう。

slim拡張子のファイルを自動で生成していって欲しいので、Gemfileに

```
gem 'slim-rails'
```
を記載し、`$ bundle install` します。

これだけで今後のファイル自動生成関係のコマンドでslimファイルが生成されます。

### すでにerbファイルがあるが変えたい場合
生成済みのerbをslimにいい感じに変えたい場合、html2slimを使うと手っ取り早いです。少なければ手動でもなんとかなるけど今後のためにも入れておきましょう。

```
$ gem install html2slim
```

で、インストールしたら下記コマンドをプロジェクト直下のターミナルで実行します。

```
$ for i in app/views/**/*.erb; do erb2slim $i ${i%erb}slim && rm $i; done
```
すでにerbファイルが存在する場合には上記のコマンドを打ってerb拡張子のファイルを全てslimに変更しておきましょう。

<a id="anc2"></a>

# セットアップ

何はともあれまずはセットアップです。deviseはgemなので、使用するためにはgemをインストールしておく必要があります。

Gemfile
```
source ‘https://rubygems.org'
git_source(:github) { |repo| “https://github.com/#{repo}.git” }
.
.
中略
.
.
gem ‘devise’ #追記

```

上記のようにGemfileに

```
gem ‘devise’
```

と追記します。
反映させるために
```
$ bundle install
```
無事インストールできたら、
```
$ rails g devise:install
```
というコマンドが使えるようになっているのでプロジェクト直下のターミナルで
```
$ rails g devise:install
```
を入力すると下記のようなログが出力されます。

```
      create  config/initializers/devise.rb
      create  config/locales/devise.en.yml
===============================================================================

Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

===============================================================================
```

このコマンドで生成されるのは
1. deviseの設定ファイル
2. deviseの英語用のlocaleファイル

です。また、セットアップ方法も出力されるのでそれに従って進めていきます。

1.デフォルトのurlオプションを設定する
```
 1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

```
ここに書かれているように、deviseで使用する`default_url_option`を設定します。メールの存在確認などで生成されるリンクのurlの基本設定です。

開発環境では` config/environments/development.rb`に
```
config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }
```
と設定します。
本番環境では、hostの部分にアプリケーションのドメインを入力しましょう。
```
config.action_mailer.default_url_options = { :host => ドメイン, :protocol => ‘https’}
```

2. トップページへアクセスしたときのルーティングを確立する

これは既存のアプリケーションなどにdeviseを導入する場合など、トップページにアクセスした場合のルーティングが設定されている場合はスキップしても構わないです。今回は作っていないので`routes.rb`ファイルに適当にコントローラーとルーティングを設定します。
とりあえずhomeコントローラーを作成
```
rails g controller home index
```
`routes.rb`に`root to: ‘home#index’`と記載
rails sを行ってlocalhostにアクセスした場合にこのようになればOKです。(変換前なのでerbになってます。)

![image-01](/images/blog/2020-04-17-devise-introduction/image-01.png)

3. flash メッセージを表示するために、views/layouts/application.html.slimにフラッシュメッセージを表示させる部分を追記します。

```
doctype html
html
  head
    title
      | DeviseCommentaly
    = csrf_meta_tags
    = csp_meta_tag
    = stylesheet_link_tag ‘application’, media: ‘all’, ‘data-turbolinks-track’: ‘reload’
    = javascript_pack_tag ‘application’, ‘data-turbolinks-track’: ‘reload’
  body
    p.notice = notice  #追記
    p.alert = alert     #追記
    = yield

```
最後に、deviseのviewファイルをカスタマイズするために
```
rails g devise:views
```
コマンドを入力します。
このコマンドで
```
app/views/devise/confirmations/new.html.slim
app/views/devise/mailer/confirmation_instructions.html.slim
app/views/devise/mailer/email_changed.html.slim
app/views/devise/mailer/password_change.html.slim
app/views/devise/mailer/reset_password_instructions.html.slim
app/views/devise/mailer/unlock_instructions.html.slim
app/views/devise/passwords/edit.html.slim
app/views/devise/passwords/new.html.slim
app/views/devise/registrations/edit.html.slim
app/views/devise/registrations/new.html.slim
app/views/devise/sessions/new.html.slim
app/views/devise/shared/_error_messages.html.slim
app/views/devise/shared/_links.html.slim
app/views/devise/unlocks/new.html.slim
```
というファイル群が生成されます。これらはパスワード再設定やメールアドレス変更のためのviewファイル群になります。
ここで作成したページ群は後ほど使用します。

<a id="anc3"></a>
# deviseをmodelに適用する（model新規作成の場合）


deviseで制御するmodelを作成していきます。よくあるのはUserモデルになると思うので、今回はUserモデルを作成していきます。

```
$ rails g devise user
```

このコマンドでデバイス用のマイグレーションファイル、model、routes.rbにdeviseの提供する各ルーティングを提供する`devise_for :users`というルーティングが記載されます。

migrationfileの中身はこんな感じです

```
# frozen_string_literal: true

class DeviseCreateUsers < ActiveRecord::Migration[6.0]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.string :email,              null: false, default: “”
      t.string :name,               null: false  # 追加
      t.string :encrypted_password, null: false, default: “”

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      # t.integer  :sign_in_count, default: 0, null: false
      # t.datetime :current_sign_in_at
      # t.datetime :last_sign_in_at
      # t.inet     :current_sign_in_ip
      # t.inet     :last_sign_in_ip

      ## Confirmable
      # t.string   :confirmation_token
      # t.datetime :confirmed_at
      # t.datetime :confirmation_sent_at
      # t.string   :unconfirmed_email # Only if using reconfirmable

      ## Lockable
      # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
      # t.string   :unlock_token # Only if unlock strategy is :email or :both
      # t.datetime :locked_at


      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```

コメントアウトされているいくつかの定義は冒頭で紹介したいくつかの機能を実現するために使用されるカラム群です。

今回は使用しないのでデフォルトのままでいきたいと思います。また、他のカラムを	Userモデルに持たせたい場合はここに追記することもできます。今回はnameというカラムを追加して名前が登録できるようにしました。

ちなみに生成されたモデルはこんな感じ。
```
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable
end
```

バージョンによって多少の差はありますが、今回の構成だと

```
database_authenticatable   #=> ログイン時に正当性を検証する機能。認証はPOSTリクエストかHTTPベーシック認証が使える
registerable    #=> 新規登録処理を実行しDBに保存する機能、またそのアカウントを編集、削除できるようにする機能
recoverable    #=> パスワードのリセットを行い、変更手順を送る機能
rememberable   #=>   保存されたCookieからユーザー情報を記憶するためのトークンを管理する機能
validatable    #=> デフォルトでe-mailとパスワードの検証を提供する。optionであるためカスタマイズができる。

```
という機能がmodel生成時点で有効になっています。

続いて作成されたmigrationファイルを実行し、定義を行います。
```
$ rails db:migrate
```
これでDeviseコマンドで作成されたマイグレーションファイルを実行できました。

ログイン画面がうまく表示されるか確認してみましょう。
サーバーを起動し、`users/sign_up` にアクセスしてみます。

![image-02](/images/blog/2020-04-17-devise-introduction/image-02.png)

このように表示されていることが確認できれば成功です。

<a id="anc4"></a>
# deviseをmodelに適用する（modelがすでに存在する場合）


あまりないケースかもしれませんが、既にUserモデルが存在しており、そのモデルに対してdeviseを使用したい場合でも、
```
$ rails g devise user
```
コマンドを実行することで`create_table`の部分が`change_table`になり、特に何かしなくてもdeviseの機能が使えるようになります。

一点留意点として既に同名のカラムが存在している場合はmigration実行時にエラーになりますので、その場合は追加する記載の該当部分を削除してから実行を行うようにすれば良いです。


<a id="anc5"></a>

# Deviseでコントローラーをカスタマイズする(新規登録処理)

先ほど`null false` オプションをつけてUserモデルにnameカラムを追加したので`devise:views`で作成したファイルにnameカラムに値を入れられるように適当にfieldを追加しておきます。

/app/views/devise/registrations/new.html.slim
```
h2
  | Sign up
= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f|
  = render “devise/shared/error_messages”, resource: resource
  .field
    = f.label :name
    br
    = f.text_field :name
  .field
    = f.label :email
    br
    = f.email_field :email, autofocus: true, autocomplete: "email"
  .field
    = f.label :password
    - if @minimum_password_length
      em
        | (
        = @minimum_password_length
        |  characters minimum)
    br
    = f.password_field :password, autocomplete: “new-password”
  .field
    = f.label :password_confirmation
    br
    = f.password_field :password_confirmation, autocomplete: "new-password"
  .actions
    = f.submit "Sign up"
= render “devise/shared/links"
```

ただ、このままだとsign_upの処理はdeviseのデフォルトのコントローラーが処理を行うため、独自に追加したnameというカラムはそのままでは保存できません。
このような場合はdeviseのコントローラーをオーバーライドして、必要な処理を追加してあげる必要があります。今回はUserモデルに対してなので

```
$ rails g devise:controllers users
```
とターミナルに入力します。
出力されるログは以下。


```
 create  app/controllers/users/confirmations_controller.rb
      create  app/controllers/users/passwords_controller.rb
      create  app/controllers/users/registrations_controller.rb
      create  app/controllers/users/sessions_controller.rb
      create  app/controllers/users/unlocks_controller.rb
      create  app/controllers/users/omniauth_callbacks_controller.rb
===============================================================================

Some setup you must do manually if you haven't yet:

  Ensure you have overridden routes for generated controllers in your routes.rb.
  For example:

    Rails.application.routes.draw do
      devise_for :users, controllers: {
        sessions: 'users/sessions'
      }
    end

===============================================================================
```

このコマンドで新規登録やログインなどそれぞれの処理の際に呼ばれるコントローラーが生成されます。また、オーバーライドしたコントローラーにルーティングを渡したいので、例にあるように`routes.rb`に
```
 devise_for :users, controllers: {
        registrations: 'users/registrations’
        sessions: 'users/sessions'
      }
```

とルーティングを設定します。この記述によってusersにおける処理の行き先を指定しています。

ルーティングを指定し、コントローラーが自分の作成したものを経由するようになったらnameカラムを保存できるようにcontrollerに手を加えていきます。ちなみに自動生成されたusers/registrations_controller.rbは下記のようになっています。

```
# frozen_string_literal: true

class Users::RegistrationsController < Devise::RegistrationsController
  # before_action :configure_sign_up_params, only: [:create]
  # before_action :configure_account_update_params, only: [:update]

  # GET /resource/sign_up
  # def new
  #   super
  # end

  # POST /resource
  # def create
  #   super
  # end

  # GET /resource/edit
  # def edit
  #   super
  # end

  # PUT /resource
  # def update
  #   super
  # end

  # DELETE /resource
  # def destroy
  #   super
  # end

  # GET /resource/cancel
  # Forces the session data which is usually expired after sign
  # in to be expired now. This is useful if the user wants to
  # cancel oauth signing in/up in the middle of the process,
  # removing all OAuth session data.
  # def cancel
  #   super
  # end

  # protected

  # If you have extra params to permit, append them to the sanitizer.
  # def configure_sign_up_params
  #   devise_parameter_sanitizer.permit(:sign_up, keys: [:attribute])
  # end

  # If you have extra params to permit, append them to the sanitizer.
  # def configure_account_update_params
  #   devise_parameter_sanitizer.permit(:account_update, keys: [:attribute])
  # end

  # The path used after sign up.
  # def after_sign_up_path_for(resource)
  #   super(resource)
  # end

  # The path used after sign up for inactive accounts.
  # def after_inactive_sign_up_path_for(resource)
  #   super(resource)
  # end
end


```
このファイルの中の
`before_action :configure_sign_up_params, only: [:create]`
と

```
 def configure_account_update_params
  devise_parameter_sanitizer.permit(:account_update, keys:[:attribute])
 end
```

のコメントアウトを外しておきます。
nameカラムに値を入れたいので、`:attribute`となっている部分を`:name`に変えてあげます。複数ある場合も同様です。

この部分を変更することによって任意の値を新規登録時に保存させることができます。
この段階ではconfirmableを有効にしていないので、新規登録が正常に実行されるとdeviseがログイン処理も行います。
ログイン後はデフォルトだと`root_path`にリダイレクトされるので、今回はHome#indexに遷移します。

<a id="anc6"></a>

# Deviseでコントローラーをカスタマイズする(ログアウト)

一連の動作を何度でも行えるように、ログアウト処理をつけておきましょう。
ログイン処理後に遷移するhome#indexにログアウト用のリンクをつけましょう。

```

h1
  | Home#index
p
  | Find me in app/views/home/index.html.slim

= link_to 'ログアウト', destroy_user_session_path, method: :delete

```

これでボタンを押せばログアウトできるようになりました。

<a id="anc7"></a>

# ログインに伴う制御を行う

現状までの状態で新規登録を行いそのままログイン、ログアウトまでの流れを実装できましたが、このような認証機能を入れたい理由の大半はログインしなければアクセスできないページが必要だからということが多いです。
今回はログインしていればアクセスすることができ、新規登録の際に登録したnameを表示するだけの簡単なmypageを作ってそれを検証してみます。

```
$ rails g controller mypage show
```

こんな感じでmy page 用のコントローラーを作成します。今回使用するのはshowメソッドにしようと思うのでshow もこのタイミングで設定してしまいます。
routes.rbには自動で`get ‘mypage/show’`のようなメソッドが生成されているのでこのままアクセスできます。今回はそのままこのルーティングを使ってアクセスしていきます。

![image-03](/images/blog/2020-04-17-devise-introduction/image-03.png)

ではこのページに制御をかけていきます。
Deviseを導入すると使えるようになるメソッドがいくつか提供されていて、その中で認証を行うためには
`authenticate_user!`というメソッドを使用します。今回はbefore_actionを使用してmypageコントローラー全体に認証を設定します。

/app/controllers/mypages_controller.rb
```
class MypageController < ApplicationController
  before_action :authenticate_user!  #追加

  def show
  end
end

```

これだけです。これでDeviseがログイン済みなのかどうか判定してくれます。この状態で先ほどの `mypage/show`にアクセスしてみましょう。

ログアウトした状態で`mypage/show`にアクセスするとログインページにリダイレクトされ、画像のようにデフォルトメッセージとして
`You need to sign in or sign up before continuing.`と表示されます。

![image-04](/images/blog/2020-04-17-devise-introduction/image-04.png)

これでログインしているときにしか表示させないページという制御ができるようになったので、このページにログインしたユーザーの名前を表示していきます。

現在ログインしているユーザーを取得するにはdeviseをインストールすることで使えるようになったメソッドを使います。Userモデルに適用しているので`current_user`というメソッドが利用できます。userという部分は適用するモデルによって変化するので、もしPersonというモデルに適用している場合であれば`current_person`になります。

mypage/show.html.slimに下記のように記載します

/app/views/mypages/show.html.slim
```
h1 Mypage#show
p Find me in app/views/mypage/show.html.slim

//追加
= current_user.name
```

この状態でログインして`mypage/show`にアクセスしてみましょう。
今回の例ではnameには「test」という名前で登録をしているので、testとmypage/showに表示されれば成功です。

![image-05](/images/blog/2020-04-17-devise-introduction/image-05.png)


