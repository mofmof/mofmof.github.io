---
layout: blog
title: Unityで画面遷移する - ゾンビの部屋にタイトル画面を実装
category: blog
tags: [Unity, Asset Store, ゾンビ]
summary: 先日ゾンビ部屋を作ったので、タイトル画面をつけてUnityで画面遷移してみたいと思います。
author: aharada
image: /images/blog/2019-02-19-unity-zombies-title/title.png
---

先日ゾンビ部屋を作ったので、タイトル画面をつけてUnityで画面遷移してみたいと思います。

[3Dオレの部屋をVR対応してゾンビさんと対面する](/blog/unity-my-room-for-oculus-go.htm)

## タイトル画面の背景

やりたいことは画面遷移なので、タイトル画面はなんでもいいんだけど、せっかくだから雰囲気出すために町にゾンビがウヨウヨしてる感じを表現してみる。

それっぽい家のAssetsを見つけたのでこれを使います。

[UK Terraced Houses Pack FREE - Asset Store](https://assetstore.unity.com/packages/3d/environments/urban/uk-terraced-houses-pack-free-63481)

例によってダウンロード/インポートしておく。

`Assets -> UK Terraced Houses FREE -> Prefabs`の下にある家を適当に画面に配置していきます。

ちなみに`Inspector`の下の黒いバーを上にドラッグすると、選択している`Prefab`のビジュアルが見えるので便利。

![Prefab Before](/images/blog/2019-02-19-unity-zombies-title/prefab-before.png)

![Prefab After](/images/blog/2019-02-19-unity-zombies-title/prefab-after.png)

こんな感じに配置。

![背景を配置](/images/blog/2019-02-19-unity-zombies-title/background.png)


正面に並んだ家が見えるように`Main Camera`を移動する。

```
Position X:-16, Y:2, Z:-12
Rotation X:0, Y:90, Z:0
```

## ゾンビさん達を配置

続いて`Assets -> Prefabs -> Zombie`から`Hierarchy`にドラッグしてゾンビを3体適当に配置。どこでもいいんだけど、全員同じ方向に歩いてると不自然なので、それぞれ`Rotation`のYをバラバラにした。

3体目はカメラより後ろに配置して、後ろからでてくるようにする。

![ゾンビを配置](/images/blog/2019-02-19-unity-zombies-title/zombies.png)

前回と同じフリーのゾンビさんを使ってるので、ちょっとアニメーションを調整します。

`walk_in_place`を外して、最初からちゃんと歩くようにします。また、`GoWalk`というBool値のパラメータを定義しておく。

![アニメーション](/images/blog/2019-02-19-unity-zombies-title/animation.png)


`Hierarchy -> Zombie`に`New Script`を`Add Component`します。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ZombieController : MonoBehaviour
{
    Animator animator;
    float time = 0f;  // 単位: 秒
    public float thresholdTime = 5f;

    void Start () {
		animator = GetComponent<Animator>();
	  }

    void FixedUpdate()
    {
        time += Time.deltaTime;
        if (time > thresholdTime) {
            animator.SetBool("GoWalk", true);
        }
    }
}
```

`Threshold Time`を`Inspector`から上書きできるようになるので、3体のゾンビさんにバラバラの値を設定しておきます。

![Threshold Time](/images/blog/2019-02-19-unity-zombies-title/animation.png)

この数値はゾンビが歩き出すまでの時間を制御していて、全てのゾンビがピッタリ合わせて歩くのを防いでいます。もうちょっと良いやり方あるかも知れないけど分からんので一旦これで。

そういえば、よく見かける`deltaTime`ってなんだろうなって思ったら、下記エントリに解説が載ってた。

[Unity - 一定時間で消えるオブジェクトをつくる - おれ、Unity2Dでゲーム作るんだ。](http://unity2d.hateblo.jp/entry/2015/03/24/014409)

## 画面遷移するボタン
クリックしたら画面遷移するボタンを配置します。`Hierarchy -> Create -> UI -> Button`すると、`Canvas`,`Button`,`EventSystem`などが設置される。

`Button`がよく分からん場所に配置されてたりするので、`Pos X, Pos Y, Pos Z`の座標を全て0に設定しておく。

続いて`Button`に`New Script`を`Add Component`する。

```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.SceneManagement;

public class ButtonHandler : MonoBehaviour
{
    // Start is called before the first frame update
    void Start()
    {
    }

    // Update is called once per frame
    void Update()
    {

    }

    public void OnClick() {
        Debug.Log("Button click!");
        SceneManager.LoadScene ("Room");
    }
}
```

`Button -> Inspector -> Button(Script) -> On Click()`を設定する。

![On Click](/images/blog/2019-02-19-unity-zombies-title/onclick.png)

よっしゃこれでいけるだろ！とやってみたらエラーでた。

```
Scene 'Room' couldn't be loaded because it has not been added to the build settings or the AssetBundle has not been loaded.
To add a scene to the build settings use the menu File->Build Settings...
UnityEngine.SceneManagement.SceneManager:LoadScene(String)
ButtonHandler:OnClick() (at Assets/ButtonHandler.cs:21)
UnityEngine.EventSystems.EventSystem:Update()
```

`Room`なんて`Scene`はねえよボケってことらしいので、`Room`の`Scene`を開いている状態で`File -> Build Settings -> Scenes In Build -> Add Open Scenes`しておく。これでOK。

![シーンを追加](/images/blog/2019-02-19-unity-zombies-title/add-scene.png)

試してみたら無事画面遷移出来ました！

![タイトル画面](/images/blog/2019-02-19-unity-zombies-title/title.png)

![遷移後](/images/blog/2019-02-19-unity-zombies-title/transition.png)

## BGMを設定
せっかくなので雰囲気が出るようにBGMを設定します。いつもお世話になっているフリー素材屋さんへ。

[フリーBGM素材 『不気味な部屋』 試聴ページ｜フリーBGM DOVA-SYNDROME](https://dova-s.jp/bgm/play10480.html)

ダウンロードしたmp3ファイルを`Project -> Audios`フォルダにドラッグしておき、`Hierarchy -> Create -> Audio -> Audio Source`で追加したファイルを選択する。

やりました！不気味なBGMが流れます！
