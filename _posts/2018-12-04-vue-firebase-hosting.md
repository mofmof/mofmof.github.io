---
layout: blog
title: Vue.jsで作ったサイトをFirebaseでホスティングする
category: blog
tags: [vue.js, firestore, firebase, hosting]
summary: 前回、Vue.js + Firestoreを使用してラーメン一覧を表示するところまで作った。今度はこれをFirebaseでホスティングしてみる。
author: aharada
image: /images/blog/2018-12-04-vue-firebase-hosting/default.png
---

前回、Vue.js + Firestoreを使用してラーメン一覧を表示するところまで作った。今度はこれをFirebaseでホスティングしてみる。

[Vue.js + Firestoreでデータを取得してラーメン一覧を表示する](/blog/vue-firestore-ramens.html)

いつもはホスティングはnetlifyを使っているんだけど、Firebaseにもあるらしいのでせっかくだから試してみようかと。

プロジェクトのルートへ移動して、ホスティング用のファイルを生成する。

Vue.jsはビルドすると`dist`ディレクトリにファイルを吐くので、公開ディレクトリは`dist`と指定すること。

```
$ cd todo-vue-cli3
$ firebase init hosting

=== Project Setup

First, let's associate this project directory with a Firebase project.
You can create multiple project aliases by running firebase use --add,
but for now we'll just set up a default project.

? Select a default Firebase project for this directory: hogehoge-project

=== Hosting Setup

Your public directory is the folder (relative to your project directory) that
will contain Hosting assets to be uploaded with firebase deploy. If you
have a build process for your assets, use your build's output directory.

? What do you want to use as your public directory? dist
? Configure as a single-page app (rewrite all urls to /index.html)? No
✔  Wrote dist/404.html
✔  Wrote dist/index.html

i  Writing configuration info to firebase.json...
i  Writing project information to .firebaserc...

✔  Firebase initialization complete!
```

デプロイしてみたら怒られた。

```
$ firebase deploy
Error: HTTP Error: 410, This version of the Firebase CLI is no longer able to deploy to Firebase Hosting. Please upgrade to a newer version (>= 4.1.0). If you have further questions, please reach out to Firebase support.
```

うーん。`npm`で`firebase-tools`をアップグレードしても解消せず、どうやっても`firebase-tools`のバージョンが3系のまま。。

`/usr/local/bin/lib/node_modules`の中に`firebase-tools`がインストールされているみたい。グローバル領域に入ってるということだろう。

コマンドでアンインストールしても消えなかったので、`mv`してしまった。一応これで`yarn add`した`firebase-tools`が参照されるようになったぽい。

```
$  ls /usr/local/lib/node_modules
@vue           firebase-tools n              pure-prompt    serverless     yarn
$ mv /usr/local/lib/node_modules/firebase-tools ~
$ firebase --version
6.1.2
$ todo-vue-cli3 git:(master) ✗ which firebase
/Users/harada/.nodebrew/current/bin/firebase
```

気を取り直してデプロイする。

```
$ firebase deploy

=== Deploying to 'hogehoge-project'...

i  deploying hosting
i  hosting[hogeoge-project]: beginning deploy...
i  hosting[hogeoge-project]: found 2 files in dist
✔  hosting[hogeoge-project]: file upload complete
i  hosting[hogeoge-project]: finalizing version...
✔  hosting[hogeoge-project]: version finalized
i  hosting[hogeoge-project]: releasing new version...
✔  hosting[hogeoge-project]: release complete

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/hogeoge-project/overview
Hosting URL: https://hogeoge-project.firebaseapp.com
```

とりあえずデプロイは成功した模様だが、なんらかのデフォルトの画面がでてるっぽい。

![デフォルトの画面](/images/blog/2018-12-04-vue-firebase-hosting/default.png)

ソースコードを確認すると、`dist`には何も吐かれていない。おそらくVue側でビルドする必要がありそうだ。

```
$ yarn build
```

```
$ firebase deploy
=== Deploying to 'hogehoge-project'...

i  deploying hosting
i  hosting[hogehoge-project]: beginning deploy...
i  hosting[hogehoge-project]: found 7 files in dist
✔  hosting[hogehoge-project]: file upload complete
i  hosting[hogehoge-project]: finalizing version...
✔  hosting[hogehoge-project]: version finalized
i  hosting[hogehoge-project]: releasing new version...
✔  hosting[hogehoge-project]: release complete

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/hogehoge-project/overview
Hosting URL: https://hogehoge-project.firebaseapp.com
```

できたー！！

![ホスティング成功画面](/images/blog/2018-12-04-vue-firebase-hosting/ramens.png)

Netlifyは神lifyだとは思うが、cliでちょちょいと適当にコマンドでホスティング出来るのもなかなかイケてる。Firebase側にまとめたいときには十分使えそう。

GitHub
[GitHub - harada4atsushi/todo-vue-cli3-](https://github.com/harada4atsushi/todo-vue-cli3-)
