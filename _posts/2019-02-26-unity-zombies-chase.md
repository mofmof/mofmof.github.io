---
layout: blog
title: Unityで追いかけてくるゾンビを実装する
category: blog
tags: [Unity, Asset Store, ゾンビ, NavMesh]
summary: 最近めっきりUnityにハマっています。今日はゾンビに追いかけられる恐怖体験を実装してみたいと思います。
author: aharada
image: /images/blog/2019-02-26-unity-zombies-chase/stage.png
---

最近めっきりUnityにハマっています。今日はゾンビに追いかけられる恐怖体験を実装してみたいと思います。

前回のタイトル画面をコピーするところから始めます。

[Unityで画面遷移する - ゾンビの部屋にタイトル画面を実装](blog/unity-zombies-title.html)

Unityチュートリアルの[Survial Shooter Creating Enemy #1](https://unity3d.com/jp/learn/tutorials/projects/survival-shooter/enemy-one?playlist=17144)を参考にやっていきます。

## ゾンビ側の設定

前回のタイトル画面とは別に作りたいのでシーンをコピーしておきます。やり方は簡単なので割愛。こんな感じでゾンビが配置された状態からスタートです。

![ステージ](/images/blog/2019-02-26-unity-zombies-chase/stage.png)

どのゾンビでもいいので、`Add Component`していきます。

`Rigidbody`は、`Constraints`の`Freeze Position`のYのみチェック、`Freeze Rotation`のXとZにチェックを入れて転がったりしないようにします。

`Capsule Colider`は`Center`を`X:0, Y:1, Z:0`にする。

`Sphere Colider`は`Is Trigger`をチェック、`Center`を`X:0, Y:1, Z:0`にする。

`Nav Mesh Agent`は、`Speed:0.2, Radius:0.3, Height: 1.8`に変更。この`Nav Mesh Agent`というのでフィールドを学習して移動できる範囲を決めることが出来ます。詳しくはあんまり分かっていないけど。

![ゾンビの設定](/images/blog/2019-02-26-unity-zombies-chase/zombie.png)

## プレイヤーの配置

以前、[Unityで3Dオレの部屋を作って歩き回る](/blog/unity-my-room.html)でやったとおりにPlayerを配置し、同様に、`Main Camera`オブジェクトに`FollowPlayer`スクリプトを設定して、それぞれの`Rotation`で向きを揃える。今回は`Y:90`にしている。

## NavMeshでゾンビの移動を設定する

`Window -> AI -> Navigation`を開く。

`Bake`を開いて、`Agent Radius:0.75, Agent Height: 1.9, Step Height:0.38`を設定する。それぞれの数値はゾンビの大きさに合わせるっぽいがまだあんまり理解してない。

`Bake`ボタンを押す。これで学習してくれるっぽい？

![Navigation](/images/blog/2019-02-26-unity-zombies-chase/navigation.png)

続いて、NavMeshを使ってプレイヤーを追いかけるスクリプトを書きます。ゾンビに`New Script`を`Add Component`します。

EnemyMovement.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyMovement : MonoBehaviour
{
    Transform player;
    UnityEngine.AI.NavMeshAgent nav;

    void Awake()
    {
        player = GameObject.FindGameObjectWithTag("Player").transform;
        nav = GetComponent<UnityEngine.AI.NavMeshAgent>();
    }

    void Update()
    {
        nav.SetDestination(player.position);
    }
}
```

最後に全てのゾンビさんに反映させるために`Prefab`に`Apply All`します。

たったこれだけ。簡単！！

こんな感じで3体のゾンビさんが追いかけてきます。

![追いかけてくるゾンビ](/images/blog/2019-02-26-unity-zombies-chase/chase.gif)

## 残り

`Nav Mesh Agent`を設定するとなぜかゾンビさんが少しだけ浮遊してしまいます。`Navigation -> Bake`の`Step Height`あたりを疑っているんですがまだ未解決。

あと、ゾンビさん登場時に`idel`アニメーションが入っているのですが、問答無用にプレイヤーを追跡してくるので、起動時に歩いていないのに空中を滑るように移動してくる点も未解決。
