---
layout: blog
title: 3Dオレの部屋をVR対応してゾンビさんと対面する
category: blog
tags: [Unity, Asset Store, Oculus Go, VR, ゾンビ]
summary: やはり3Dと言えばゾンビ。ゾンビと言えば3D。せっかくだしオシャレなインテリアとして部屋にゾンビさんを置いてみます。
author: aharada
image: /images/blog/2019-02-12-unity-my-room-for-oculus-go/walking-dead.gif
---

前回Unityで3Dオレの部屋をサクッと作ったので、今回はOculus Goのコントローラーで操作出来るようにしてみます。

[Unityで3Dオレの部屋を作って歩き回る](/blog/unity-my-room.html)

Oculus Go対応も過去にやってるので参照されたし。

[Oculus GoでUnityチュートリアルの玉転がしゲームをVR化する - もふもふ技術部](/blog/oculus-go-roll-a-ball.html)

前回同様に、`File -> Build Settings -> Platform`でAndroidを選択し`Switch Platform`ボタンを押して、`Build And Run`でとりあえずVR空間になることを確認。

またまた同様Asset Storeから`Oculus Integration`をダウンロード/インポートします。インポートが終わったらとりあえず壊れていないことを確認するためビルドしてみる。しかしUnityでVR開発はとにかく待ち時間が長い。待ってばかりだ。

特にビルドエラーもなく表示出来たので次のステップへ行く。

## Oculus Goのコントローラー対応

例によって`MainPlayerController.cs`の`FixedUpdate()`と`Update()`にコードを追加します。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class MainPlayerController : MonoBehaviour
{
    void Update() {
        OVRInput.Update();
    }

    void FixedUpdate ()
    {
#if UNITY_ANDROID && !UNITY_EDITOR
        OVRInput.FixedUpdate();
        Vector2 vector =  OVRInput.Get(OVRInput.Axis2D.PrimaryTouchpad, OVRInput.Controller.RTrackedRemote);
        float x = vector.x;
        float y = vector.y;

        float speed = 0.0f;
        if (x > 0.6) {
            speed += 60.0f;
            Quaternion rot = Quaternion.AngleAxis(speed * Time.deltaTime, Vector3.up);
            transform.localRotation = rot * transform.localRotation;
        } else if (x < -0.6) {
            speed += -60.0f;
            Quaternion rot = Quaternion.AngleAxis(speed * Time.deltaTime, Vector3.up);
            transform.localRotation = rot * transform.localRotation;
        }

        if (y > 0.7) {
            transform.Translate(Vector3.forward * 0.1f);
        } else if (y < -0.7) {
            transform.Translate(Vector3.forward * -0.1f);
        }
#else
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
#endif
    }
}
```

タッチパッド部分の左右上下の端の方をタッチすると回転したり前進・後進したりするようになる。

ちょっと冗長なコードあるけど、ビルドがクソ遅くてデバッグ大変なのでスルーして欲しい。

## ゾンビさんと対面する

やはり3Dと言えばゾンビ。ゾンビと言えば3D。せっかくだしオシャレなインテリアとして部屋にゾンビさんを置いてみます。

Asset Storeにフリーのゾンビさんがいるので、ダウンロード/インポート。

![フリーのゾンビさん](/images/blog/2019-02-12-unity-my-room-for-oculus-go/free-zombie.png)

`Project -> Assets -> Zombie -> Prefabs -> Zombie`をSceneビューにドラッグ・ドロップし、MainPlayerの正面に来るように座標を合わせる。

```
Position X:3, Y:0, Z:5
Rotation X:0, Y:-90, Z:0
Scale X:2, Y:2, Z:2
```

![ゾンビさんを配置1](/images/blog/2019-02-12-unity-my-room-for-oculus-go/set-zombie1.png)

![ゾンビさんを配置2](/images/blog/2019-02-12-unity-my-room-for-oculus-go/set-zombie2.png)

デフォルトでアニメーションが設定されていて、何もしていないのに歩きだして勝手に倒れてしまいます。

![倒れるゾンビさん](/images/blog/2019-02-12-unity-my-room-for-oculus-go/down-zombie.png)

そのまま歩き続けるようにアニメーションを削除します。

`Project -> Assets -> Zombie -> Animation -> Zombie`を選択し、`Inspector -> Open`するとAnimatorビューが表示されるので`attack`と`fallingback`を削除します。

![アニメーションを削除](/images/blog/2019-02-12-unity-my-room-for-oculus-go/animation.png)

これでゾンビさんが歩き続けてくれるようになります。そのまままっすぐ歩いていくゾンビさんを見ていると、ソファと壁を突き抜けて空中散歩していく圧巻のシーンを見れます(プレイヤーは落下していくけど)。

![歩くゾンビさん](/images/blog/2019-02-12-unity-my-room-for-oculus-go/walking-dead.gif)


## 番外：ログを出力する
Oculus Goのコントローラーを操作したときどんな値が返ってくるのか知りたかったのでデバッグログを出力したかった。

ログを表示したい箇所でこんなコードを書く。

```
Debug.Log("hogehoge");
```

Oculus GoはAndroidでもあるのでこんな感じのコマンドを叩けばログが出力される。だがしかし`FixedUpdate`メソッド内でデバッグログ出力されると大量に流れるので読みづらい。

```
$ adb logcat -s Unity ActivityManager PackageManager
```
