---
layout: blog
title: Apollo使ってGraphQLを自由自在に操（りたい）
category: blog
tags: [Ruby on Rails, GraphQL, Apollo]
summary: RailsでGraphQLに触れてみますその２。前回より新しいバージョンのGemを使いつつ、Apolloを導入してみます。
author: iwai
image: /images/blog/2019-04-15-graphql-latest-apollo/apollo_logo.png
---

前回は画面からqueryを投げてデータを取得するところまでやってみました。今回はもうちょっと実用的な形でTODOアプリを進化させようと思います。

前回[Rails×GraphQLやる - もふもふ技術部](/blog/graphql-with-rails.html)


## TL;DR

がしかし、導入したGemのバージョンが古いので、これを最新版にアップデート＆最新の構成でもう一度前回と同じ仕様の実現までやってから、GraphQLをいい感じに扱えるApolloというライブラリを導入していじっていこうと思います。


## 前回のやり直し

前回はgemのバージョンを1.7.7指定していたのですが、今日(2019/4/15)時点では1.9.4が最新です。ジェネレータを使って生成されるファイルの内容も以前と変わっているので、最新キャッチアップをしてから先に進もうと思います。コードは[これ](https://github.com/yubachiri/todo)のmasterからブランチ切ってやります。

まずはGemfileの編集とbundle installですね。

```
# graphql
gem 'graphql', '1.9.4'
gem 'graphiql-rails'
```

したらファイル生成

```
$ rails g graphql:install
```

すると、色々できます。

```
create  app/graphql/types
create  app/graphql/types/.keep
create  app/graphql/todo_schema.rb
create  app/graphql/types/base_object.rb
create  app/graphql/types/base_enum.rb
create  app/graphql/types/base_input_object.rb
create  app/graphql/types/base_interface.rb
create  app/graphql/types/base_scalar.rb
create  app/graphql/types/base_union.rb
create  app/graphql/types/query_type.rb
add_root_type  query
create  app/graphql/mutations
create  app/graphql/mutations/.keep
create  app/graphql/types/mutation_type.rb
add_root_type  mutation
create  app/controllers/graphql_controller.rb
route  post "/graphql", to: "graphql#execute"
gemfile  graphiql-rails
route  graphiql-rails
```

ルーティングは自動で前回と同じ内容になりますが、types以下の構成がだいぶ変わりましたね。

一旦graphiqlから動作確認してみましょう。

![graphiql_confirm](/images/blog/2019-04-15-graphql-latest-apollo/graphiql_confirm.png)

バッチリ。

続いて、Taskの型ファイルも生成しましょう。

```
rails g graphql:object Task id:ID! title:String! status:Int! priority:Int!
```

できたのはこちら。

```
module Types
  class TaskType < Types::BaseObject
    field :id, ID, null: false
    field :title, String, null: false
    field :status, Integer, null: false
    field :priority, Integer, null: false
  end
end
```

Rubyになりましたね。次は最初のタスクを取得するのとIDで検索するQueryを定義してみます。

```
module Types
  class QueryType < Types::BaseObject
    field :first_task, TaskType, null: false, description: "return first task"
    field :task_find_by, TaskType, null: false, description: "return a task" do
      argument :task_id, Integer, required: true
    end

    def first_task
      Task.first
    end

    def task_find_by(task_id:)
      Task.find_by(id: task_id)
    end
  end
end
```

![graphiql_search](/images/blog/2019-04-15-graphql-latest-apollo/graphiql_search.png)

書き方もぼちぼち変わりましたね。個人的にはこっちの方が親しみを感じます。

field名はキャメルケースでもスネークケースでもいいみたいです。Query投げる時はキャメルケースじゃないとだめみたいでした。

以前はfield内にresolverを書いていましたが、今回はデフォルトでは同名のメソッドがresolverとして動作するみたいです。resolver_method:というオプションをフィールドに渡すとメソッドの指定ができます。こんな感じですね。

```
field :resolver_method_exam, String, null: false, description: "this is resolver sample", resolver_method: :say_hello

def say_hello
  "Hello, resolver!"
end
```

というわけで前回やったとこまで追いついたので、これを楽に扱うライブラリ、Apolloを導入します。

## 導入しないとどうなるか

適当にボタンの設置とcoffeeでajax。

```
button#first-task-button first_task
```

```
$ ->
  $('#first-task-button').on 'click', ->
    query = {query: '{firstTask {id title status priority}}'};
    $.ajax
      url: '/graphql',
      type: 'POST',
      dataType: 'json',
      data: query
    .done (result) ->
      console.log(result)
    .fail (error) ->
      console.log(error)
```

なんか嫌ですね。これを無限に定義しないといけないとなると、悲しい未来が見えます。

なので、クライアントサイドから楽にGraphQLを扱うためのライブラリ、Apolloを使ってみます。あと色々と楽なのでVueも入れます。

## VueとApollo導入する

webpacker入れてvue導入

```
gem 'webpacker'
```

```
rails webpacker:install
```

```
rails webpacker:install:vue
```

/app/javascript/packs/vue_apollo_sample.jsを作成

```
import Vue from 'vue/dist/vue.esm.js'

document.addEventListener('DOMContentLoaded', () => {
  const app = new Vue({
    el: '#mount_target',
    data: {
      message: 'hello, vue'
    }
  })
})

```

ついでにGraphQLで色々試すためのページを作りましょうか。

routes.rb
```
resources :tasks do
  collection do
    get :graph, to: 'tasks#graph_index'
  end
end
```

tasks_controller.rb
```
  # 追加
  def graph_index
  end
```

graph_index.html.slim
```
= javascript_pack_tag 'vue_apollo_sample'

#mount_target
  p = "{{message}}"
```

これで、今作ったページでVueが動作するようになっているはずです。/tasks/graphにhello, vueが表示されていればOK。

したらApolloの導入ですね。vue向けのライブラリも一緒に入れましょう。

```
yarn add graphql graphql-tag apollo-client apollo-boost vue-apollo
```

それぞれどんなものかざっくり。

- garphql: graphqlをjsで扱うためのもの
- graphql-tag: クエリを楽に投げるための便利ツール
- apollo-client: apollo本体
- apollo-boost: apollo-clientを簡単に扱うためのラッパー
- vue-apollo: vue向け統合ライブラリ

ではいざ！

## Apollo実装編

基本的にapollo-boostに乗っかっていきます。

Vue×Apolloなところ...vue_apollo_sample.js
```
import Vue from 'vue/dist/vue.esm.js'
import ApolloClient from "apollo-boost";
import VueApollo from "vue-apollo";
import { gql } from "apollo-boost";

const client = new ApolloClient({
  uri: "http://localhost:3008/graphql",
  request: async operation => {
    operation.setContext({
      headers: {
        'X-CSRF-Token': document.querySelector('meta[name=csrf-token]').getAttribute('content'),
      },
    });
  }
});

const apolloProvider = new VueApollo({
  defaultClient: client
});

Vue.use(VueApollo);

document.addEventListener('DOMContentLoaded', () => {
  // graphql-tagが活躍しているところ
  const ALL_TASK_QUERY = gql`
  query allTask{
    allTask {
      id
      title
      status
      priority
    }
  }`

  // 引数ありなクエリ
  const SEARCH_TASK_QUERY = gql`
  query taskSearchBy($taskName: String!){
    taskSearchBy(taskName: $taskName) {
      id
      title
      status
      priority
    }
  }`

  const app = new Vue({
    el: '#mount_target',
    // これを渡すことで、this.$apolloからクエリを投げることができるようになる
    apolloProvider: apolloProvider,
    mounted() {
      self = this;
      this.$apollo.query({
        query: ALL_TASK_QUERY
      })
        .then(function (result) {
          self.tasks = result.data.allTask;
        })
        .catch(function (error) {
          console.log(error);
        });
    },
    data: {
      tasks: [],
    },
    methods: {
      search: function (e) {
        this.$apollo.query({
          query: SEARCH_TASK_QUERY,
          variables: {
            taskName: e.target.value
          }
        })
          .then(function (result) {
            self.tasks = result.data.taskSearchBy;
          })
          .catch(function (error) {
            console.log(error);
          });
      }
    }
  })
})
```

クエリ検索の仕様を、idからtitleの部分一致に変えました。

query_type.js
```
module Types
  class QueryType < Types::BaseObject
    field :all_task, [TaskType], null: false, description: "return all task"
    field :first_task, TaskType, null: false, description: "return first task"
    field :task_search_by, [TaskType], null: false, description: "return a task" do
      argument :task_name, String, required: true
    end
    field :resolver_method_exam, String, null: false, description: "this is resolver sample", resolver_method: :say_hello

    def all_task
      Task.all
    end

    def first_task
      Task.first
    end

    def task_search_by(task_name:)
      Task.where('title like ?', "%#{task_name}%")
    end

    def say_hello
      "Hello, resolver!"
    end
  end
end
```

viewはdataのtasksを全件表示したり検索input置いたりしてます。入力されるたびに検索走ります。結果はApolloがキャッシュしてくれます。

graph_index.html.slim
```
= javascript_pack_tag 'vue_apollo_sample'

#mount_target
  b search:
  input[type="text" @input="search"]
  div[v-for="task in tasks" :key="task.id"]
    = "{{task.title}}"
```

すると、こうなる

![graph_all](/images/blog/2019-04-15-graphql-latest-apollo/graph_all.png)
![graph_search](/images/blog/2019-04-15-graphql-latest-apollo/graph_search.png)

う、動いたーーー！ Vueを導入したのもありますが、jQueryで頑張るパターンと比較してなにかつらみから解放された感がありますね。

今回はサンプルということもあり1ファイルのjsで頑張りましたが、クエリ自体は別ファイルで管理するとか、Apolloのproviderは親コンポーネント/処理は子コンポーネントという構成にしたりとか、もっと人道的な工夫はできます。あと、apollo-boostはカスタマイズには向かないので、業務で使うならapollo-clientでしっかり設定していくべきとの言説を見かけました。

とかとかありますがひとまず動いた！達成感！
