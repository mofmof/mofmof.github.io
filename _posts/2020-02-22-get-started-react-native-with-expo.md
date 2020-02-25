---
layout: blog
title: Expo + React Nativeに入門して、iOSネイティブのCollectionViewっぽいものを作る
category: blog
tags: [React Native, Expo, CollectionView]
summary: 前回はExpoの良さを理解するために、あえてExpoなしでReactNativeに入門してみた。今回はいよいよExpoを体験してみることにします。
author: aharada
image: 
---

前回はExpoの良さを理解するために、あえてExpoなしでReactNativeに入門してみた。今回はいよいよExpoを体験してみることにします。

[React Nativeに入門して、iOSネイティブのTableViewっぽいものを作る](/blog/get-started-react-native.html)

React Natvieの公式チュートリアルと、Expoの公式チュートリアルの両方を参考にした。

[https://facebook.github.io/react-native/docs/getting-started](https://facebook.github.io/react-native/docs/getting-started)

[https://expo.io/learn](https://expo.io/learn)

## インストールから動くところまで

コマンドラインツールをインストール

```
$ npm install -g expo-cli
```

プロジェクトを生成しようとするもexpoコマンドないぞ、とのこと。

```
$ expo init ExpoProject
zsh: command not found: expo
```

React Native公式チュートリアルにNode 10以上を使えよと書いてある。

> Assuming that you have Node 10 LTS or greater installed, you can use npm to install the Expo CLI command line utility:

確認してみたら8系が入ってた。

```
$ node -v
v8.16.0
```

一旦コマンドラインツールをアンインストール。

```
$ npm uninstall -g expo-cli
```

nodenvでnodeのバージョン管理をしてたみたいなので、グローバルのバージョンを変更する。

```
$ nodenv versions                                                                                                                                                    ✘ 1
  system
* 8.16.0 (set by /Users/atsushiharada/.nodenv/version)
  12.4.0
```

```
$ nodenv global 12.4.0
$ nodenv versions
  system
  8.16.0
* 12.4.0 (set by /Users/atsushiharada/source/.node-version)
```

再びコマンドラインツールをインストール。

```
$ npm install -g expo-cli
```

プロジェクトを生成。あれれ？

```
$ expo init ExpoProject
zsh: command not found: expo
```

よく知らんがnpxつけてみたら出来た。

```
$ npx expo init ExpoProject
```

テンプレートを選択出来るので適当に`expo-template-bare-minimum`を選択した。

```
Your project is ready at /Users/atsushiharada/source/ExpoProject

Before running your app on iOS, make sure you have CocoaPods installed and initialize the project:

  cd ExpoProject/ios
  pod install

Then you can run the project:

  cd ExpoProject
  yarn android
  yarn ios
```

yarnで起動してみろと言われたけど普段npmしか使ってないので無視した。なんかエラー。

```
$ npm run ios

> @ ios /Users/atsushiharada/source/ExpoProject
> react-native run-ios

error Could not find "Podfile.lock" at /Users/atsushiharada/source/ExpoProject/ios/Podfile.lock. Did you run "pod install" in iOS directory?
error Could not find the following native modules: RNGestureHandler, RNReanimated, RNScreens. Did you forget to run "pod install" ?
info Found Xcode project "ExpoProject.xcodeproj"
info Building (using "xcodebuild -project ExpoProject.xcodeproj -configuration Debug -scheme ExpoProject -destination id=FD39E116-8B82-48DE-8D16-CFB30089D26F")
........
error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. To debug build logs further, consider building your app with Xcode.app, by opening ExpoProject.xcodeproj. Run CLI with --verbose flag for more details.
note: Using new build system
note: Planning build
note: Constructing build description
error: /Users/atsushiharada/source/ExpoProject/ios/Pods/Target Support Files/Pods-ExpoProject/Pods-ExpoProject.debug.xcconfig: unable to open file (in target "ExpoProject" in project "ExpoProject") (in target 'ExpoProject' from project 'ExpoProject')
error: /Users/atsushiharada/source/ExpoProject/ios/Pods/Target Support Files/Pods-ExpoProject/Pods-ExpoProject.debug.xcconfig: unable to open file (in target "ExpoProject" in project "ExpoProject") (in target 'ExpoProject' from project 'ExpoProject')
error: /Users/atsushiharada/source/ExpoProject/ios/Pods/Target Support Files/Pods-ExpoProject/Pods-ExpoProject.debug.xcconfig: unable to open file (in target "ExpoProject" in project "ExpoProject") (in target 'ExpoProject' from project 'ExpoProject')
error: /Users/atsushiharada/source/ExpoProject/ios/Pods/Target Support Files/Pods-ExpoProject/Pods-ExpoProject.debug.xcconfig: unable to open file (in target "ExpoProject" in project "ExpoProject") (in target 'ExpoProject' from project 'ExpoProject')


** BUILD FAILED **


npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! @ ios: `react-native run-ios`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the @ ios script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

npm ERR! A complete log of this run can be found in:
npm ERR!     /Users/atsushiharada/.npm/_logs/2020-02-23T01_41_06_972Z-debug.log
```

うーん。`pod install`した？って怒られてるみたいだけどよく分からん。

Expoの公式ドキュメントみてみると、`expo start`しろと書いてあるのでそっちやってみる。

```
$ npx expo start
```

なんかExpoの画面が表示される。

![Expo起動](/images/blog/2020-02-22-get-started-react-native-with-expo/expo-server.png)

QRコードをiPhoneで読み込んでみると「使用可能なデータがみつかりません」と怒られた。

あーiPhone実機にも専用アプリをいれる必要があるわけか。公式ドキュメントに書いてある。

アプリを落としたらサインアップしてもっかいQRコード読んで見る。

動いたーー！

![アプリ画面](/images/blog/2020-02-22-get-started-react-native-with-expo/blank.PNG)


## iOSネイティブのCollectionView(タイル)みたいの作る

前回同様NativeBaseを使う。公式ドキュメントにグリッド型のUIのサンプルがあるので参考にする。

[https://docs.nativebase.io/Components.html#Layout](https://docs.nativebase.io/Components.html#Layout)

```
$ npm install native-base --save
$ npm install react-native-easy-grid --save
```

App.js

```
import React, { Component } from 'react';
import { Container, Header } from 'native-base';
import { Col, Row, Grid } from 'react-native-easy-grid';
export default class App extends Component {
  render() {
    return (
      <Container>
        <Header />
        <Grid>
          <Col style={{ backgroundColor: '#635DB7', height: 200 }}></Col>
          <Col style={{ backgroundColor: '#00CE9F', height: 200 }}></Col>
        </Grid>
      </Container>
    );
  }
}
```

出来たー！！

![グリッドUI](/images/blog/2020-02-22-get-started-react-native-with-expo/grid.PNG)

## タイル型のグリッドにラーメン画像を並べてみる

ちょっと開発中に実機確認するのだるいなーと感じてきたので今度はシミュレータで起動してみる。

`expo start`したときに表示される画面に「Run on iOS simulator」というリンクがあるので、これクリックするだけ。めっちゃ楽やんけ。

App.js

```
import React, { Component } from 'react';
import { Image } from "react-native";
import { Container, Header } from 'native-base';
import { Col, Row, Grid } from 'react-native-easy-grid';
export default class App extends Component {
  render() {
    const rows  = []
    for (let i = 0; i < 6; i += 1) {
      rows.push(
        <Row>
          <Col>
            <Image style={{ flex: 1 }} source={{ uri: 'https://ramen-image-url' }} resizeMode="contain" />
          </Col>
          <Col>
            <Image style={{ flex: 1 }} source={{ uri: 'https://ramen-image-url' }} resizeMode="contain" />
          </Col>
          <Col>
            <Image style={{ flex: 1 }} source={{ uri: 'https://ramen-image-url' }} resizeMode="contain" />
          </Col>
        </Row>
      )
    }

    return (
      <Container>
        <Header />
        <Grid>
          { rows } 
        </Grid>
      </Container>
    );
  }
}
```

出来たーーーー！

![ラーメン画像をGrid表示](/images/blog/2020-02-22-get-started-react-native-with-expo/ramen-grid-view.png)

なんだかViewを作る部分がrenderの中で分離してしまってキモい。ちょっと調べた感じだとReactではこんな感じが普通っぽい。Vueの`v-for`みたいな方がスマートに見えるけどどうなんだろ。

