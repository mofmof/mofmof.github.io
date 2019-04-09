---
layout: blog
title: Rails×GraphQLやる
category: blog
tags: [Ruby on Rails, GraphQL]
summary: RailsでGraphQLに触れてみます。適当に作ったみんな大好きTODOアプリベースで触ってくので超絶わかりやすいと思います。
author: iwai
image: /images/blog/2019-04-09-graphql-with-rails/GraphQL Logo.png
---

なんかよくわかんないけどなんか熱いらしいGraphQLを触ってみようと思いました。実家のような安心感のあるRailsをベースに学んでいこうと思います。

<br>

## TL;DR

TODOアプリを適当に作り、タスクの取得とタスクの検索をGraphQLでやってみる。とりあえずQueryだけ触ります。

コードは[こちら](https://github.com/yubachiri/todo)

masterをベースにやっていきます。graphqlブランチが今回・次回分の作業が完了したものです。

<br>

## GraphQLとは

> GraphQLはFacebookにより開発されたオープンソースの言語です。API作成の仕組みとしてRESTの代わりに使えます。RESTはAPIの設計と実装に使う概念上の設計モデルですが、GraphQLは標準化された言語、型付け、仕様を持ちクライアントとサーバー間を強力に結びつけます。異なるデバイス間の通信に標準化された言語があることで、大型かつクロスプラットフォームのアプリ開発がよりシンプルになります。

[アプリ開発の流れを変える「GraphQL」はRESTとどう違うのか比較してみた](https://www.webprofessional.jp/rest-2-0-graphql/)より引用

WebAPIの規格であり、better RESTを謳っています([HOW TO GRAPHQL](https://www.howtographql.com/basics/1-graphql-is-the-better-rest/))。色々削ぎ落として言うと、クエリ言語です。これに則ってAPIを定義するといい感じになるようになってます。どういうものなのかは見ていけば感覚で掴めると思います。

## ベースとなるRailsアプリケーションを作る

[これ](https://github.com/yubachiri/todo)をcloneすればOKです。Dockerで動くのでお手軽ですよ。
ただdeviseでuser作るとかscaffoldでtask作るとかしただけですし、お好みで適当なアプリを用意していただいてもよいです。

## GraphQLがどんなもんなのか軽く触ってみる

さて、コードを触っていきましょう。

まずはgemの導入です。

Gemfileに追加
```
gem 'graphql', '1.7.7'
gem 'graphiql-rails'
```
で`bundle install`

したら`rails g graphql:install`します。
`app/graphql`以下に色々できたり、`routes.rb`にもちょろっと追加されたりしますね。

```
  if Rails.env.development?
    mount GraphiQL::Rails::Engine, at: "/graphiql", graphql_path: "/graphql"
  end
  post "/graphql", to: "graphql#execute"
```

これによって諸々準備が整います。というかもう動くので、試しに触ってみましょう。さっき入れた`graphiql-rails`が、ブラウザ上でGraphQLを触れるようにしてくれています。

サーバを立ち上げ、[http://localhost:3000/graphiql](http://localhost:3000/graphiql)へ。githubからクローンしている場合は[http://localhost:3008/graphiql](http://localhost:3008/graphiql)です。

こんな画面が出るはず！

![graphiql](/images/blog/2019-04-09-graphql-with-rails/graphiql.png)

自動生成されているもので動作確認できるので、してみましょう。左側に
```
{
  testField
}
```
と入れて、ページ上部の実行ボタンを押してみてください。右側に
```
{
  "data": {
    "testField": "Hello World!"
  }
}
```
が表示されたら成功です！いえーい！

![graphi_result](/images/blog/2019-04-09-graphql-with-rails/graphi_result.png)


## ちょっとGraphQLの話

GraphQLではデータの取得や更新など、全て基本的にPOSTリクエストで行います。そんで、リクエスト時に何を渡すかで何が返ってくるかを制御します。

GraphQLはリクエストに対して、QueryもしくはMutationで対応します。
それぞれデータ取得系/データ更新系という使い分けです。

さっきの操作では、`/graphql`に`{testField}`をPOSTし、結果"Hello world!"が返ってきましたね。これは`graphql_controller`の`excute`から`/app/graphql/types/query_type.rb`の`field :testField`内`resolve`が実行された結果です。

ぼちぼちTODOアプリを触りつつ解説していきますね。

## 型の定義

これから下記のような`Task`のレコードが取得できるようGraphQLをいじります。

```
# == Schema Information
#
# Table name: tasks
#
#  id         :bigint(8)        not null, primary key
#  title      :string           not null
#  deadline   :datetime         not null
#  status     :integer          not null
#  user_id    :bigint(8)        not null
#  created_at :datetime         not null
#  updated_at :datetime         not null
#  priority   :integer          default("top"), not null
#

class Task < ApplicationRecord
  belongs_to :user

  enum status: {
    untouched: 0,
    started: 1,
    completed: 2
  }

  enum priority: {
    top: 0,
    next: 1,
    other: 2
  }
end
```

GraphQLは受け取るもの・返すものを考える際に型を検証します。今回はTaskを取得したいので、Task型を定義しましょう。generatorを使って生成します。

`rails g graphql:object Task id:ID! title:String! status:Int! priority:Int!`

下記のような型定義ファイルができます。fieldの名前はカラム名と一致させましょう。

/app/graphql/types/task_type.rb
```
Types::TaskType = GraphQL::ObjectType.define do
  name "Task"
  field :id, !types.ID
  field :title, !types.String
  field :status, !types.Int
  field :priority, !types.Int
end
```

## Taskを取得するQuery

そしたら`/app/graphql/types/query_type.rb`の編集です。デフォルトのfieldは消して、taskのレコードを返すQueryを定義しましょう。

```
Types::QueryType = GraphQL::ObjectType.define do
  name "Query"

  field :firstTask, Types::TaskType do
    description "return first task"
    resolve ->(obj, args, ctx) {
      Task.first
    }
  end
end
```

ここまでできたら/graphiqlでfirstTaskにクエリを投げてみてください。あ、taskのレコードはあらかじめ作成しておいてくださいね。

![graph_error](/images/blog/2019-04-09-graphql-with-rails/graph_error.png)

おっと、エラーですね。

これはTaskモデルに要求するfieldを指定していないため発生するものです。クライアント側で欲しい情報を明示しておく必要があるんですね。

![graph_task](/images/blog/2019-04-09-graphql-with-rails/graph_task.png)

無事取れました。

## でもTask.firstしか取れないんじゃ困る

そんな声にお答えして、今回はおしまいにします。firstTaskに引数を渡せば万事OKですよ。それは命名と実装がずれるので、渡したIDのタスクを返すqueryを定義してみましょうか。

```
Types::QueryType = GraphQL::ObjectType.define do
  name "Query"

  field :firstTask, Types::TaskType do
    description "return first task"
    resolve ->(obj, args, ctx) {
      Task.first
    }
  end

  # 追加
  field :findTaskBy, Types::TaskType do
    description "return a task"
    argument :id, types.Int
    resolve ->(obj, args, ctx) {
      Task.find_by(id: args.id)
    }
  end
end
```

`argument :id, types.Int`を書いてやることで、findTaskByはidがキーのハッシュを受け取ることができるようになります。受け取った引数には、resolve内でargs経由でアクセスできます。

もしこれを定義しないで引数にidを渡すとこんなエラーが出ます。

```
{
  "errors": [
    {
      "message": "Field 'findTaskBy' doesn't accept argument 'id'",
      "locations": [
        {
          "line": 2,
          "column": 14
        }
      ],
      "fields": [
        "query",
        "findTaskBy",
        "id"
      ]
    }
  ]
}
```

さて、`/graphiql`で動かしてみましょう。引数はこんな感じで渡せばOKです。

```
{
  findTaskBy(id: 1){
    id
    title
    status
    priority
  }
}
```


結果

![graph_task_by_id](/images/blog/2019-04-09-graphql-with-rails/graph_task_by_id.png)

## おまけ

こんなにも徒然とfieldを追加してったら絶対つらみ出るじゃん！ということなのですが、GraphQLにはSchemaというものが存在しており、gem graphqlにそれを吐かせることができます。百聞は一見に如かず。

Rakefile
```
# Add your own tasks in files placed in lib/tasks ending in .rake,
# for example lib/tasks/capistrano.rake, and they will automatically be available to Rake.

require_relative 'config/application'
require 'graphql/rake_task' # 追加

Rails.application.load_tasks
GraphQL::RakeTask.new(schema_name: 'TodoSchema') # 追加
```

実行

`rake graphql:schema:dump`

結果

/schema.graphql
```
type Mutation {
  # An example field added by the generator
  testField: String
}

type Query {
  # return a task
  findTaskBy(id: Int): Task

  # return first task
  firstTask: Task
}

type Task {
  id: ID!
  priority: Int!
  status: Int!
  title: String!
}
```

データ取得系APIの定義が`type Query`以下に記載されています。返すのはTaskTypeということですね。わかりやすい。

こいつがあると`/graphiql`の画面右上からもAPI定義を確認できるようになります。

![graph_schema](/images/blog/2019-04-09-graphql-with-rails/graph_schema.png)

## 以上

次回はTODOアプリの画面側からごにょごにょしようと思います。
