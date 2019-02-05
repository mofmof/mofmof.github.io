---
layout: blog
title: Unityで3Dオレの部屋を作って歩き回る
category: blog
tags: [Unity, Asset Store]
summary: VRで部屋の中を自由に動き回りたかったので、まずはUnity単独で部屋の中を動き回れるアプリを作る。
author: aharada
image: /images/blog/2019-02-05-unity-my-room/walk.gif
---

VRで部屋の中を自由に動き回りたかったので、まずはUnity単独で部屋の中を動き回れるアプリを作る。

## 部屋を作って入る

まずは適当に新規プロジェクトを作ります。

自前で部屋を組み立てるのは大変だるいので、Asset Storeからimportします。今回は[Pack Gesta Furniture #1 - Asset Store](https://assetstore.unity.com/packages/3d/props/furniture/pack-gesta-furniture-1-28237)というAssetを使用します。

![Asset](/images/blog/2019-02-05-unity-my-room/room.png)

ちなみにGoogleでAsset Storeと欲しいシーンをキーワードに入れて検索すれば色々見つかる。FPS用のマップとか、VR定番のゾンビとか色々揃ってて至れり尽くせり。

[Unity用無料3DモデルAsset　100個まとめ - NAVER まとめ](https://matome.naver.jp/odai/2143282255561744801)

例によってAsset Storeで検索して、Download/Importします。相変わらず時間かかる。

![Assetをダウンロード](/images/blog/2019-02-05-unity-my-room/asset-store.png)

![インポート](/images/blog/2019-02-05-unity-my-room/import.png)

Importが済んだら、Assetsの中にFurniture_ges1というフォルダが出来るので`scene`とかをポチポチしてたらなんかアプリに適用された。

Build And Runしてみと、3Dの部屋の中入ることが出来ます。簡単。

スクショ取りそこねたスマン。

## 部屋の中にプレイヤーを設置

なかなか苦戦した。

まずは[はじめてのUnity - Unity](https://unity3d.com/jp/learn/tutorials/projects/hajiuni-jp)のチュートリアルのときやったことを思い出し、MainPlayerオブジェクトを置きます。

今回はプレイヤーは不可視で良いので`Hierarchy -> Create -> Create Empty`します。

Positionを`X:0, Y:1, Z:5`、Rotationを`X:0, Y:90, Z:5`に設定。

このままだと床を突き抜けて永久に落下するので、`MainPlayer`に`Rigidbody`と`Box Colider`を`Add Component`します。設定値はとりあえずデフォルトでOK。

![MainPlayer](/images/blog/2019-02-05-unity-my-room/main-player.png)

あと床の方も同様に`Box Colider`を`Add Component`する。床は`Hierarchy -> scene -> Models -> interior -> Plane001`にある。

ここまで出来たらプレイヤーが床を突き抜けなくなる。

続いて追従するカメラを設定する。これもチュートリアルと同じ要領でOK。

元々`Camera`オブジェクトは存在してるので、Positionを`X:0, Y:2, Z:0`、Rotationを`X:0, Y:0, Z:0`にして、`Hierarchy`から`MainPlayer`にドラッグ＆ドロップする。

![Camera](/images/blog/2019-02-05-unity-my-room/camera.png)

`Camera`オブジェクトに`Sciprt`を`Add Component`する。

FollowPlayer.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class FllowPlayer : MonoBehaviour
{
    public Transform target;  // ターゲットへの参照
    private Vector3 offset;  // 相対座標

    void Start ()
    {
        //自分自身とtargetとの相対距離を求める
        offset = GetComponent<Transform>().position - target.position;
    }

    void Update ()
    {
        // 自分自身の座標に、targetの座標に相対座標を足した値を設定する
        GetComponent<Transform>().position = target.position + offset;
    }
}
```

VRオレの部屋できたー！！

## 部屋の中を移動できるようする
ここはチュートリアル通りにはいかなかった。`BoxColider`と`Rigidbody`で`AddForce`すると立方体がころころ転がってしまう。`Freeze Rotation`をチェックすれば回転しなくなるが、`AddForce`ではカメラ視点で進行方向に進むのが難しかったので別のやりかたにする。

![Rotation Object](/images/blog/2019-02-05-unity-my-room/rotation-object.gif)

`MainPlayer`オブジェクトに`Script`を`Add Component`します。

MainPlayerController.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MainPlayerController : MonoBehaviour
{
    public float speed = 10;

    void FixedUpdate ()
    {
        // 左右キー入力に応じて回転速度を決定
        float speed = 0.0f;
        if (Input.GetKey(KeyCode.LeftArrow)) { speed += -90.0f; }
        if (Input.GetKey(KeyCode.RightArrow)) { speed += 90.0f; }

        // Y軸(Vector3.up)周りを１フレーム分の角度だけ回転させるQuaternionを作成
        Quaternion rot = Quaternion.AngleAxis(speed * Time.deltaTime, Vector3.up);

        // 元の回転値と合成して上書き
        transform.localRotation = rot * transform.localRotation;

        if (Input.GetKey (KeyCode.UpArrow))
            transform.Translate( Vector3.forward * 0.1f);
        if (Input.GetKey (KeyCode.DownArrow))
            transform.Translate( Vector3.back * 0.1f);
    }
}
```


これで左右キー入力でプレイヤーの向きを変えて、上下キーで前進・後退出来るようになりました！！

ちなみに`Colider`を設定していないオブジェクトもあるので、壁とか突き抜けちゃいます。

![部屋を歩く](/images/blog/2019-02-05-unity-my-room/walk.gif)
