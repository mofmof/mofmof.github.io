---
layout: blog
title: UnityのTerrainでゲームっぽい地形を生成する
category: blog
tags: [Unity, Asset Store, ゾンビ, Terrain, Skybox]
summary: Unityはすごいなあ。地形を生成するのもこんなに簡単！やってみましょう。
author: aharada
image: /images/blog/2019-02-26-unity-zombies-chase-field/field2.png
---

Unityはすごいなあ。地形を生成するのもこんなに簡単！やってみましょう。

参考: [【Unity入門】3分で地面を生成!Terrainで見栄えの良い地面を作ろう](https://www.sejuku.net/blog/70742)

空っぽのSceneがある状態からスタート。まずは`Hiearchy -> Create -> Terrain`でX,Y,Zそれぞれ0の位置に配置します。このTerrainってやつが地形を配置出来るやつっぽい。

![Terrain](/images/blog/2019-02-26-unity-zombies-chase-field/terrain.png)

続いて。`Asset Store`から地形生成出来る`Asset`をダウンロード/インポートします。

[Terrain Toolkit 2017](https://assetstore.unity.com/packages/tools/terrain/terrain-toolkit-2017-83490)

`Project -> com -> heparo -> terrain -> toolkit -> scripts -> Terrain Toolkit`を`Terrain`の`Inspector`にドラッグ・ドロップする。

![Terrain Toolkit](/images/blog/2019-02-26-unity-zombies-chase-field/asset-store.png)

続いて`Inspector`から`Terrain Toolkit -> Terrain models -> PAMPA`を選択して草原っぽいのを作る。

ひとまず生成出来た。

![フィールド1](/images/blog/2019-02-26-unity-zombies-chase-field/field1.png)

表面に凹凸が激しく、ゾンビさんが歩くにはしんどそうなので調整してあげます。

`Inspector`から`Terrain Toolkit -> Toolkit -> Erode -> Thermal -> Filters -> Preset`を`Fast,Harsh Erosion`にして`Apply thermal erosion`ボタンを押します。

これでゾンビさんでも歩けそうになりましたね！ゾンビさんが喜ぶ顔を見るのが楽しみです。

![フィールド2](/images/blog/2019-02-26-unity-zombies-chase-field/field2.png)

## 背景の空を変更する

Unityデフォルトの空がかなり微妙な雰囲気を醸し出すので変更したい。

参考: [【Unity3D】背景（Skybox）を変える方法【図解】｜Unishar-ユニシャー](https://miyagame.net/skybox/)

これも実は`Asset Store`で手に入っちゃう。すごい。探すときちょっと困ったんだけど、`2D -> Textures & Materials -> Sky`っていうカテゴリに入ってる。背景はただのテクスチャだから3D,2Dは関係ない模様。

[Free HDR Sky](https://assetstore.unity.com/packages/2d/textures-materials/sky/free-hdr-sky-61217)というのを使うことにする。

![Free HDR Sky](/images/blog/2019-02-26-unity-zombies-chase-field/free-hdr-sky.png)

`Window -> Rendering -> Lighting Settings`で`Skybox Material`に先程インポートしたSkyboxのテクスチャをドロップすれば出来上がり。

![Skybox Materialを設定](/images/blog/2019-02-26-unity-zombies-chase-field/skybox-material.png)

空がきれいになった！

![空](/images/blog/2019-02-26-unity-zombies-chase-field/sky.png)

## ゾンビを配置する

前回の、[Unityで追いかけてくるゾンビを実装する](/blog/unity-zombies-chase.html)と同様に適当にゾンビさんとPlayerを配置して、`Navigation`から`Bake`します。

![Bake](/images/blog/2019-02-26-unity-zombies-chase-field/bake.png)

ゾンビさんが追いかけてくるけど、地形が平らじゃないせいなのか、ちょいちょい変な方向に歩いていく。

![追いかけてくるゾンビ](/images/blog/2019-02-26-unity-zombies-chase-field/zombies.gif)

## おまけ

やばいAssetを見つけてしまった。

[Hyahhaa Easy Outlaw -free](https://assetstore.unity.com/packages/3d/characters/hyahhaa-easy-outlaw-free-110898)

![ヒャッハー](/images/blog/2019-02-26-unity-zombies-chase-field/hyahha.png)
