---
layout: blog
title: Nuxt.js + Vuetify でサクッと管理画面をつくってみた
category: blog
tags: [Vue.js, Nuxt.js, Vuetify]
summary: Vue.jsのCSSフレームワークVuetifyには便利コンポーネントがたくさんある！との噂を聞き、億劫になりがちな管理画面をサクッと作ってみました。
author: terapasta
image: /images/blog/2019-10-29-nuxt-vuetify/nuxt-vuetify.png
---

Railsエンジニアの方であれば、簡単な管理画面を作る際にrails_adminやactive_adminにお世話になっている、いやなりすぎている方も多いのではないでしょうか？私のように、、、

Railsを使っていないSPAの管理画面を作ることになった時、Nuxt.js + Vuetifyという選択をしたら幸せになった気がしたのでちょろっと共有します。

ちなみに今回の記事の範囲でNuxt.jsを使うメリットはあまり無いのですが、管理画面に必須の認証機能の実装やaxiosを使った通信等も考慮すると、Nuxt.jsを選んでおくと楽ができると思います。

vuetifyのドキュメントは[こちら](https://vuetifyjs.com/ja/components/api-explorer)

### この記事でやること
- Vuetifyのコンポーネントを使った簡単なCRUD処理

### この記事でやらないこと
- 認証周り
- API経由のデータ操作
- イケイケダッシュボードづくり

## nuxtプロジェクト作成
何はともあれ
```
$ npx create-nuxt-app admin-sample
```
その後はEnterを連打しつつも「UI framework」だけはVuetifyをちゃっかり選択。

![create-nuxt-app](/images/blog/2019-10-29-nuxt-vuetify/create-nuxt-app.png)

促されるままに次のコマンドを実行すると、ちょっとリッチな画面が登場。
```
$ cd admin-sample
$ npm run dev
```
![first-view](/images/blog/2019-10-29-nuxt-vuetify/first-view.png)

今回はこちらのデフォルトの画面をベースに作っていきます！

## 下準備
本記事では実際にAPIをからデータの取得は行わないのでダミーのユーザーデータを作っておきます。
store/index.js
```
export const state = () => ({
  users: [
    {
      id: 1,
      name: 'sato',
      email: 'sato@sample.com',
      memo: 'セパタクロ,トラックバック,相撲,山手線一周,占い',
    },
    {
      id: 2,
      name: 'suzuki',
      email: 'suzuki@sample.com',
      memo: '華道,競馬,折り紙,鉄道旅行,ランニング',
    },
    {
      id: 3,
      name: 'takahashi',
      email: 'takahashi@sample.com',
      memo: '落語,携帯電話基地局巡り,公営競技,語学,スキップ',
    },
    {
      id: 4,
      name: 'tanaka',
      email: 'tanaka@sample.com',
      memo: 'プラモデル,カメラ,カポエイラ,ジャニーズ追っかけ,切手収集',
    },
    {
      id: 5,
      name: 'watanabe',
      email: 'watanabe@sample.com',
      memo: 'タイル施工,公営競技,BCL,油絵,サックス',
    },
    {
      id: 6,
      name: 'ito',
      email: 'ito@sample.com',
      memo: 'お笑い鑑賞,油絵,骨董品収集,パズル,ロッククライミング',
    },
    {
      id: 7,
      name: 'yamamoto',
      email: 'yamamoto@sample.com',
      memo: 'モデル撮影,書道,ギター,競馬,競輪',
    },
    {
      id: 8,
      name: 'nakamura',
      email: 'nakamura@sample.com',
      memo: 'パスタ作り,韓流ドラマ,野鳥観察,工芸,動物園巡り',
    },
    {
      id: 9,
      name: 'kobayashi',
      email: 'kobayashi@sample.com',
      memo: 'グルメ,ソムリエ,pixiv,音楽鑑賞,太極拳',
    },
    {
      id: 10,
      name: 'kato',
      email: 'kato@sample.com',
      memo: 'プログラミング,ラップ,食べ歩き,将棋,K1',
    },
  ]
})

export const getters = {
  getUsers (state) {
    return state.skets
  },
}

export const mutations = {
  addUser (state, paylaod) {
    state.users.push(paylaod.user)
  },
  updateUser (state, paylaod) {
    state.users.forEach((user, index) => {
      if (user.id === paylaod.user.id) {
        state.users.splice(index, 1, paylaod.user)
      }
    })
  },
  removeUser (state, paylaod) {
    state.users.forEach((user, index) => {
      if (user.id === paylaod.user.id) {
        state.users.splice(index, 1)
      }
    })
  }
}


```

## ナビゲーションバーに項目を追加
まずはUser管理っぽいものを追加してあげます。アイコンは[Material Design Icon](https://materialdesignicons.com/)から選び放題なので、それっぽいものを選びます。

layouts/default.vue
```
・
・
<script>
export default {
  data () {
    return {
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-apps',
          title: 'Welcome',
          to: '/'
        },
        {
          icon: 'mdi-chart-bubble',
          title: 'Inspire',
          to: '/inspire'
        },
        {
          icon: 'mdi-account',
          title: 'Users',
          to: '/users'
        }
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Vuetify.js'
    }
  }
}
</script>
```

## 一覧テーブルを作成
ここからVuetifyが火を噴きます。
`v-data-table` を使って上げるといとも簡単にそれっぽいものができてしまいます。
基本的には `headers` にtextとvalueをセットしたオブジェクトを渡して、itemsに一覧表示をしたいリスト渡す。それだけ！
しかも、デフォルトで各項目のソート機能とページネーション機能付き。最高。

今回は+αで下記のオプションも付けています
- items-per-page: 1ページ辺りに表示するitemの数。
- search: dataに渡すmodelを定義してあげると、それだけで検索ができちゃう。

users.vue
```
<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-card v-if="users">
      <v-card-title>
        ユーザー一覧
        <v-spacer />
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="検索"
          sigle-line
        />
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="users"
        :items-per-page="5"
        :search="search"
        sort-by="id"
        :sort-desc="true"
        class="elevation-1"
      >
      </v-data-table>
    </v-card>
  </v-layout>
</template>

<script>
export default {
  data () {
    return {
      search: '',
      headers: [
        { text: 'ID', value: 'id' },
        { text: 'メールアドレス', value: 'email' },
        { text: '名前', value: 'name' },
        { text: 'メモ', value: 'memo' },
      ],
    }
  },
  computed: {
    users () {
      return this.$store.getters['getUsers']
    }
  },
}
</script>

```

## 編集・削除機能を追加
まずは一覧に編集ボタンを追加します。
users.vue
```
<v-data-table>
  ・
  ・
  <template v-slot:item.actions="{ item }">
    <v-icon
      small
      @click="edit(item)"
    >
      mdi-pencil
    </v-icon>
    <v-icon
      small
      @click="remove(item)"
    >
      mdi-delete
    </v-icon>
  </template>
</v-data-table>
・
・
  headers: [
    ・
    ・
    { text: '操作', value: 'actions' }
  ],
```

次に編集モーダルを実装します。実装というと大げさなくらい `v-dialog` というコンポーネントを使うと一瞬です。 
編集・削除機能を実装した後のコードがこちら。

users.vue
```
<template>
  <v-layout
    column
    justify-center
    align-center
  >
    <v-card v-if="users">
      <v-card-title>
        ユーザー一覧
        <v-spacer />
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="検索"
          sigle-line
        />
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="users"
        :items-per-page="5"
        :search="search"
        class="elevation-1"
      >
        <template v-slot:top>
          <v-dialog v-model="dialog" max-width="500px">
            <v-card>
              <v-card-title>
                <span class="headline">ユーザー編集</span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="6">
                      <v-text-field v-model="user.email" label="メールアドレス" />
                    </v-col>
                    <v-col cols="6">
                      <v-text-field v-model="user.name" label="名前" />
                    </v-col>
                    <v-col cols="12">
                      <v-text-field v-model="user.memo" label="メモ" />
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer />
                <v-btn @click="close">閉じる</v-btn>
                <v-btn class="primary" @click="update">更新する</v-btn>
                <v-spacer />
              </v-card-actions>
            </v-card>
          </v-dialog>
        </template>
        <template v-slot:item.actions="{ item }">
          <v-icon
            small
            @click="edit(item)"
          >
            mdi-pencil
          </v-icon>
          <v-icon
            small
            @click="remove(item)"
          >
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </v-card>
  </v-layout>
</template>

<script>
export default {
  data () {
    return {
      dialog: false,
      search: '',
      headers: [
        { text: 'ID', value: 'id' },
        { text: 'メールアドレス', value: 'email' },
        { text: '名前', value: 'name' },
        { text: 'メモ', value: 'memo' },
        { text: '操作', value: 'actions' }
      ],
      user: {},
    }
  },
  computed: {
    users () {
      return this.$store.getters['getUsers']
    }
  },
  methods: {
    edit (user) {
      this.user = Object.assign({}, user)
      this.dialog = true
    },
    update () {
      const payload = { user: this.user }
      this.$store.commit('updateUser', payload)
      this.close()
    },
    remove (user) {
      const payload = { user: user }
      this.$store.commit('removeUser', payload)
    },
    close () {
      this.dialog = false
      this.user = {}
    },
  }
}
</script>


```

![users-list](/images/blog/2019-10-29-nuxt-vuetify/users-list.png)

![modal](/images/blog/2019-10-29-nuxt-vuetify/modal.png)

## 新規登録機能の追加
最後に新規登録をつけて完成です！
まずは `methods` の中にadd, createを追加します。

users.vue
```
add (user) {
  this.user = {}
  this.dialog = true
},
create () {
  const payload = { user: this.user }
  this.$store.commit('addUser', payload)
  this.close()
},
```

新規登録でもモーダルを使いたいので、編集機能のものを使い回せるように新規登録/編集を判別する `computed` を追加してモーダルを修正します。

users.vue
```
isPersistedUser () {
  return !!this.user.id
},
formTitle () {
  return this.isPersistedUser ? 'ユーザー編集' : 'ユーザー追加'
}
```

users.vue
```
<v-card-title>
  <span class="headline">{{ formTitle }}</span>
</v-card-title>
・
・
<v-card-actions>
  <v-spacer />
  <v-btn @click="close">閉じる</v-btn>
  <v-btn v-if="isPersistedUser" class="primary" @click="update">更新する</v-btn>
  <v-btn v-else class="primary" @click="create">追加する</v-btn>
  <v-spacer />
</v-card-actions>
```

あとはよくある丸いプラスのボタンでも置いておきましょう。

users.vue
```
<template v-slot:activator="{ on }">
  <v-btn fab dark small color="dark" class="mb-2"
    @click="add"
  >
    <v-icon dark>
      mdi-plus
    </v-icon>
  </v-btn>
</template>
```
![add-modal](/images/blog/2019-10-29-nuxt-vuetify/add-modal.png)

## まとめ
いかがでだったしょうか？
Vuetifyを使うことで、一切CSSも難しい実装もなくサクッといい感じの管理画面のCRUD処理が作れました。Vuetifyには他にも多彩なコンポーネントが用意されていてやりたいと思うことはだいたいあるような感覚です。

Railsのgemで管理画面を作っていると、後々カスタマイズを要求されて死に至ることがありますが、Nuxt.js + Vuetifyで作っていればその心配もなく、楽しくカスタマイズできそうです。
