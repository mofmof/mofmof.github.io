---
layout: blog
title: iOSでARやる 入門から簡単なアプリ作成まで
category: blog
tags: [Swift, iOS, AR, ARKit, SceneKit]
summary: ホットなxRの中でも、ARはスマホさえあれば簡単に体験できるカジュアルさが最高です。カメラを通してちょっと違った世界を覗けるこのエキサイティングな技術に触れてみませんか。
author: iwai
image: /images/blog/2019-07-22-ar-introduction/ar-image.png
---

mofmofの新規事業立ち上げ第二弾で、SwiftでARを扱ったのでAR入門＆ちょびっとだけ得た知見を書き殴ろうかと思います。
 

## tl;dr
- 特徴点検出
- ARKitとSceneKitがざっくり動く
- 北にオブジェクト置く

まっさらな状態からここまでやります


## 環境

新しめのmac
新しめのiPhone
新しめのXcode

これを書いてる私はmojave、iOS12、Xcode10みたいな感じでやりました。
対応機種はこんな感じ。公式スクショしてきました。

![](https://paper-attachments.dropbox.com/s_608E106655241054B7B14C820904B246C117D529E2AAC74796682A01ACCEF9A9_1563686104285_+2019-07-21+14.13.57.png)


詳しくはこちら(2019/07/26) https://www.apple.com/jp/ios/augmented-reality/


## 手を動かす前に！ ~15秒座学~

今回主に触るものの概要説明です。

- ARKit
    - カメラを通して特徴点を検出、それを基準に空間トラッキングしてくれる
    - iPhoneの各種センサを用いて自己位置の推定もしている
- SceneKit
    - 3Dオブジェクトを扱うフレームワーク
    - オブジェクトの位置とか回転とか、物理演算担当
    - ARKitとこいつとで現実世界にオブジェクトがあるような表現を頑張っている


## プロジェクト準備

### プロジェクト作成編

1. Xcodeを立ち上げて、Create a new projectしましょう
2. Single View Appでnext
3. 適当にプロジェクト名を入れます
4. 任意の場所でcreate

### 下準備の設定編

1. カメラの利用許可は必須なので、設定しておきます
    1. Info.plistを開きます
    2. 一番上に表示されている “Information Property List”にカーソルをのせると右端に出てくるプラスマークをクリックします
    3. Privacy - Camera Usage Descriptionを選択します
    4. Valueの欄に「AR機能を利用するためにカメラを使用します。」と書いておきます
2. なくても実機デバッグは可能ですが、Required device capabilitiesを設定しておきます
    1. Required device capabilitiesにカーソルをのせると表示されるプラスマークをクリック
    2. 追加されたItemのValueに”arkit”と入力
    3. 補足: Device Capbilityの役割とか https://developer.apple.com/library/archive/documentation/DeviceInformation/Reference/iOSDeviceCompatibility/DeviceCompatibilityMatrix/DeviceCompatibilityMatrix.html
3. (任意) 北にオブジェクトを置くときはiOSの位置情報サービスを利用するので、利用許可の設定をします
    1. 一番上に表示されている “Information Property List”にカーソルをのせると右端に出てくるプラスマークをクリックします
    2. Location When In Use Usage Descriptionを選択します
    3. Valueの欄に「北を検出するために位置情報を使用します。」と書いておきます

参考:

![](https://paper-attachments.dropbox.com/s_608E106655241054B7B14C820904B246C117D529E2AAC74796682A01ACCEF9A9_1563687018993_+2019-07-21+14.30.07.png)


準備OK！


## ARに触れていく編 特徴点検出まで

最初に、ARを表示するためのViewを設置します。
ARKit SceneKit Viewを置き、広げておきましょう。ちなみにSceneKitは3D用、SpriteKitは2D用という違いがあります。

![](https://paper-attachments.dropbox.com/s_608E106655241054B7B14C820904B246C117D529E2AAC74796682A01ACCEF9A9_1563687293455_+2019-07-21+14.34.05.png)


したらコードで触るために、なんやかんやいじっていきます。

まずは

- ARKitのimport
- 追加したARKit SceneKit ViewのOutlet接続

をします。

```
    import UIKit
    import ARKit // 追加
    
    class ViewController: UIViewController {
    
        @IBOutlet weak var arSceneView: ARSCNView! // control + ドラッグで追加
        
        override func viewDidLoad() {
            super.viewDidLoad()
        }
    
    }
```

ARKitはARSessionを通して諸々を制御するらしいので、arSceneViewにsessionを設定します。ARKitが特徴点を見つけてくれるところを眺めるためにデバッグオプションも追加しておきます。

```
    class ViewController: UIViewController {
    
        @IBOutlet weak var arSceneView: ARSCNView!
        
        override func viewDidLoad() {
            super.viewDidLoad()
    
            // 以下追加
            arSceneView.session = ARSession()
            
            arSceneView.showsStatistics = true
            arSceneView.debugOptions = ARSCNDebugOptions.showFeaturePoints
        }
    
    }
```

で、セッションはviewDidApperで走らせます。ついでにセッションを止めるのも書いておきます。

```
    class ViewController: UIViewController {
        override func viewDidLoad() {
            略
        }
        
        // 追加
        override func viewDidAppear(_ animated: Bool) {
            let configuration = ARWorldTrackingConfiguration()
            arSceneView.session.run(configuration)
        }
    
        // 追加
        override func viewWillDisappear(_ animated: Bool) {
            arSceneView.session.pause()
        }
    
    }
```

`ARWorldTrackingConfiguration`はARSessionのトラッキングオプションです。どんな情報を元にトラッキングするかということが設定できます。オプションはいくつかありますが、これがベーシックなものかと思います。

詳しくはこちら https://developer.apple.com/documentation/arkit/arconfiguration

ここまでやればとりあえず動くようにはなっていると思います。
実機繋いでRunしてみましょう。

よーし、次はオブジェクト設置ですね。



## ARに触れいていく編 オブジェクトの設置

画面をタップしたらオブジェクトが設置されるようにしていきます。

まずはTap Gesture Recognizerの追加。


![](https://paper-attachments.dropbox.com/s_608E106655241054B7B14C820904B246C117D529E2AAC74796682A01ACCEF9A9_1563716686387_+2019-07-21+22.28.13.png)



Actionで繋ぎます。

```
    @IBAction func handleTap(_ sender: Any) {
    }
```

中身はこんな感じにしましょう。

```
    @IBAction func handleTap(_ sender: Any) {
        guard let camera = arSceneView.pointOfView else { return } // 1
    
        let box = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0)
        let boxNode = SCNNode(geometry: box)
    
        let relativePosition = SCNVector3(x: 0, y: 0, z: -1) // 2
        boxNode.position = camera.convertPosition(relativePosition, to: nil) // 3
    
        arSceneView.scene.rootNode.addChildNode(boxNode)
    }
```

SCNBoxなSCNNodeを作って、ARSceneに追加しているところですね。

位置設定周りは、

1. カメラの座標を取得
2. z方向に-1、つまり前方に1mを表現したSCNVector3を作成
3. カメラの座標を、上記で準備した相対位置を加味しつつAR世界の絶対座標に変換

ということを行なっています。結果、カメラの前方1mの位置が指定できているわけです。
その位置にBoxのNodeを追加することでオブジェクトの設置を実装しました。

ここまでできたら動かしてみましょう。画面をタップした際に、豆腐みたいなオブジェクトが現れるはずです。


## ARに触れいていく編 北に置きたい

会社でダジャレをさらっと言うのが流行っているので勘弁してください。
最後のステップですね。豆腐が北に現れるようにしていきます。
今回の新規事業の肝でした。

カメラの位置と向き&位置情報サービスからなんとかします。

位置情報の取得下ごしらえ。

やることは三つあります。


1. 位置情報フレームワークを追加します

targets → generalの下部ですね。

![](https://paper-attachments.dropbox.com/s_608E106655241054B7B14C820904B246C117D529E2AAC74796682A01ACCEF9A9_1563719871498_+2019-07-21+23.36.56.png)



2. CoreLocationをimportします。
3. delegateを設定

```
    import UIKit
    import ARKit
    import CoreLocation // 追加
    
    class ViewController: UIViewController, CLLocationManagerDelegate { // 追加
        let locationManager = CLLocationManager()
```

ついでにlocationManagerのインスタンスを作っておきます。

準備OK。

動かす

```
    override func viewDidLoad() {
        super.viewDidLoad()
        
        arSceneView.session = ARSession()
        
        arSceneView.showsStatistics = true
        arSceneView.debugOptions = ARSCNDebugOptions.showFeaturePoints
        
        // 追加
        locationManager.delegate = self
        locationManager.startUpdatingHeading()
    }
```

viewDidLoadで方角取得を起こします。
そして、方角がupdateされるたびに呼ばれるメソッドはこちら。

```
    func locationManager(_ manager:CLLocationManager,didUpdateHeading newHeading:CLHeading)
```

ここで他の箇所にも色々と変更が入ります。全容はこちら。
変更箇所にはコメントを入れてあります。

```
    import UIKit
    import ARKit
    import CoreLocation
    
    class ViewController: UIViewController, CLLocationManagerDelegate {
        let locationManager = CLLocationManager()
        var isRotationInitialized = false // フラグを追加
        var northRotate: SCNVector4? // 北方向を保持する変数を追加
    
        @IBOutlet weak var arSceneView: ARSCNView!
        
        override func viewDidLoad() {
            super.viewDidLoad()
            
            arSceneView.session = ARSession()
            
            arSceneView.showsStatistics = true
            arSceneView.debugOptions = ARSCNDebugOptions.showFeaturePoints
            
            locationManager.delegate = self
            locationManager.startUpdatingHeading()
        }
        
        override func viewDidAppear(_ animated: Bool) {
            let configuration = ARWorldTrackingConfiguration()
            arSceneView.session.run(configuration)
        }
        
        override func viewWillDisappear(_ animated: Bool) {
            arSceneView.session.pause()
        }
        
        @IBAction func handleTap(_ sender: Any) {
            guard let camera = arSceneView.pointOfView else { return }
            guard let rotation = northRotate else { return } // northRotateの初期化を確認
    
            let box = SCNBox(width: 0.1, height: 0.1, length: 0.1, chamferRadius: 0)
            let boxNode = SCNNode(geometry: box)
    
            let relativePosition = SCNVector3(x: 0, y: 0, z: -1)
            camera.rotation = rotation // カメラの回転を北に向ける
            boxNode.position = camera.convertPosition(relativePosition, to: nil)
    
            arSceneView.scene.rootNode.addChildNode(boxNode)
        }
        
        func locationManager(_ manager:CLLocationManager,didUpdateHeading newHeading:CLHeading){
            let nowHeading = newHeading.magneticHeading // 現在向いている方向を0度~359度で取得
            
            if !isRotationInitialized {
                // (x軸, y軸, z軸, 回転)で回転を表現
                // 回転はラジアンで受け付けるため度数を変換している
                northRotate = SCNVector4(0, 1, 0, (nowHeading / 180) * Double.pi)
                isRotationInitialized = true
            }
        }
        
    }
```

微妙にややこしいです。

AR内の表現は右手座標系となっており、回転も右ねじです。なので、SCNVector4のy軸を1にすると、反時計回りの回転が表現できます。反時計周りに起動時の度数分回せば北を向くので、それを`func locationManager(_ manager:CLLocationManager,didUpdateHeading newHeading:CLHeading)`で計算させます。
起動時に真東を向いてたのなら、反時計回りに90度回せば北向くよね、ということです。

カメラの回転を表すrotationは、ARのセッションが続いている間は起動時を(0,0,0,0)とするので、その時点からどれくらい回せば北を向くかが保持できていれば今回の要件が達成できます。そんなこんなで上記のようなコードになります。

というわけで、画面をタップすると北に豆腐が現れるようになったと思います。完成！


## いかがでしたか。

考えることのベクトルがwebと違っていたり、3Dプログラミングならではな知識が必要だったりで、とてもエキサイティングです。
ARめちゃくちゃ楽しい。
