---
layout: blog
title: Joy-Conのジャイロセンサーの威力を試す
category: blog
tags: [Unity, Joy-Con, 羊]
summary: Joy-Conのセンサーを使って何かやってみます。
author: iwai
image: /images/blog/2019-04-04-unity-joycon-play/sheep_shoot_sum.png
---

前回UnityでJoy-Conの入力を受け付けるとこまでやったので、今回はその入力を使ってUnityっぽい何かをしてみようと思います。

<br>

## TL;DR

ジャイロセンサーで向きを自由自在に変えられる羊にビームを撃たせます。羊やらビームやらのオブジェクト置いて記事内のスクリプトをコピペすればすぐに実現できます。`JoyConSample.cs`の完成形はページ最下部に置いてあります。

<br>

## ジャイロセンサーの入力を扱う

前回、[UnityでJoy-Conを使ってみる - もふもふ技術部](/blog/unity-joycon-introduce.html)でUnity世界にJoy-Conを召喚しました。すでにJoy-Conから諸々の値を受け取れる状態なので、これをこねくりまわします。

ジャイロセンサーを扱うにあたって、今回はこちらの記事を参考にさせていただいてます。

[Nintendo SwitchのJoy-Conを使ってユニティちゃんの腕を動かす](https://qiita.com/fuloru169/items/54cbb3571d389aa074f7)

<br>

なにはともあれ、羊がいないと話が始まりません。可愛らしい動物たちのアセットがストアにあるので、インポートしましょう。

[Farm Animals Set](https://assetstore.unity.com/packages/3d/farm-animals-set-97945)

インポートしたら、Prefabsから`SheepWhite`をSceneにドラッグ&ドロップで羊を設置します。可愛い。

![sheep-intro](/images/blog/2019-04-04-unity-joycon-play/sheep_intro.png)
![sheep-init](/images/blog/2019-04-04-unity-joycon-play/sheep_init.png)

地面は適当に`GameObject > 3D Object > Plane`を設置して、`Standard Assets`(公式が用意してるすごいアセット。ストアで無料です)の`Environment > TerrainAssets > SurfaceTextures > GrassHillAlbedo`をセットしたMaterialをくっつけました。

![plane](/images/blog/2019-04-04-unity-joycon-play/plane.png)

で羊ちゃんが出現したので、こいつをジャイロセンサーで操作していきましょう。

まず、羊ちゃんの名前を`BeamSheep`にしておきます。安直。

そしたら前回作成した｀JoyConSample.cs`を変更します。

羊をスクリプト内で扱えるようにするところ

```
public class JoyConSample : MonoBehaviour
{
    private static readonly Joycon.Button[] m_buttons =
        Enum.GetValues (typeof (Joycon.Button)) as Joycon.Button[];

    private List<Joycon> m_joycons;
    private Joycon m_joyconL;
    private Joycon m_joyconR;
    private Joycon.Button? m_pressedButtonL;
    private Joycon.Button? m_pressedButtonR;

    // 追加
    private GameObject sheep;

    private void Start ()
    {
        SetControllers ();
        
        // 追加
        sheep = GameObject.Find ("BeamSheep");

        以下略
```

ジャイロセンサーの値で羊の向きを変えるところ

```
private void Update ()
{
    m_pressedButtonL = null;
    m_pressedButtonR = null;

    if (m_joycons == null || m_joycons.Count <= 0) return;

    SetControllers ();

    foreach (var button in m_buttons)
    {
        if (m_joyconL.GetButton (button))
        {
            m_pressedButtonL = button;
        }
        if (m_joyconR.GetButton (button))
        {
            m_pressedButtonR = button;
        }
    }

    if (Input.GetKeyDown (KeyCode.Z))
    {
        m_joyconL.SetRumble (160, 320, 0.6f, 200);
    }
    if (Input.GetKeyDown (KeyCode.X))
    {
        m_joyconR.SetRumble (160, 320, 0.6f, 200);
    }

    // 以下追加と変更箇所
    const float MOVE_PER_CLOCK = 0.01f;
    Vector3 joyconGyro;

    // Joy-Conのジャイロ値を取得
    joyconGyro = m_joyconR.GetGyro ();
    Quaternion sheepQuaternion = sheep.transform.rotation;
    // Joy-Conを横持ちした時にうまいこと向きが変わるようになっています。
    sheepQuaternion.x += -joyconGyro[0] * MOVE_PER_CLOCK;
    sheepQuaternion.y -= -joyconGyro[2] * MOVE_PER_CLOCK;
    sheepQuaternion.z -= -joyconGyro[1] * MOVE_PER_CLOCK;
    sheep.transform.rotation = sheepQuaternion;

    // 右Joy-ConのBボタンを押すと羊の向きがリセットされます。
    if (m_joyconR.GetButtonDown (m_buttons[0]))
    {
        sheep.transform.rotation = new Quaternion (0, 0, 0, 0);
    }

    以下略
```

ここまでやると羊が自由自在に向きを変えるようになっていると思います。

カメラと羊の位置をいい感じにして再生してみましょう。**Joy-Conは横持ちしてください。** Joy-Conと羊の同期にはちょいちょいズレがありますが、ご愛嬌です。Bボタンを押せば向きがリセットされます。

![wild_sheep](/images/blog/2019-04-04-unity-joycon-play/sheep.mov.gif)

<br>

## ビーム撃たす

完全に蛇足ですが、せっかくなので羊にビームを撃たせます。

[参考 - 【Unity】ParticleSystemでレーザービームを作る](http://r-ngtm.hatenablog.com/entry/2017/05/29/162730)

この記事を丸々参考にして、ビームを作成します。Unityのバージョンが違うためか設定項目名は若干異なりましたが、今回はこんな感じで作りました。名前は`SheepBeam`にしました。

![sheep_beam](/images/blog/2019-04-04-unity-joycon-play/sheep_beam.png)

次に、ビームのオン/オフ制御を楽にするため`SheepNozzle`を作成します。HierarchyのBeamSheep上で右クリックし、`3D Object > Cube` を作成します。
- 名前は`SheepNozzle`に変更
- Scaleを0.001, 0.001, 0.001にしちゃいます
- 位置は羊の口元に移動させます
- BeamSheepとSheepNozzleのRotationは全て0にしておきましょう

できたら、`SheepNozzle`の下に`SheepBeam`を置きましょう。こうです。

![hierarchy](/images/blog/2019-04-04-unity-joycon-play/beamsheep_sheepbeam.png)

ここまできたらもうビームを垂れ流す羊を自由自在に動かせるようになっていることと思います。が、それは非常に迷惑なのでビームのオン/オフをスクリプトで制御しましょう。`JoyConSample.cs`を編集します。

フィールドの追加

```
public class JoyConSample : MonoBehaviour
{
    private static readonly Joycon.Button[] m_buttons =
        Enum.GetValues (typeof (Joycon.Button)) as Joycon.Button[];

    private List<Joycon> m_joycons;
    private Joycon m_joyconL;
    private Joycon m_joyconR;
    private Joycon.Button? m_pressedButtonL;
    private Joycon.Button? m_pressedButtonR;

    private GameObject sheep;

    // 追加
    [SerializeField]
    private GameObject nozzle;

```

制御ロジック追加

```
private void Update ()
{
    m_pressedButtonL = null;
    m_pressedButtonR = null;

    if (m_joycons == null || m_joycons.Count <= 0) return;

    SetControllers ();

    foreach (var button in m_buttons)
    {
        if (m_joyconL.GetButton (button))
        {
            m_pressedButtonL = button;
        }
        if (m_joyconR.GetButton (button))
        {
            m_pressedButtonR = button;
        }
    }

    if (Input.GetKeyDown (KeyCode.Z))
    {
        m_joyconL.SetRumble (160, 320, 0.6f, 200);
    }
    if (Input.GetKeyDown (KeyCode.X))
    {
        m_joyconR.SetRumble (160, 320, 0.6f, 200);
    }

    const float MOVE_PER_CLOCK = 0.01f;
    Vector3 joyconGyro;

    joyconGyro = m_joyconR.GetGyro ();
    Quaternion sheepQuaternion = sheep.transform.rotation;
    sheepQuaternion.x += -joyconGyro[0] * MOVE_PER_CLOCK;
    sheepQuaternion.y -= -joyconGyro[2] * MOVE_PER_CLOCK;
    sheepQuaternion.z -= -joyconGyro[1] * MOVE_PER_CLOCK;
    sheep.transform.rotation = sheepQuaternion;

    if (m_joyconR.GetButtonDown (m_buttons[0]))
    {
        sheep.transform.rotation = new Quaternion (0, 0, 0, 0);
    }


    // 追加
    // 右Joy-ConのAボタンでオン/オフを切り替えます
    if (m_joyconR.GetButtonDown (m_buttons[1]))
    {
        if (nozzle.activeSelf)
        {
            nozzle.SetActive (false);
        }
        else
        {
            nozzle.SetActive (true);
        }
    }
}
```

すると、前回作成して`JoyConSample`をセットしてある`Example`オブジェクトに、Nozzleをセットできるようになります。そこに`SheepNozzle`をドラッグ&ドロップしましょう。

![example_hie](/images/blog/2019-04-04-unity-joycon-play/example_hie.png)
![example_nozzle](/images/blog/2019-04-04-unity-joycon-play/example_nozzle.png)

最後に`SheepNozzle`を非アクティブにして完成です。名前横のチェックを外しましょう。こうすることにより、Aボタンを押すまではビームを吐かなくなります。

![nozzle_active](/images/blog/2019-04-04-unity-joycon-play/nozzle_active.png)

<br>

## 完成

満足した

![sheep_beam_completed](/images/blog/2019-04-04-unity-joycon-play/sheep_beam_trimed.mov.gif)

## スクリプト最終形

```
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

public class JoyConSample : MonoBehaviour
{
    private static readonly Joycon.Button[] m_buttons =
        Enum.GetValues (typeof (Joycon.Button)) as Joycon.Button[];

    private List<Joycon> m_joycons;
    private Joycon m_joyconL;
    private Joycon m_joyconR;
    private Joycon.Button? m_pressedButtonL;
    private Joycon.Button? m_pressedButtonR;

    private GameObject sheep;

    [SerializeField]
    private GameObject nozzle;

    private void Start ()
    {
        SetControllers ();

        sheep = GameObject.Find ("BeamSheep");
    }

    private void Update ()
    {
        m_pressedButtonL = null;
        m_pressedButtonR = null;

        if (m_joycons == null || m_joycons.Count <= 0) return;

        SetControllers ();

        foreach (var button in m_buttons)
        {
            if (m_joyconL.GetButton (button))
            {
                m_pressedButtonL = button;
            }
            if (m_joyconR.GetButton (button))
            {
                m_pressedButtonR = button;
            }
        }

        if (Input.GetKeyDown (KeyCode.Z))
        {
            m_joyconL.SetRumble (160, 320, 0.6f, 200);
        }
        if (Input.GetKeyDown (KeyCode.X))
        {
            m_joyconR.SetRumble (160, 320, 0.6f, 200);
        }

        const float MOVE_PER_CLOCK = 0.01f;
        Vector3 joyconGyro;

        joyconGyro = m_joyconR.GetGyro ();
        Quaternion sheepQuaternion = sheep.transform.rotation;
        sheepQuaternion.x += -joyconGyro[0] * MOVE_PER_CLOCK;
        sheepQuaternion.y -= -joyconGyro[2] * MOVE_PER_CLOCK;
        sheepQuaternion.z -= -joyconGyro[1] * MOVE_PER_CLOCK;
        sheep.transform.rotation = sheepQuaternion;

        if (m_joyconR.GetButtonDown (m_buttons[0]))
        {
            sheep.transform.rotation = new Quaternion (0, 0, 0, 0);
        }

        if (m_joyconR.GetButtonDown (m_buttons[1]))
        {
            if (nozzle.activeSelf)
            {
                nozzle.SetActive (false);
            }
            else
            {
                nozzle.SetActive (true);
            }
        }
    }

    private void SetControllers ()
    {
        m_joycons = JoyconManager.Instance.j;
        if (m_joycons == null || m_joycons.Count <= 0) return;
        m_joyconL = m_joycons.Find (c => c.isLeft);
        m_joyconR = m_joycons.Find (c => !c.isLeft);
    }
}

```
