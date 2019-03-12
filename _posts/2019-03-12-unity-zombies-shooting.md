---
layout: blog
title: Unityで増殖するゾンビを撃って戦うゲームみたいの作る
category: blog
tags: [Unity, Asset Store, ゾンビ]
summary: 前回WebGLにSwitch PlatformしてしまったのでPC, Mac & Linux Standaloneに戻してから始めます。
author: aharada
image: /images/blog/2019-03-12-unity-zombies-shooting/set-zombie.png
---

前回`WebGL`に`Switch Platform`してしまったので`PC, Mac & Linux Standalone`に戻してから始めます。

[Unityで作った湧き出るゾンビをWEBで公開してみる - もふもふ技術部](/blog/unity-zombies-web.html)

## 事前準備

`Terrain`が広大すぎて配置が大変なので、高低差が小さく、もっと狭いフィールドを作ります。

以前に`Terrain`を使ったときのエントリ。

[UnityのTerrainでゲームっぽい地形を生成する - もふもふ技術部](/blog/unity-zombies-chase-field.html)

Inspectorの歯車マークを開いてサイズを指定することが出来る。

![Terrain](/images/blog/2019-03-12-unity-zombies-shooting/terrain.png)

こんな感じのフィールドが出来る。

![フィールド生成](/images/blog/2019-03-12-unity-zombies-shooting/field.png)

前回作ったゾンビを改めて配置して事前準備OK。

![ゾンビを配置](/images/blog/2019-03-12-unity-zombies-shooting/set-zombie.png)

## プレイヤーが銃を撃つ

Unity公式チュートリアルのSurvival Shooterの実装を参考にします。

[Harming Enemies - Unity](https://unity3d.com/jp/learn/tutorials/projects/survival-shooter/harming-enemies?playlist=17144)

まずはPlayerが銃を撃ったときのビジュアルを作っていきます。

HierarchyからPlayerを選択してCreate Empty ChildしてGunという名前をつけて、Yを0.8くらいにしておく。

![Gun](/images/blog/2019-03-12-unity-zombies-shooting/gun.png)

Line Rendererというコンポーネントを使って、弾道を表現する。MaterialsにDefault-Lineを設定、PositionsのWidthは0.015を設定、ほかはデフォルトでOK。

![Line Renderer](/images/blog/2019-03-12-unity-zombies-shooting/line.png)

続いて、Lightコンポーネントで撃ったときのマズルフラッシュ(光るやつ)を表現する。Colorに黄色を選択し、他はデフォルト。

![Light](/images/blog/2019-03-12-unity-zombies-shooting/light.png)

Audio Sourceコンポーネントで撃ったときの音を設定する。Play On Awakeはチェックをはずす。

Asset Storeから銃の音が入ってるAssetをダウンロード/インポートし、AudioClipにimpacterというサウンドを設定する。

[Fog of War Gun Sound FX Free - Asset Store](https://assetstore.unity.com/packages/audio/sound-fx/weapons/fog-of-war-gun-sound-fx-free-66100)

![impacter](/images/blog/2019-03-12-unity-zombies-shooting/impacter.png)

SciprtをAdd Componentする。

PlayerShooting.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class PlayerShooting : MonoBehaviour
{
    public int damagePerShot = 20;
    public float timeBetweenBullets = 0.15f;
    public float range = 100f;


    float timer;
    Ray shootRay = new Ray();
    RaycastHit shootHit;
    int shootableMask;
    ParticleSystem gunParticles;
    LineRenderer gunLine;
    AudioSource gunAudio;
    Light gunLight;
    float effectsDisplayTime = 0.2f;


    void Awake ()
    {
        shootableMask = LayerMask.GetMask ("Shootable");
        // gunParticles = GetComponent<ParticleSystem> ();
        gunLine = GetComponent <LineRenderer> ();
        gunAudio = GetComponent<AudioSource> ();
        gunLight = GetComponent<Light> ();
    }


    void Update ()
    {
        timer += Time.deltaTime;

if(Input.GetButton ("Fire1") && timer >= timeBetweenBullets && Time.timeScale != 0)
        {
            Shoot ();
        }

        if(timer >= timeBetweenBullets * effectsDisplayTime)
        {
            DisableEffects ();
        }
    }


    public void DisableEffects ()
    {
        gunLine.enabled = false;
        gunLight.enabled = false;
    }


    void Shoot ()
    {
        timer = 0f;
        gunAudio.Play ();
        gunLight.enabled = true;

        // gunParticles.Stop ();
        // gunParticles.Play ();

        gunLine.enabled = true;
        gunLine.SetPosition (0, transform.position);

        shootRay.origin = transform.position;
        shootRay.direction = transform.forward;

        if(Physics.Raycast (shootRay, out shootHit, range, shootableMask))
        {
            EnemyHealth enemyHealth = shootHit.collider.GetComponent <EnemyHealth> ();
            if(enemyHealth != null)
            {
                enemyHealth.TakeDamage (damagePerShot, shootHit.point);
            }
            gunLine.SetPosition (1, shootHit.point);
        }
        else
        {
            gunLine.SetPosition (1, shootRay.origin + shootRay.direction * range);
        }
    }
}
```

これで銃が撃てるようになるぞ！

## ゾンビに当たるようにする
銃らしきものを撃てるようになったので、今度はゾンビさんに当たって倒せるようにします。

PlayerShooting.csで`shootableMask = LayerMask.GetMask ("Shootable");`を使っている箇所があり、これは銃が当たるレイヤーを指定してます。なのでゾンビさんをShootableというレイヤーに設定する必要がある。

![Shootable Layer](/images/blog/2019-03-12-unity-zombies-shooting/shootable.png)

続いて、EnemyHealthというScriptをAdd Componentします。これはゾンビさんのHPを管理し、HPがなくなったらオブジェクトを破棄するなどの処理が記述されてます。

EnemyHealth.cs

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class EnemyHealth : MonoBehaviour
{
    public int startingHealth = 100;
    public int currentHealth;
    public float sinkSpeed = 2.5f;
    public int scoreValue = 10;
    public AudioClip deathClip;


    Animator anim;
    AudioSource enemyAudio;
    ParticleSystem hitParticles;
    CapsuleCollider capsuleCollider;
    bool isDead;
    bool isSinking;


    void Awake ()
    {
        anim = GetComponent <Animator> ();
        enemyAudio = GetComponent <AudioSource> ();
    //     hitParticles = GetComponentInChildren <ParticleSystem> ();
        capsuleCollider = GetComponent <CapsuleCollider> ();
        currentHealth = startingHealth;
    }


    void Update ()
    {
        if(isSinking)
        {
            transform.Translate (-Vector3.up * sinkSpeed * Time.deltaTime);
        }
    }


    public void TakeDamage (int amount, Vector3 hitPoint)
    {
        if(isDead)
            return;

        enemyAudio.Play ();

        currentHealth -= amount;

        // hitParticles.transform.position = hitPoint;
        // hitParticles.Play();

        if(currentHealth <= 0)
        {
            Death ();
        }
    }

    void Death ()
    {
        isDead = true;
        // capsuleCollider.isTrigger = true;
        anim.SetTrigger ("Dead");

        enemyAudio.clip = deathClip;
        enemyAudio.Play ();
    }


    public void StartSinking ()
    {
        GetComponent <UnityEngine.AI.NavMeshAgent> ().enabled = false;
        GetComponent <Rigidbody> ().isKinematic = true;
        isSinking = true;
        // ScoreManager.score += scoreValue;
        Destroy (gameObject, 2f);
    }
}
```

銃が当たったときのうめき声と、ゾンビさんが死んだときの断末魔を設定します。

怖い音声を集めたAssetが配布されているのでこれを使います。ホントUnityはなんでも揃っていて大変素晴らしい。しかも無料。

[Horror Sfx - Asset Store](https://assetstore.unity.com/packages/audio/sound-fx/horror-sfx-32834)

EnemyHealthのDeath ClipとAudio SourceにそれぞれZombie_04とZombie_03という音声を設定。

![ゾンビの音声を設定](/images/blog/2019-03-12-unity-zombies-shooting/audio-zombie.png)

これで銃を撃ったら当たるようになります！

## ゾンビが死んで消えるアニメーション
Asset Storeからインポートしたゾンビのデフォルトのアニメーションをこんな感じに編集する。

![アニメーション](/images/blog/2019-03-12-unity-zombies-shooting/animation.png)

walk -> fallingbackの遷移を、DeadパラメータのTriggerで遷移するように設定。

![アニメーションの遷移](/images/blog/2019-03-12-unity-zombies-shooting/transition.png)

最後にfallingbackアニメーション後に自動的に沈んで消えるアニメーションイベントを設定。

Project -> Zombie -> Animations -> Zombie@fallingbackのInspectorを開いて、Eventsを設定。StartSinkingが呼ばれるようにする。どういうわけでEnemyHealth.StartSinkingが呼ばれるのかはよく分からん。

![イベント](/images/blog/2019-03-12-unity-zombies-shooting/event.png)

以上でOK。

湧き出るゾンビを銃でバンバン倒していくゲームが出来ます！

![ゾンビを倒す](/images/blog/2019-03-12-unity-zombies-shooting/game.gif)
