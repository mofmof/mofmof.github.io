---
layout: blog
title: UnityでJoy-Conを使ってみる
category: blog
tags: [Unity, Joy-Con]
summary: Nintendo SwitchのJoy-Conの入力をUnityで受け付けてみる編です。
author: iwai
image: /images/blog/2019-04-03-unity-joycon-introduce/output.png
---

原田見習いの岩井です。師匠がUnityにお熱な傍らで、弟子の僕もここぞとばかりに時間をつぎ込みVR新規事業を模索していました。Unity超楽しい。自分のクリエイティビティの限界を感じる。

とはいえUnityは超楽しいので、もうちょっとUnityの深淵に近づいてみたい。深淵に近づくとき、深淵もまた云々。

<br>

## TL;DR

複数回にわたってお届けする（予定）Joy-Conで遊ぶ編、今回はUnity世界にJoy-Conを召喚するところまでやってみます。

<br>

## Joy-Conとは

任天堂株式会社より2017/3/3に発売されたゲームハードであるNintendo Switchのコントローラーです。こんなやつ。

![Joy-Con](/images/blog/2019-04-03-unity-joycon-introduce/joycon-official.png)

[公式の周辺機器紹介ページ](https://www.nintendo.co.jp/hardware/switch/accessories/)


なんとこやつ、各種ボタンに加えて加速度センサーやジャイロセンサー、果てはモーションIRカメラまで搭載されていて、しかもUnityでそれらを簡単に受け取り・処理するための便利なライブラリもあると！
これは**いちスプラトゥーンプレイヤーとして、いちスマブラプレイヤーとして、いち任天堂ファン**として触らずにはいられません！！**あああ！！**

ちなみに、こちらの記事を全体的に参考にさせていただいてます。

[【Unity】Nintendo Switch の Joy-Con のジャイロ・加速度・傾きの値を取得したり、振動させたりすることができる「JoyconLib」紹介](http://baba-s.hatenablog.com/entry/2017/11/12/090000)

<br>

---

<br>

## 環境を整える

1. まずはUnityで適当に新規プロジェクトを作ります。今回はmofmofの人間がJoy-Conで遊ぶので
![project_name](/images/blog/2019-04-03-unity-joycon-introduce/project.png)
にしました。mofulという単語はこの世にないです（たぶん）


2. Scriptやらを入手
それぞれGitHubからZIPで入手しておきましょう。

    [JoyconLib](https://github.com/Looking-Glass/JoyconLib)
    
    [Unity-Wiimote](https://github.com/Flafla2/Unity-Wiimote)


3. Unityに導入
- `JoyconLib-master > Packages > con.lookingglass.joyconlib > JoyconLib_scripts` をプロジェクトのAssetsに追加します。
- ` Unity-Wiimote-master > Assets > Wiimote > Plugins > (OS) > hidapi(bundle or dll)` のうち、OSに応じたものをプロジェクトのAssetsに追加します。macならhidapi.bundleをインポートすればOKです。

    結果、こんな感じ

    ![project-tree](/images/blog/2019-04-03-unity-joycon-introduce/project-tree.png)

<br>

## Joy-Conをmacに接続する

Bluetoothでつなぐことができます。

1. Joy-Conのシンクロボタンを長押しする
![joycon-side](/images/blog/2019-04-03-unity-joycon-introduce/joycon-side.png)

2. システム環境設定 > Bluetooth より、それぞれ接続しましょう
![joycon-connect](/images/blog/2019-04-03-unity-joycon-introduce/joycon-connect.png)

<br>

## Joy-Con疎通

さて、実際に入力値を受け取ってみます。

まずはお好みでC#スクリプトをおくScriptsフォルダを作成します。
したらProject > CreateからC#Scriptを新規作成しましょう。名前は適当に`JoyConSample`とかで。これならたぶん他のnamespaceと被らないです。

`JoyConSample.cs`の内容はこちら、丸コピペで大丈夫です。参考記事と一部相違がありますのでご注意。

```JoyConSample.cs
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

    private void Start ()
    {
        SetControllers ();
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
    }

    private void OnGUI ()
    {
        var style = GUI.skin.GetStyle ("label");
        style.fontSize = 24;

        if (m_joycons == null || m_joycons.Count <= 0)
        {
            GUILayout.Label ("Joy-Con が接続されていません");
            return;
        }

        if (!m_joycons.Any (c => c.isLeft))
        {
            GUILayout.Label ("Joy-Con (L) が接続されていません");
            return;
        }

        if (!m_joycons.Any (c => !c.isLeft))
        {
            GUILayout.Label ("Joy-Con (R) が接続されていません");
            return;
        }

        GUILayout.BeginHorizontal (GUILayout.Width (960));

        foreach (var joycon in m_joycons)
        {
            var isLeft = joycon.isLeft;
            var name = isLeft ? "Joy-Con (L)" : "Joy-Con (R)";
            var key = isLeft ? "Z キー" : "X キー";
            var button = isLeft ? m_pressedButtonL : m_pressedButtonR;
            var stick = joycon.GetStick ();
            var gyro = joycon.GetGyro ();
            var accel = joycon.GetAccel ();
            var orientation = joycon.GetVector ();

            GUILayout.BeginVertical (GUILayout.Width (480));
            GUILayout.Label (name);
            GUILayout.Label (key + "：振動");
            GUILayout.Label ("押されているボタン：" + button);
            GUILayout.Label (string.Format ("スティック：({0}, {1})", stick[0], stick[1]));
            GUILayout.Label ("ジャイロ：" + gyro);
            GUILayout.Label ("加速度：" + accel);
            GUILayout.Label ("傾き：" + orientation);
            GUILayout.EndVertical ();
        }

        GUILayout.EndHorizontal ();
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

<br>

---

相違箇所の解説をします。個々のコード解説は冒頭の参考記事に詳しく記されていますのでご覧になってください。
知識が浅きこと水たまりの如しなので有識者の方コメントいただければ幸いです。

- Joy-Conは `Start()`  の  `m_joycons = JoyconManager.Instance.j;` でインスタンス化しています。
- が、Joy-Conの接続にちょびっと時間がかかるため  `Start()`  の時点ではmac自体に（？）Unityに（？）接続されておらず、JoyconManagerからまだインスタンス化できないみたいなんですよね。
- なので、 `Update()` 時にも  `m_joycons`  が空なら再度インスタンス化して、改めてprivate変数にも突っ込もう作戦をとりました。
  - そんな思いを込めたハンドメイドなメソッドが末尾の`SetControllers()`くんです。
- privateなボタンがnullだと  `Update()`  の `if (m_joyconL.GetButton(button))`  が無限にエラー吐きます。

以上！

あともう一つ、下記のようなエラーも発生していました。どうやら実行環境依存みたいです。

`FormatException: Input string was not in a correct format.`

File > BuildSettings > ウィンドウ左下のPlayerSettings… > InspectorのOtherSettings > 中部の Configrationのうち、Scripting Runtime Versionで設定できます。
.NET4.0Equivalentで発生し、3.5に下げたら解消しました。ご参考までに。

---

<br>

ちょっと逸れましたが続きです。

Joy-Conの入力を確かめるために、空のオブジェクトを作成しましょう。

Hierarchy上でCreateEmptyします。名前はすごくなんでもいいのでExampleとかつけておいてください。

できたら、それをInspectorに表示し、JoyConSampleスクリプトとJoyconManagerスクリプトをくっつけましょう。ドラッグ&ドロップです。

![object-scripts](/images/blog/2019-04-03-unity-joycon-introduce/object-scripts.png)

<br>

## 完成

![output](/images/blog/2019-04-03-unity-joycon-introduce/output.png)

なんと完成。あとは再生ボタンを押せば、JoyConSample内が画面出力までやってくれます。
お手軽〜！

無限の可能性を感じますね。今後模索していきます。
