---
layout: blog
title: Expo + React Native + Firestoreでデータを取得してみる
category: blog
tags: [React Native, Expo, Firebase, Firestore]
summary: 前回React Native + Expoに入門してみて、これはだいぶイイのではと思ったので、もう少し踏み込んで試してみます。今回はFirebaseのFirestoreに接続してデータを引っ張ってくるところまでやります。
author: aharada
image: 
---

前回React Native + Expoに入門してみて、これはだいぶイイのではと思ったので、もう少し踏み込んで試してみます。今回はFirebaseのFirestoreに接続してデータを引っ張ってくるところまでやります。

[Expo + React Nativeに入門して、iOSネイティブのCollectionViewっぽいものを作る](/blog/get-started-react-native-with-expo.html)

このあたりの記事を参考にした。公式はあんまり情報が充実していない感。

- [https://qiita.com/ruxes7/items/e38d79dcc878249fa33c](https://qiita.com/ruxes7/items/e38d79dcc878249fa33c)
- [https://docs.expo.io/versions/latest/guides/using-firebase/](https://docs.expo.io/versions/latest/guides/using-firebase/)
- [https://docs.expo.io/versions/latest/guides/using-firebase/#firebase-sdk-setup](https://docs.expo.io/versions/latest/guides/using-firebase/#firebase-sdk-setup)

前回同様、新規にプロジェクトを作成します。プロジェクト名はお好きにどうぞ。テンプレートは何でも良いのですが、今回はblankを選択した。

 
```
$ expo init MenstagramRN
```

起動してみる。

```
$ cd MenstagramRN
$ expo start
```

画面から「Run on iOS simulator」を選択。ちゃんとアプリが起動することを確認する。

![アプリ起動](/images/blog/2020-02-29-get-started-expo-firestore/launch.png)

## Firebaseの導入

Firebaseのプロジェクトの作成は画面からポチポチやってれば出来るので省略しますね。

[https://firebase.google.com/?hl=ja](https://firebase.google.com/?hl=ja)

Firebase SDKをインストール。expoはデフォルではyarnを推しているが、個人的にはnpm使ってるのでnpmで行きます。

```
$ npm i --save firebase
```

firebase.js

```
import firebase from "firebase";
import 'firebase/firestore';
  
const firebaseConfig = {
  apiKey: "<apiKey>",
  authDomain: "<authDomain>",
  databaseURL: "<databaseURL>",
  projectId: "<projectId>",
  storageBucket: "<storageBucket>",
  messagingSenderId: "<messagingSenderId>",
  appId: "<appId>"
};

const firebaseApp = !firebase.apps.length
  ? firebase.initializeApp(firebaseConfig)
  : firebase.app()

const db = firebaseApp.firestore();

export default db
```

apiKeyなどの取得は以下記事を参考に。

[Vue.jsでFirebaseのFirestoreにアクセスしてデータを取得する](/blog/vue-firestore.html)

![ウェブアプリにFirebaseを追加](/images/blog/2020-02-29-get-started-expo-firestore/firebase.png)

上記コードでは修正済みだけど、こんなエラーにちょっとハマった。

```
App named '[DEFAULT]' already exists
```

どうやら`firebase.initializeApp()`が複数回呼ばれてしまう問題らしいので、以下のように修正している。

```
const firebaseApp = !firebase.apps.length
  ? firebase.initializeApp(firebaseConfig)
  : firebase.app()
```

App.js

Firestoreにramensというコレクションがある前提。

```
import React from 'react';
import { StyleSheet, Text, View } from 'react-native';
import db from './firebase'


export default function App() {
  db.collection("ramens").get().then((querySnapshot) => {
      querySnapshot.forEach((doc) => {
          console.log(`${doc.id} => ${doc.data()}`);
      });
  });

  return (
    <View style={styles.container}>
      <Text>Open up App.js to start working on your app!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

これで起動すると以下のWarningが出てデータが取得出来ない。この問題に1,2時間くらいハマってしまったぜ。

```
[Unhandled promise rejection: ReferenceError: Can't find variable: atob]
```

解決方法はStackOverflowに載ってた。

[https://stackoverflow.com/questions/60361519/cant-find-a-variable-atob](https://stackoverflow.com/questions/60361519/cant-find-a-variable-atob)

Firebase SDKの7.9.1以降で発生する問題なので、7.9.0にしてあげれば解決。

```
$ npm install firebase@7.9.0
```

で、大事なのはキャッシュをクリアしてexpoのサーバを起動すること。これやってなくてハマった。

```
$ expo r -c
```

よっしゃログにデータが出力されたぞ！(ramensにデータが入っている前提)

```
wXtaxkaLDdGD0uEAuYwC => [object Object]
x8ynrV7ax6d69igBQTZU => [object Object]
xiKbMDicwbuK6s5PwTtB => [object Object]
xkG5lYuYcE3mAJ8z9htT => [object Object]
y1Rr3GpVwAt3hbbHn3Ou => [object Object]
y506NXdGaHpAsYuPXGBS => [object Object]
yDw2Zrzq37iR9xDQDe2L => [object Object]
yIk0BSb9mHG8M4JxOCoA => [object Object]
yJVLqRMouUVi8mgTznyO => [object Object]
yVPoJm60tBoYJV68XPvt => [object Object]
yW81smGartshw7ZaCbD3 => [object Object]
yxJNPxKWjDXNmVFliuxm => [object Object]
zFlVfy6up3X7fwyftsyK => [object Object]
zKu3Di4cIS78iLqwTObt => [object Object]
zVQIFmiRa6HxwRmaMpOU => [object Object]
ziVr7DCF73IQUW5GuCr1 => [object Object]
zp94IyKmlvhFhPK4JnFU => [object Object]
zuoMnNc9wOnFlB1b7u3U => [object Object]
```

サクッと終わる予定だったのが思いの外ハマってしまったので、データを画面に表示するところはまた次回にでもやりまあす。

