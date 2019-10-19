---
layout: blog
title: Vue.js + Vue Test Util + Jestでコンポーネントの単体テストを書きたい
category: blog
tags: [Vue.js, jest, ユニットテスト]
summary: Vueのテストまだ書いたことなかったので入門します。Vue.jsで使わているテストフレームワークをざっとぐぐると、mochaかjestって印象、どちらかというとjestの方がモダンな雰囲気だったので、jestにします。
author: aharada
image: /images/blog/2019-10-19-vue-unit-test/component.png
---

個人的な趣味で作っているサービスをVue.js + Firebaseという構成で書かれている。

個人サービス程度にテストはいらんだろって意見もあるけども、毎日書く時間が取れない個人サービスの場合、手を入れようとしたときに自分のコードがどうなっているか覚えていないので、むしろテストがある方が非常に開発がしやすい。

Vueのテストまだ書いたことなかったので入門します。Vue.jsで使わているテストフレームワークをざっとぐぐると、mochaかjestって印象、どちらかというとjestの方がモダンな雰囲気だったので、jestにします。

## 事前準備

まずvue-cliはアップデートが早かった気がするので最新版にしておく。

```
$ vue --version
3.10.0
```

アップデートする。

```
$ npm install -g @vue/cli
```

もう4系なのかよオイ。

```
$ vue --version
@vue/cli 4.0.4
```

公式ドキュメントみながらvue-cliを使って新しいプロジェクトを生成する。

[Creating a Project Vue CLI](https://cli.vuejs.org/guide/creating-a-project.html#vue-create)

```
$ vue create vue-unit-test-sample
```

どうやらこの時点でテストフレームワークを選択出来るっぽいがスルーしてしまった。

ちなみにパッケージマネージャはyarnではなくnpmを指定。今どきはyarnじゃなくてnpmに戻ってきているらしいよ。

サーバ起動してみる。

```
$ cd vue-unit-test-sample
$ npm run serve
```

http://localhost:8080/ を開く。OKだ。

![Vue](/images/blog/2019-10-19-vue-unit-test/vue-new.png)

## テスト対象の準備

公式ドキュメントに載っているHelloコンポーネントをコピペする。これはusernameに7文字未満の文字列が入力されているとエラーが表示される、という単純なコンポーネント。

[Vue コンポーネントの単体テスト — Vue.js](https://jp.vuejs.org/v2/cookbook/unit-testing-vue-components.html)

Hello.vue

```
<template>
  <div>
    <input v-model="username">
    <div 
      v-if="error"
      class="error"
    >
      {{ error }}
    </div>
  </div>
</template>

<script>
export default {
  name: 'Hello',
  data () {
    return {
      username: ''
    }
  },

  computed: {
    error () {
      return this.username.trim().length < 7
        ? 'Please enter a longer username'
        : ''
    }
  }
}
</script>
```

App.vue

```
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <Hello />
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import Hello from './components/Hello.vue'

export default {
  name: 'app',
  components: {
    Hello
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

画面はこんな感じ

![コンポーネント](/images/blog/2019-10-19-vue-unit-test/component.png)

一旦jestを使っていないテストコード。

__tests__/Hello.test.js

```
import { shallowMount } from '@vue/test-utils'
import Hello from './Hello.vue'

test('Hello', () => {
  // コンポーネントを描画します
  const wrapper = shallowMount(Hello)

  // `username`は空白を除外して7文字未満は許されません
  wrapper.setData({ username: ' '.repeat(7) })

  // エラーが描画されることをアサートします
  expect(wrapper.find('.error').exists()).toBe(true)

  // 名前を十分な長さにします
  wrapper.setData({ username: 'Lachlan' })

  // エラーがなくなったことをアサートします
  expect(wrapper.find('.error').exists()).toBe(false)
})
```

## Jestでテストを書く

公式ドキュメントに従ってやっていきます。

[Jest を使用した単一ファイルコンポーネントのテスト Vue Test Utils](https://vue-test-utils.vuejs.org/ja/guides/testing-single-file-components-with-jest.html)

```
$ npm install --save-dev jest @vue/test-utils vue-jest babel-jest
```

`package.json`に追記

```
// package.json
{
  "scripts": {
    "test": "jest"
  }

  // ...
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "vue"
    ],
    "transform": {
      ".*\\.(vue)$": "vue-jest"
      "^.+\\.js$": "<rootDir>/node_modules/babel-jest"
    }
  }
}
```

テスト実行してみるも上手く行かない。

```
$ npm run test

...

FAIL  src/components/__tests__/Hello.test.js
  ● Test suite failed to run

    Cannot find module 'babel-core'

      at Object.<anonymous> (node_modules/vue-jest/lib/compilers/babel-compiler.js:1:15)
```

`.babelrc`の設定飛ばしていたので一応やる。

.babelrc

```
{
  "presets": [["env", { "modules": false }]],
  "env": {
    "test": {
      "presets": [["env", { "targets": { "node": "current" } }]]
    }
  }
}
```

```
$ npm install --save-dev @babel/core @babel/preset-env
```

テスト実行してみたけど謎のエラー。

```
$ npm run test

...

   Cannot find module 'babel-preset-env' from '/Users/atsushiharada/source/vue-unit-test-sample'
    - Did you mean "@babel/env"?
```

`.babelrc`を書き換えてみる。

```
{
    "presets": ["@babel/preset-env"]
}
```

再度テスト実行。エラー内容がもとに戻った。

```
$ npm run test

...


  ● Test suite failed to run

    Cannot find module 'babel-core'

      at Object.<anonymous> (node_modules/vue-jest/lib/compilers/babel-compiler.js:1:15)
```

何やらbridgeなるものが必要っぽい。とりあえずテスト動かしたいので詳細は追わなかった。

[nuxtでjestするとbabel-coreを見つけてくれない - Qiita](https://qiita.com/kawadumax/items/48568f8bdff7a65e3f46)

```
$ npm install --save-dev babel-core@bridge
```

テスト実行。

ｷﾀ━━━━(ﾟ∀ﾟ)━━━━!!

```
~/s/vue-unit-test-sample ❯❯❯ npm run test                                     master ✱ ◼

> vue-unit-test-sample@0.1.0 test /Users/atsushiharada/source/vue-unit-test-sample
> jest

 PASS  src/components/__tests__/Hello.test.js
  Hello
    ✓ run test !!!!!!!!! (3ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        3.852s
Ran all test suites.
```

次はテストをjestの形式に書き直します。jestでは、テスト対象がおいてあるディレクトリに`__tests__`というディレクトリを作って、その下にテストコード入れるっぽい？ちょっとキモいけど郷に従います。

> Jest は、テスト対象のコードのすぐ隣に__tests__ディレクトリを作成することを推奨していますが、適切にテストを構造化することは自由です。

参考

[Vue meetupでテスト書いている人が少なかったのでオレオレテストを晒してみる Part. 1 - Qiita](https://qiita.com/ykhirao/items/adf22bfc068cea933eed)

```
$ tree src
src
├── App.vue
├── assets
│   └── logo.png
├── components
│   ├── Hello.vue
│   ├── HelloWorld.vue
│   └── __tests__
│       └── Hello.test.js
└── main.js
```

`__tests__/Hello.test.js`をjestで書き換える。

```
describe('Hello', () => {
  it('半角スペース7文字入力の場合、エラーになること', () => {
      const wrapper = shallowMount(Hello)
      wrapper.setData({ username: ' '.repeat(7) })
      expect(wrapper.find('.error').exists()).toBe(true)
  })

  it('7文字以上の文字列を入力した場合、エラーにならないことこと', () => {
    const wrapper = shallowMount(Hello)
    wrapper.setData({ username: '1234567' })
    expect(wrapper.find('.error').exists()).toBe(false)
  })
})
```

通ったー！

```
$ npm run test

> vue-unit-test-sample@0.1.0 test /Users/atsushiharada/source/vue-unit-test-sample
> jest

 PASS  src/components/__tests__/Hello.test.js
  Hello
    ✓ 半角スペース7文字入力の場合、エラーになること (23ms)
    ✓ 7文字以上の文字列を入力した場合、エラーにならないことこと (3ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        1.428s
```

ちょっと手数多かったけどそれほど難しくはない。次に気になるのは、外部APIコールするメソッドなどをコンポーネントに持たせてたりするけど、そのへんどうやってテストするといいんだろう。

そもそも、外部APIコールする部分は別のレイヤーに切り出してモック出来るようにしておくのが良さそうだけど。