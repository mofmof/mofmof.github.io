---
layout: blog
title: React Nativeに入門して、iOSネイティブのTableViewっぽいものを作る
category: blog
tags: [React Native]
summary: 個人的にはSwiftネイティブでアプリを書くのが好きなんですが、やはり新規開発でiOS/Androidのプロトタイプをサクッとみたいなシーンではちょっとつらい。昨今の空気感からすると、React NativeやFlutterが市民権を得はじめている気がするので、ちょっと入門してみる。
author: aharada
image: 
---

個人的にはSwiftネイティブでアプリを書くのが好きなんですが、やはり新規開発でiOS/Androidのプロトタイプをサクッとみたいなシーンではちょっとつらい。昨今の空気感からすると、React NativeやFlutterが市民権を得はじめている気がするので、ちょっと入門してみる。

昔Tiatium Mobileの波に乗ろうとして盛大にズッコケたのでクロスプラットフォーム系の技術は全て眉唾ものに見えていなのですが食わず嫌いはほどほどにしておきます。

## React Nativeのインストールから動くところまで

公式のチュートリアルではExpo使えと書いてある。が、Expoの良さを理解するためにあえて使わずにやってみる。

[https://facebook.github.io/react-native/docs/getting-started](https://facebook.github.io/react-native/docs/getting-started)

こちらの記事も参考にした。

[https://qiita.com/nskydiving/items/41e446ef5c821359ab79](https://qiita.com/nskydiving/items/41e446ef5c821359ab79)


nodeは既に入っていたのでインストールはスキップ。

watchmanなるものをインストール。

```
$ brew install watchman
```

XcodeもCocoaPodsも既に入れているので省略。

チュートリアル通り、シミュレーターを起動してみる。

```
$ open -a Simulator
```

ReactNativeプロジェクトを生成する。入れてたはずか途中でCocoaPods入ってねえからインスコしろと怒られたのでgemでインストールした。

```
$ npx react-native init AwesomeProject
```

うーん。なんかエラー。

```
✖ Installing CocoaPods dependencies (this may take a few minutes)
error Error: Failed to install CocoaPods dependencies for iOS project, which is required by this template.
Please try again manually: "cd ./AwesomeProject/ios && pod install".
CocoaPods documentation: https://cocoapods.org/
```

メッセージのとおりやってみるもやっぱりエラー。

```
$ cd ./AwesomeProject/ios && pod install

~

xcrun: error: SDK "iphoneos" cannot be located
xcrun: error: SDK "iphoneos" cannot be located
xcrun: error: SDK "iphoneos" cannot be located
xcrun: error: unable to lookup item 'Path' in SDK 'iphoneos'
/Users/atsushiharada/Library/Caches/CocoaPods/Pods/External/glog/2263bd123499e5b93b5efe24871be317-1f3da/missing: Unknown `--is-lightweight' option
Try `/Users/atsushiharada/Library/Caches/CocoaPods/Pods/External/glog/2263bd123499e5b93b5efe24871be317-1f3da/missing --help' for more information
configure: WARNING: 'missing' script is too old or missing
configure: error: in `/Users/atsushiharada/Library/Caches/CocoaPods/Pods/External/glog/2263bd123499e5b93b5efe24871be317-1f3da':
configure: error: C compiler cannot create executables
See `config.log' for more details
```

解決方法が載ってた。

[https://stackoverflow.com/questions/58934022/react-native-error-failed-to-install-cocoapods-dependencies-for-ios-project-w](https://stackoverflow.com/questions/58934022/react-native-error-failed-to-install-cocoapods-dependencies-for-ios-project-w)

```
$ sudo xcode-select --switch /Applications/Xcode.app
```

削除してもう一度プロジェクトを生成する。今回はエラーなくいけた！

```
$ rm -rf AwesomeProject
$ npx react-native init AwesomeProject
```

アプリを起動してみるもエラー。

```
$ npx react-native run-ios
error Unrecognized command "run-ios".
info Run "react-native --help" to see a list of all available commands.
```

ダメでした。react-nativeパッケージをインストールしなおしてみる。

```
$ npm install -g react-native
$ npx react-native run-ios
Command `run-ios` unrecognized. Make sure that you have run `npm install` and that you are inside a react-native project.
```

あ、プロジェクトのディレクトリに移動してなかったわ。気を取り直して。

```
$ cd AwesomeProject
$ npx react-native run-ios
```

動いたーー！

![Tutorial](/images/blog/2020-02-21-get-started-react-native/tutorial.png)

## NativeBase導入

この記事によるとOSSでいろんなコンポーネントが公開されているけど、なんでもホイホイ導入するのは良くないよ、だから[NativeBase](https://nativebase.io/)を使えって言ってる。

[https://blog.craftz.dog/lessons-learned-from-creating-my-mobile-app-to-build-a-high-quality-react-native-app-dcf021ce37ef](https://blog.craftz.dog/lessons-learned-from-creating-my-mobile-app-to-build-a-high-quality-react-native-app-dcf021ce37ef)

今さっきホイホイ導入するなよって言われたのにも関わらず、ホイホイとNativeBaseを導入してみることにします。

インストールする。

```
$ npm install native-base --save
```

公式のドキュメントとQiitaの記事を読みながら進める。

[https://docs.nativebase.io/docs/GetStarted.html](https://docs.nativebase.io/docs/GetStarted.html)

[https://qiita.com/keneo/items/d3a36d973c0b89829aa9](https://qiita.com/keneo/items/d3a36d973c0b89829aa9)

なにやらわからないけどlinkする。

```
$ react-native link
```

## iOSネイティブっぽいTableViewを置いてみる

App.js

```
/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 *
 * @format
 * @flow
 */

import {
  SafeAreaView,
  StyleSheet,
  ScrollView,
  View,
  // Text,
  StatusBar,
} from 'react-native';

import React, {Component} from 'react';
import { Container, Header, Content, List, ListItem, Text } from 'native-base';


class App extends Component {
  render() {
    return (
      <Container>
        <Header />
        <Content>
          <List>
            <ListItem>
              <Text>Simon Mignolet</Text>
            </ListItem>
            <ListItem>
              <Text>Nathaniel Clyne</Text>
            </ListItem>
            <ListItem>
              <Text>Dejan Lovren</Text>
            </ListItem>
          </List>
        </Content>
      </Container>
    );
  }
}

export default App;
```

なんかシミュレータがエラー吐いてる。

```
unrecognized font family 'ionicons'
```

シミュレータを再起動したらなおった。


```
$ react-native run-ios
```

出来たー！

![TableView](/images/blog/2020-02-21-get-started-react-native/tableview.png)
