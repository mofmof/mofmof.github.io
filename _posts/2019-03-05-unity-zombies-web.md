---
layout: blog
title: Unityで作った湧き出るゾンビをWEBで公開してみる
category: blog
tags: [Unity, Asset Store, ゾンビ, WebGL]
summary: 今日は増殖するゾンビを実装します。前回、地形の生成と追いかけてくるゾンビを実装したので続きから行きますね。
author: aharada
image: /images/blog/2019-03-05-unity-zombies-web/zombies.png
---

今日は増殖するゾンビを実装します。前回、地形の生成と追いかけてくるゾンビを実装したので続きから行きますね。

- [UnityのTerrainでゲームっぽい地形を生成する](/blog/unity-zombies-chase-field.html)
- [Unityで追いかけてくるゾンビを実装する](/blog/unity-zombies-chase.html)

Unityのチュートリアルを参考にすれば簡単。

[Spawning Enemies - Unity](https://unity3d.com/jp/learn/tutorials/projects/survival-shooter/more-enemies?playlist=17144)

## ゾンビを生まれる場所とスクリプト

`Hierarchy -> Create Empty`してオブジェクト名を`ZombieSpawnPoint`にし、`Inspector`でキューブから赤い吹き出しに変更しておきます。

![赤い吹き出し](/images/blog/2019-03-05-unity-zombies-web/red.png)

`Position X:223.52, Y:164.78, Z:220.17`に設定。`Terrain`で作ったフィールドがでかすぎて良い感じの位置を探すのが大変。。

再び`Hierarchy -> Create Empty`してオブジェクト名を`EnemyManegement`にして、座標を`reset`します。

![座標をリセット](/images/blog/2019-03-05-unity-zombies-web/reset.png)

`Add Component`して、ゾンビを一定間隔で産み落として増殖させるスクリプトを書きます。

EnemyManager.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyManager : MonoBehaviour
{
    public GameObject enemy;
    public float spawnTime = 3f;
    public Transform[] spawnPoints;


    void Start ()
    {
        InvokeRepeating ("Spawn", spawnTime, spawnTime);
    }


    void Spawn ()
    {
        int spawnPointIndex = Random.Range (0, spawnPoints.Length);
        Instantiate (enemy, spawnPoints[spawnPointIndex].position, spawnPoints[spawnPointIndex].rotation);
    }
}
```


最初からいるゾンビを1体にして、`ZombieSpawnPopint`らへんに配置します。合わせて`Player`も近くに配置するとこんな感じになります。

![ゾンビとプレイヤーを配置](/images/blog/2019-03-05-unity-zombies-web/layout.png)

これで準備OK。

## WEBで公開
今日はUnityで作ったアプリをWEBで公開してみます。

`Build Settings -> Platform -> WebGL`を選択して`Switch Platform`します。

以下のような感じで、何かとエラーが出るので一旦`Oculus Integration`をごっそり削除するとビルド出来るようになります。

```
Assets/Oculus/LipSync/Scripts/OVRLipSyncMicInput.cs(104,13): error CS0103: The name 'Microphone' does not exist in the current context
```

ビルド出来たらhtmlファイルも吐かれるので好きなホスティングサービスにのっければ出来上がり！

湧き出るゾンビ体験が出来ます！

[Unity WebGL Player MyRoom](https://unity-app-publish.firebaseapp.com/)

![湧き出るゾンビ](/images/blog/2019-03-05-unity-zombies-web/zombies.png)
