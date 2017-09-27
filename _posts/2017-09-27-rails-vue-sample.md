---
layout: blog
title: Rails + Vue.js + Ajaxでボタンを押したらAPIにGETリクエストしてデータを画面に表示させる
category: blog
tags: [Vue.js, vue, Rails, Ruby on Rails]
summary: 今回はRails作ったサーバサイドにAjaxでリクエストして、取得した値をViewに反映する、というような実践に寄せたサンプルをやってみる。
author: aharada
---

前回はひとまずRails上でVue.jsを動かすところまでやったので、今回はRails作ったサーバサイドにAjaxでリクエストして、取得した値をViewに反映する、というような実践に寄せたサンプルをやってみる。

前回の記事

[http://tech.mof-mof.co.jp/blog/rails-vue.html](http://tech.mof-mof.co.jp/blog/rails-vue.html)

こちらを参考にした

- [http://qiita.com/cohki0305/items/a678b0b17c5b496c1de9](http://qiita.com/cohki0305/items/a678b0b17c5b496c1de9)
- [http://qiita.com/naoki85/items/51a8b0f2cbf949d08b11](http://qiita.com/naoki85/items/51a8b0f2cbf949d08b11)

## サーバサイドのデータを準備する

employees(社員テーブルみたいな)のを適当に作る。フィールドはなんでもいい。

```
$ rails g model Employee name:string age:integer email:string
$ rails db:migrate
```

seed.rb

```
Employee.create(name: '竹野内豊', age: 42, email: 'y.takenouchi@example.com')
Employee.create(name: '反町隆史', age: 43, email: 't.sorimachi@example.com')
Employee.create(name: '豊川悦司', age: 55, email: 'e.toyokawa@example.com')
```

seedを実行してテストデータを投入する

```
$ rails db:seed
```

## 実装

employeesのデータにアクセスするためのcontrollerを実装

```
$ rails g controller employees
```

employees_controller.rb

indexで普通に全データを取得して、Ajaxでshowアクションを叩いてVue.jsで描画する設計。

```
class EmployeesController < ApplicationController
 def index
   @employees = Employee.all
 end

 def show
   @employee = Employee.find(params[:id])
   render json: @employee
 end
end
```

routes.rb

```
Rails.application.routes.draw do
  resources :employees, only: [:index, :show]
end

```

employees/index.html.erb

```
<div id="employees">
 <% @employees.each do |employee| %>
   <button v-on:click="setEmployeeInfo(<%= employee.id %>)"><%= employee.name %></button>
 <% end %>
 <div>
   <p>name: {{ employeeInfo.name }}</p>
   <p>age: {{ employeeInfo.age }}</p>
   <p>email: {{ employeeInfo.email }}</p>
 </div>
</div>

<%= javascript_pack_tag 'employees/index' %>
```

app/javascript/packs/employees/index.js

```
import Vue from 'vue/dist/vue.esm.js'
import App from '../app.vue'
import axios from 'axios';

new Vue({
  el: '#employees',
  data: {
    employeeInfo: {},
  },
  methods: {
    setEmployeeInfo(id){
      axios.get(`employees/${id}`)
        .then(res => {
          console.log(res.data)
          this.employeeInfo = res.data;
        });
    }
  }
});
```

ajaxを簡単に実装するためのライブラリをインストールしておく

```
$ yarn add axios
```

まずはブラウザからshowアクションを叩いてみて、jsonが返ってくることを確認しておく

http://localhost:3000/employees/1

```
{"id":1,"name":"竹野内豊","age":42,"email":"y.takenouchi@example.com","created_at":"2017-09-27T10:40:19.920Z","updated_at":"2017-09-27T10:40:19.920Z"}
```

よしOK。

そしたらボタンを押してみて、employeesから取得したデータが画面に反映されることを確認。

できたああああ！

![](/images/blog/2017-09-27-rails-vue-sample/rails-vue.gif)

## ハマった点

実装している最中にちょっとハマったのですが、jsファイルに以下をちゃんと書かないとタグが全部消えたりしました。あんまり意味は分かっていませんが書いておかねばならないらしい。

```
import Vue from 'vue/dist/vue.esm.js'
```

参考

[http://qiita.com/naoki85/items/51a8b0f2cbf949d08b11](http://qiita.com/naoki85/items/51a8b0f2cbf949d08b11)
