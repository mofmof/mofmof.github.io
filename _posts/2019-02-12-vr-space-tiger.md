---
layout: blog
title: VR宇宙空間で癒やしの世界を作る
category: blog
tags: [Unity, Asset Store, Oculus Go, VR, 宇宙]
summary: ちょっとVR酔いで疲れてきたので、仮想宇宙空間を癒やしの世界を作って癒やされてみたいと思います。
author: aharada
image: /images/blog/2019-02-12-vr-space-tiger/planet-tiger.gif
---

ちょっとVR酔いで疲れてきたので、仮想宇宙空間を癒やしの世界を作って癒やされてみたいと思います。

宇宙空間のAssetが欲しいところですが、ちょうど良さげなのが見つかりません。

下記エントリを見つけたけども`SPACE for Unity`というAssetが良さげだったんだけどちょっと高いので別のを探す。

[0013-12-22 - Nao_uの日記 - Game Programmerグループ](http://game.g.hatena.ne.jp/Nao_u/00131222)

[SPACE for Unity - Space Scene Construction Kit - Asset Store](https://assetstore.unity.com/packages/tools/level-design/space-for-unity-space-scene-construction-kit-7095?aid=1011lGbg&utm_source=aff)

`Planet Earth Free`という地球の3Dモデルを見つけたのでこれで行こう。

[Planet Earth Free - Asset Store](https://assetstore.unity.com/packages/3d/environments/sci-fi/planet-earth-free-23399)

![Planet Earth Free](/images/blog/2019-02-12-vr-space-tiger/planet-earth-free.png)

いつものようにAsset Storeからダウンロード/インポートして`Project -> Planet Earth Free -> Demo`を選択してプレイしてみたらUnity上では良い感じに表示された。

例によってOculus Goに対応させるため、Androidへ`Switch Platform`すると地球が水色の膜で覆われてしまう。。直し方がわからないので`Hierarchy`から削除しちゃう。

![水色の膜](/images/blog/2019-02-12-vr-space-tiger/planet.png)

![削除](/images/blog/2019-02-12-vr-space-tiger/glow.png)

続いて癒やし空間に必要なのは音楽。いつもお世話になってるフリー素材のページからそれらしいのを見つけてきた。いつもありがとう。

[フリーBGM素材 『Solemn Temple』 試聴ページ｜フリーBGM DOVA-SYNDROME](https://dova-s.jp/bgm/play8987.html)

Finderで開いて、`Project`にドロップすれば使えるようになります。`Project`から`Scene`ビューに該当ファイルをドロップするとプレイと同時にBGMが流れるようになります。

ref. [音をつける - Unity](https://unity3d.com/jp/learn/tutorials/projects/2d-shooting-game/adding-audio)

とりあえず。出来上がり。

## 宇宙空間でトラさんを飛ばす

本当はサメを飛ばしたかったんだけど、AssetにフリーのSharkが見つからなかった。下記リンクの写真みたいにしたかった。

[ネタバレ「シャークネード　エクストリーム・ミッション」あらすじ、キャスト、続編情報。｜キシマの映画ブログ](https://www.kishimamovie.com/entry/2016/07/28/113000)

仕方がないのでトラさんを飛ばします。

`Golden Tiger`というフリーのトラさんがAsset Storeにいるのでいつものようにダウンロード/インポート。

`Project -> Assets -> Tiger -> tiger_idle`を`Scene`ビューにドロップするとトラさんが登場します。

![トラを配置](/images/blog/2019-02-12-vr-space-tiger/set-tiger.png)

適当に配置。

```
Position X:0, Y: -28, Z:-72
Rotation X:-21, Y:17, Z-15
Scale X:6, Y:6, Z:6
```

トラにスクリプトを`Add Component`します。トラからみた前方方向にただ進むだけのスクリプトですね。

Tiger.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Tiger : MonoBehaviour
{
    void FixedUpdate() {
        transform.Translate(Vector3.forward * 0.1f);
    }
}
```

出来たー！画像では背景が真っ暗のようにみえますが、実際には瞬く星空が広がってます。癒やされます。

![癒やし空間完成](/images/blog/2019-02-12-vr-space-tiger/planet-tiger.gif)
