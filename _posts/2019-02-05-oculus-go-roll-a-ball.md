---
layout: blog
title: Oculus GoでUnityチュートリアルの玉転がしゲームをVR化する
category: blog
tags: [Oculus Go, VR, Unity, Android]
summary: 前回Oculus Goの開発環境を構築しました。続いてチュートリアルをやっていこうと思います。
author: aharada
image: /images/blog/2019-02-05-oculus-go-roll-a-ball/main.png
---

前回Oculus Goの開発環境を構築しました。続いてチュートリアルをやっていこうと思います。

[Oculus Goの開発環境を構築しようとしたが頓挫した(Mac, Unity, Android) - もふもふ技術部](https://tech.mof-mof.co.jp/blog/oculus-go-development.html)

Unity公式チュートリアルに玉転がしゲームがあります。書いてあるとおりにやれば普通に動くので、それが動いている状態から、Oculus Goに対応させる手順を書いていきます。

[はじめてのUnity - Unity](https://unity3d.com/jp/learn/tutorials/projects/hajiuni-jp)

## Oculus Go対応させる

前回エントリの開発環境構築時同様に、`File -> Build Settings -> Player Settings`でAndroidバージョンや、`XR Settings`を設定する。

あとは基本的に以下のエントリをなぞっていけばいける。

[Unityの玉転がしチュートリアルを#OculusGoに対応させる - Qiita](https://qiita.com/m0a/items/98cdc1e03cf067769578)

`File -> Build Settings -> Platform`でAndroidを選択しSwitch Platformボタンを押します。

![Switch Platform](/images/blog/2019-02-05-oculus-go-roll-a-ball/switch-platform.png)

Build And RunをしてみてOculus Goで表示されることを確認。この時点ではOculus Goのコントローラーを認識してくれないので玉が動きません。

`Window -> Asset Store`を開いて、`Oculus Integration`をダウンロード・インポートします。これがなかなか時間かかるので根気強く待ちます。

![Oculus Integration](/images/blog/2019-02-05-oculus-go-roll-a-ball/oculus-integration.png)

ここが罠なのですが、この状態でビルドしても謎のエラーで落ちます。一旦Unityを再起動してみましょう。すると`Update Oculus Utilities Plugin`のダイアログが表示されるのでYes高須クリニックします。

![Update Oculus Utilities Plugin](/images/blog/2019-02-05-oculus-go-roll-a-ball/update-oculus-utilities-plugin.png)

再度再起動が求められるので言われるがまま再起動しましょう。

続いて、チュートリアル中で作った`PlayerController`クラスをリネームしましょう。適当に`PlayerBallController`とかでOK。`Oculus Integration`の中のクラス名と重複してしまっているらしい。

ここまで出来たら`Build And Run`してみます。

`"OVRPlugin.aar" is denied.`というエラーで落ちました。運の良いことにこのエラーは他の人が解決した情報があります。

`OVRPlugin.aar`ファイルの拡張子を変えて`OVRPlugin.bak`にすれば良いらしい。詳しくはリンク先を参照。

[【Oculus Go】コントローラを表示する - Qiita](https://qiita.com/eKushida/items/0ba3599536c3aef51f04)

再びビルド。

あ、タッチパッドで動くじゃん！やったー！
動画とか載っけたいけどだいぶめんどそうなので今回はやめとく。

## ハマった謎のエラー集

サクッと書きましたが、Oculus Goアプリの開発はなかなか茨の道かも知れない。ここまで動かすのに3,4回くらい振り出しに戻ってトライしたわけだけど、やるたびやるたび違うエラーに悩まされた。つら。

### その1 System.Security.Policy
ビルド時にエラーが出る。

```
Assets/Oculus/Avatar/Scripts/OvrAvatarSkinnedMeshRenderPBSV2Component.cs(4,23): error CS0234: The type or namespace name 'Policy' does not exist in the namespace 'System.Security' (are you missing an assembly reference?)
```

該当エラー行をクリックするとエディタが開くので、`using System.Security.Policy;`の行をコメントアウトするとビルド出来るようになる。

["error CS0234: The type or namespace name 'Policy' does not exist in the namespace 'System.Security'" エラーが発生する (Unityプログラミング)](https://www.ipentec.com/document/unity-error-cs0234-the-type-or-namespace-name-policy-does-not-exist-in-namespace-system-security)

なぜ使わないものを`using`しているのか謎。

### その2 No Android devices connected

ときおり突然Oculus Goが認識されなくなることがある。ビルド時に`No Android devices connected`ってダイアログがでます。ちょいちょいでてきてうざい。

![No Android](/images/blog/2019-02-05-oculus-go-roll-a-ball/no-android.png)


USBケーブル抜き差ししたりUnity再起動したりOculus Goを再起動したりPC再起動したりすれば大抵なおる。

今回はそれでもなおらず、アレアレ？って思ってたらUSBケーブル刺さってないやんけ！！！！ばーか！！！
