---
layout: blog
title: Oculus Goの開発環境を構築しようとしたが頓挫した(Mac, Unity, Android)
category: blog
tags: [Oculus Go, VR, Unity, Android]
summary: ついに ねんがんの Oculus Go をてにいれたぞ！購入後しばらくは既存のコンテンツで一通り遊んでましたが割と早く飽きました。
author: aharada
image: /images/blog/2019-01-13-oculus-go-development/main.png
---

ついに ねんがんの Oculus Go をてにいれたぞ！

購入後しばらくは既存のコンテンツで一通り遊んでましたが割と早く飽きました。というのもVRヘッドセットは重たいし、遊ぶにもなんか神経使うのでなかなか疲れる。必然的にコンテンツもライトに楽しめるものが多い印象。

ともすると試しになんか作ってみるかーとなったのでまずは開発環境を整備せな。

結論から言うとこの記事では開発環境の構築には失敗した。ビルド時にエラーが出る問題を解消出来なかった。引き続き頑張る。

## 主に参考にした記事

- [Oculus Go とUnityとMacで始めるVR開発 - Qiita](https://qiita.com/Hirai0827/items/f62588cc7d5c6c17e364)
- [UnityではじめるOculus Go VRアプリ開発 (Mac OS)  - Qiita](https://qiita.com/shiruco/items/435bce396c42f764e4ae)

## Android SDKなど

開発は基本的にUnityでやるんだと思うけど、Oculus GOはAndroidで動いているためAndroid Studioが必要だとか。インストールしていきます。

Android Studioを動かすにはjavaが必要っぽいので確認。既に1.8が入っているのでそのままでいいや。

```
$ java -version
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```

続いてbrewを使ってAndroid Studioをインストールします。

```
$ brew cask install android-studio
```

Android Studioを起動すると色々インストールしてくれるのでしばらく待つ。けっこう長い。たぶんAndroid SDKも入れてくれる。


## Oculusの設定

まずは開発者登録みたいのが必要。適当に登録してひとまずは組織情報を登録しておく(今回はサンプルなので適当で)。

https://dashboard.oculus.com/

![ダッシュボード](/images/blog/2019-01-13-oculus-go-development/dashboard.png)

Oculus GO実機の方はスマホアプリから開発者モードをONにすればOK。

![スマホアプリ1](/images/blog/2019-01-13-oculus-go-development/sp1.png)
![スマホアプリ2](/images/blog/2019-01-13-oculus-go-development/sp2.png)

## Unityの設定

難関Unityの設定。Unityを既に使っている人がいいが、ぼくは初めてなのでちょっと手間取っている。

まずはHomebrewでUnityをインストールする。

```
$ brew cask install unity
```

Unityを起動するとログインを求められるのでGoogleアカウントでログイン(Facebookでも問題ない)

現時点で商用は先なのでUnity personalを選択。サーベイとか出てくるので適当に選択してアクティベーション完了する。

![Unity1](/images/blog/2019-01-13-oculus-go-development/unity1.png)

とりあえずプロジェクトを作ってみる

![Unity2](/images/blog/2019-01-13-oculus-go-development/unity2.png)
![Unity3](/images/blog/2019-01-13-oculus-go-development/unity3.png)

これがUnityが。思ったよりも全時代的なUIだ。

![Unity4](/images/blog/2019-01-13-oculus-go-development/unity4.png)

Preferences->External ToolsでAndroidへのパスを設定する。

![Unity5](/images/blog/2019-01-13-oculus-go-development/unity5.png)

File->Build SettingでプラットフォームをAndroidに変更する、が出来ねえ

![Unity6](/images/blog/2019-01-13-oculus-go-development/unity6.png)

Open Download Pageを開くを何かダウンロードが始まる。Android Playerなるものをインストール始めるがまあ言われるがままにしておく。

するとSwitch Platformが選択出来るようになる

![Unity7](/images/blog/2019-01-13-oculus-go-development/unity7.png)

続いてBuildの設定をしていく。

PlayerSettings->Other Settings=>Identification Package Nameを適当に変更する(まだ公開しないのでデフォルトでなければなんでも良い)。

PlayerSettings->XRSettings->Virtual Reality SDKsにOculusを設定する

![Unity8](/images/blog/2019-01-13-oculus-go-development/unity8.png)

Buildしてみると怒られた

```
Minimum API Level Not Supported on Requested VR Device
Oculus Requires a Minimum API Level of 19.
You have selected 16
```

![Unity9](/images/blog/2019-01-13-oculus-go-development/unity9.png)

Other Settings->Identification->Mnimum API Levelの項目があるのでAndroid7.1を設定しておておく(Oculus Goと同じバージョンらしい)。

Build And Runをもう一回。

うごかねーー

```
Build failure
Unable to retrieve device properties. Please make sure the Android SDK is installed and is properly configured in the Editor. See the Console for more details. See the Console for details.
```

![Unity10](/images/blog/2019-01-13-oculus-go-development/unity10.png)

UnityはAndroidSDKToolsの最新版対応していないことが原因らしい。下記からダウンロードし、`~/Library/Android/sdk/tools`と差し替える必要があるとか。

[https://dl.google.com/android/repository/tools_r25.2.5-macosx.zip](https://dl.google.com/android/repository/tools_r25.2.5-macosx.zip)

一応バックアップに退避して差し替える

```
$ mv ~/Library/Android/sdk/tools ~/Library/Android/sdk/tools.org
$ mv ~/Downloads/tools ~/Library/Android/sdk/
```

再度Build And Runしてみるがまたエラー。

```
Android SDK is outdated
SDK Tools version 25.2.5 < 26.1.1.
```

![Unity11](/images/blog/2019-01-13-oculus-go-development/unity11.png)

どれを選択しても同じエラーになってしまう。。
うごかねーーー

今日はもう時間がないのでまたあとで頑張ってみます。

## 2019/01/22追記

仕切り直してみたら一瞬で解決しました。

Oculusアプリの方で開発者モードをONにしてはいたのですが、Oculus Go本体とUSB接続時に「USBデバッグしますか？」みたいな画面が表示されているのを見落としてました。

普通にチェックして先に進めばビルド出来るようになりました！
