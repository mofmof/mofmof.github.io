---
layout: blog
title: Rails5 + Doorkeeper + DeviseでOAuthサーバーを実装する
category: blog
tags: [Rails, Ruby on Rails, OAuth, Doorkeeper, Devise]
summary: Alexaと既存のWEBサービスのユーザーアカウントを連携させるにはoauthのプロバイダを実装しないといけないっぽくて、Railsでやってみようと思った。
author: aharada
---

最近ちょっとAmazon Echo Alexaのスキル開発にハマってまして、「[【Amazon Echo入門#6】AlexaちゃんとTwitterアカウントを連携してみる](/blog/amazon-echo-alexa-skill-link-twitter.html)」というエントリを書いたりしてます。

どうやらAlexaと既存のWEBサービスのユーザーアカウントを連携させるにはoauthのプロバイダを実装しないといけないっぽくて、Railsでやってみようと思った。

oauthのgemとRails5の問題？っぽいところでめっちゃハマったので、犠牲者を増やさないためにもここに記しておきます。

## 参考リンク

- [【Rails5】Doorkeeper gemでOAuth2.0のためのAPIを作って、rubyクライアントで呼び出す](http://kansiho.hatenablog.com/entry/2017/09/02/%E3%80%90Rails5%E3%80%91Doorkeeper_gem%E3%81%A7OAuth2.0%E3%81%AE%E3%81%9F%E3%82%81%E3%81%AEAPI%E3%82%92%E4%BD%9C%E3%81%A3%E3%81%A6%E3%80%81%E3%83%86%E3%82%B9%E3%83%88%E3%81%99%E3%82%8B%E6%B5%81%E3%82%8C)
- [DoorkeeperとDeviseでOAuth2によるログイン機能を作る](https://qiita.com/kyonsuke19101/items/407f3cdfec38d1108e9d)
- [RailsでAPIでのOAuth2認証をdoorkeeper + deviseで実装する](https://qiita.com/arakaji/items/39818b207f9c1c3c4058)
- [doorkeeperを使ってomniauthのprovider&client作成を懇切丁寧にまとめた](https://qiita.com/8398a7/items/9b13ac2e7c401dee4a39)

## OAuthサーバ側

新しいRailsアプリケーションを生成する。名前はなんでも良いです。

```
$ rails new trash-day-server
```

Gemfile

```
gem 'devise'
gem 'doorkeeper'
gem 'omniauth'
gem 'oauth2'
```

Deviseのインストールやテーブルの生成。

```
$ bundle install
$ rails g devise:install
$ rails g devise user
$ rake db:migrate
```

Doorkeeperのインストールやテーブルの生成。

```
$ rails generate doorkeeper:install
$ rails generate doorkeeper:migration
```

デフォルトままでRails5だとmigrate時にエラーがでるのでmigrationファイルを編集する。

20171118031209_create_doorkeeper_tables.rb

```ruby
class CreateDoorkeeperTables < ActiveRecord::Migration[4.2]
```

```
$ rake db:migrate
```

config/initializers/doorkeeper.rb

```ruby
resource_owner_authenticator do
  current_user || warden.authenticate!(scope: :user)
end
```

クライアント側からのユーザー情報取得用の処理を実装しておく

app/controllers/api/v1/api_controller.rb

```ruby
class Api::V1::ApiController < ApplicationController
  private
  def current_resource_owner
    User.find(doorkeeper_token.resource_owner_id) if doorkeeper_token
  end
end
```

app/controllers/api/v1/users_controller.rb

```ruby
class Api::V1::UsersController < Api::V1::ApiController
  before_action :doorkeeper_authorize!
  respond_to :json

  def me
    respond_with current_resource_owner
  end
end
```

config/routes.rb

```
Rails.application.routes.draw do
  devise_for :users
  use_doorkeeper
  # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html

  namespace :api do
    namespace :v1 do
      get '/me' => 'users#me'
    end
  end

  root to: 'home#show'
end
```

```
$ rails s
```

`http://localhost:3000/oauth/applications`にアクセスするとログイン画面が表示されるので、適当にサインアップし、再度同じURLにアクセスする。

アプリケーションを登録して、application_idとsecretを発行する。

Callback urls: http://localhost:3001/users/auth/doorkeeper/callback

![アプリケーション](/images/blog/2017-11-18-rails5-devise-doorkeeper/application.png)

コマンドでテストしてみる。

```
$ rails c

> require 'oauth2'
> client_id= 'b1560de64a5faf96d6f44e1ca32c0473d88260cc3b6472c501acfb1180821d41'
> client_secret= 'eb49bd18ab22f45341686c86e302d69cc58c371aaf0ceb24139a793a31a44355'
> site = 'http://localhost:3000'
> redirect_uri = 'http://localhost:3001/users/auth/doorkeeper/callback'

> client = OAuth2::Client.new(client_id, client_secret, :site => site)
> login_url =  client.auth_code.authorize_url(redirect_uri: redirect_uri)
```

成功すれば問題なし。サーバ側はこれでOK。

## クライアント側

新しいRailsアプリケーションを生成する。名前はなんでも良い。

```
$ rails new trash-day-client
$ cd trash-day-client
```

Gemfile

```
gem 'devise'
gem 'omniauth'
gem 'omniauth-oauth2'
gem 'oauth2'
```

```
$ bundle install
```

Deviseのインストールとテーブルの生成。

```
$ rails g devise:install
$ rails g devise user
$ rake db:migrate
```

OAuthの使用する場合は追加のフィールドが必要。

```
$ rails g migration AddUidToUser
```

```ruby
class AddUidToUser < ActiveRecord::Migration[5.1]
  def change
    add_column :users, :uid, :string
    add_column :users, :provider, :string
  end
end
```

```
$ rake db:migrate
```

サーバ側で取得したapp_idとsecretをDeviseの設定に追記する。

config/initializers/devise.rb

```ruby
require File.expand_path('lib/omniauth/strategies/doorkeeper', Rails.root)
Devise.setup do |config|
  config.omniauth(:doorkeeper, 'b1560de64a5faf96d6f44e1ca32c0473d88260cc3b6472c501acfb1180821d41', 'eb49bd18ab22f45341686c86e302d69cc58c371aaf0ceb24139a793a31a44355')
end
```

lib/omniauth/strategies/doorkeeper.rb

```ruby
module OmniAuth
  module Strategies
    class Doorkeeper < OmniAuth::Strategies::OAuth2
      option :name, :doorkeeper
      option :client_options, site: 'http://localhost:3000', authorize_path: '/oauth/authorize'

      uid { raw_info['id'] }

      info do
        { email: raw_info['email'] }
      end

      def raw_info
        @raw_info ||= JSON.parse(access_token.get('api/v1/me').response.body)
      end

      def callback_url
         full_host + script_name + callback_path
      end
    end
  end
end
```

認証成功後のコールバック処理を実装する。

app/controllers/users/omniauth_callbacks_controller.rb

```ruby
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def doorkeeper
    @user = User.find_or_create_with_doorkeeper(request.env['omniauth.auth'])

    if @user.persisted?
      sign_in(@user)
      set_flash_message(:notice, :success, kind: 'doorkeeper') if is_navigational_format?
      redirect_to '/'
    else
      session['devise.doorkeeper_data'] = request.env['omniauth.auth']
      redirect_to root_url, alert: 'Doorkeeper ログインに失敗しました'
    end
  end
end
```

app/models/user.rb

```ruby
class User < ApplicationRecord
  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable, :omniauthable

  def self.find_or_create_with_doorkeeper(auth)
    user = self.find_by(provider: auth.provider, uid: auth.uid )
    return user unless user.nil?

    self.create(
      email: auth.info.email,
      provider: auth.provider,
      uid: auth.uid,
      password: Devise.friendly_token[0, 20]
    )
  end
end
```

ポート3001番で起動する。

```
$ rails s -p 3001
```

`http://localhost:3001/users/sign_in`を開くとログイン画面が開くので「Sign in with Doorkeeper」をクリック。

![Authorize](/images/blog/2017-11-18-rails5-devise-doorkeeper/authorize.png)

Authorizeをクリックして、ログインが成功することを確認。

![ログイン成功](/images/blog/2017-11-18-rails5-devise-doorkeeper/success.png)

## ハマりどころ

Rails5とomniauth-oauth2の組み合わせで発生するっぽい？バグにハマりました。

どこをどう見てもサンプルと同じように実装しているのに、サーバからクライアント側にcallbackするタイミングでエラーになってしまいます。

クライアント側のエラーログ

```
E, [2017-11-18T13:15:56.492767 #32292] ERROR -- omniauth: (doorkeeper) Authentication failure! invalid_credentials: OAuth2::Error, invalid_grant: The provided authorization grant is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client.
{"error":"invalid_grant","error_description":"The provided authorization grant is invalid, expired, revoked, does not match the redirection URI used in the authorization request, or was issued to another client."}
Processing by Users::OmniauthCallbacksController#failure as HTML
  Parameters: {"code"=>"72da2b48acfee81df7325c175a31b273fd3b1a368d4995d2b626e30e357339c2", "state"=>"56ec955d52f6935b846a80604c30ff2ce5722caeb7eb27ce"}
Redirected to http://localhost:3001/users/sign_in
Completed 302 Found in 8ms (ActiveRecord: 0.0ms)
```

doorkeeperのソースコードを読んでいったところ、`redirect_uri`をバリデーションしている箇所があるのですが、GETパラメータ付きのURLと付いていないURLで比較してvalidationしていたため`redirect_uri`が無効だよってエラーになってたみたい。

- 参考
  - [https://github.com/doorkeeper-gem/doorkeeper/issues/732](https://github.com/doorkeeper-gem/doorkeeper/issues/732)
  - [https://github.com/intridea/omniauth-oauth2/issues/81](https://github.com/intridea/omniauth-oauth2/issues/81)


クライアント側の`strategies/doorkeeper.rb`に以下を追記してオーバーライドすれば解消します。

```ruby
def callback_url
  full_host + script_name + callback_path
end
```

このバグに4時間近く持ってかれたわ。
