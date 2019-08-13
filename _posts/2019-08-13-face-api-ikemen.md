---
layout: blog
title: Microsoft AzureのFace APIで生田斗真とはらぱんを識別してみる
category: blog
tags: [AI, Azure, 顔認識, 顔検出]
summary: 前回はAmazon Rekognitionを使ってアミバとトキを識別出来るか試しました。S3縛りのところとかめんどくさそうだったので、次の選択肢であるFace APIを試してみようと思います。
author: aharada
image: /images/blog/2019-08-13-face-api-ikemen/wantedly.jpeg
---

前回はAmazon Rekognitionを使ってアミバとトキを識別出来るか試しました。S3縛りのところとかめんどくさそうだったので、次の選択肢であるFace APIを試してみようと思います。

[Amazon Rekognitionでアミバとトキの違いを認識出来るか](/blog/amazon-rekognition.html)

ここから開くとお試しスタート出来るっぽい？

既にAzureのアカウントは持っていたので、エンドポイントとSubscriptionKeyが発行された模様。

[Cognitive Services 試用エクスペリエンス](https://azure.microsoft.com/ja-jp/try/cognitive-services/my-apis/?api=face-api)

![APIキー](/images/blog/2019-08-13-face-api-ikemen/api.png)

## 顔検出

まずは簡単そうな顔検出をしてみる。クイックスタートのドキュメントを見ると、C#, Java, Node.js, Python, Goのドキュメントがあった。一番よく使うのPythonを選択。ていうかcurlのサンプルはないのかよ。

[クイック スタート:Azure REST API と Python を使用して画像から顔を検出する - Azure Cognitive Services](https://docs.microsoft.com/ja-jp/azure/cognitive-services/face/QuickStarts/Python)

mofmof社のサイトに載ってるぼくの写真を使う。

![はらぱん](/images/blog/2019-08-13-face-api-ikemen/contact-harada.png)

```
import requests
import json

subscription_key = '<SUBSCRIPTION_KEY>'
assert subscription_key

face_api_url = 'https://westcentralus.api.cognitive.microsoft.com/face/v1.0/detect'

image_url = 'https://d33wubrfki0l68.cloudfront.net/eb371b93cbeebcd5fd0232e2d52c1ab7ff75399b/b683c/images/contact/contact_harada.png'

headers = {'Ocp-Apim-Subscription-Key': subscription_key}

params = {
    'returnFaceId': 'true',
    'returnFaceLandmarks': 'false',
    'returnFaceAttributes': 'age,gender,headPose,smile,facialHair,glasses,emotion,hair,makeup,occlusion,accessories,blur,exposure,noise',
}

response = requests.post(face_api_url, params=params,
                         headers=headers, json={"url": image_url})
print(json.dumps(response.json()))
```

なんかJSONが返ってきたぞ。

32歳、いい線いってるな。

がっつり笑顔の写真なので、emotionは`happiness: 1`で、faceAttributesは`smile: 1`となっていてちゃんと表情が読み取れている。

`makeup`で化粧しているかどうか取得できるのも面白い。


```
[
	{
		"faceId": "00b39527-23d8-449b-9c3e-d08664999098",
		"faceRectangle": {
			"top": 63,
			"left": 150,
			"width": 80,
			"height": 80
		},
		"faceAttributes": {
			"smile": 1,
			"headPose": {
				"pitch": -2.2,
				"roll": 2.7,
				"yaw": 3.9
			},
			"gender": "male",
			"age": 32,
			"facialHair": {
				"moustache": 0.1,
				"beard": 0.1,
				"sideburns": 0.1
			},
			"glasses": "NoGlasses",
			"emotion": {
				"anger": 0,
				"contempt": 0,
				"disgust": 0,
				"fear": 0,
				"happiness": 1,
				"neutral": 0,
				"sadness": 0,
				"surprise": 0
			},
			"blur": {
				"blurLevel": "low",
				"value": 0.01
			},
			"exposure": {
				"exposureLevel": "goodExposure",
				"value": 0.64
			},
			"noise": {
				"noiseLevel": "low",
				"value": 0
			},
			"makeup": {
				"eyeMakeup": false,
				"lipMakeup": false
			},
			"accessories": [],
			"occlusion": {
				"foreheadOccluded": false,
				"eyeOccluded": false,
				"mouthOccluded": false
			},
			"hair": {
				"bald": 0.18,
				"invisible": false,
				"hairColor": [
					{
						"color": "black",
						"confidence": 1
					},
					{
						"color": "brown",
						"confidence": 0.94
					},
					{
						"color": "gray",
						"confidence": 0.31
					},
					{
						"color": "other",
						"confidence": 0.23
					},
					{
						"color": "blond",
						"confidence": 0.05
					},
					{
						"color": "red",
						"confidence": 0.03
					}
				]
			}
		}
	}
]
```

## 顔認識

続いて、事前に顔を覚えさせておいて、アップされた写真と照合して判別する顔認識をやってみます。

カイジに登場する黒服の識別をやろうとしたのだけど、何度やっても`{"error":{"code":"InvalidImage","message":"No face detected in the image."}}`というように、顔として認識してくれなかった。顔検出の方ではマンガの絵でも認識されているケースもあったので残念。。

仕方がないので、生田斗真とはらぱん(ぼく)を識別出来るか試してみる。

MSの公式ドキュメントは相変わらず読みにくい。こちらのエントリの方が分かりやすかった。

[Azure Face APIで顔を識別する手順をcurlで追う – hrendoh’s tech memo](https://blog.hrendoh.com/face-identification-with-azure-face-api/)

まずはPersonGroupというものを作ります。ikemenという名前にしました。

```
curl -v -X PUT \
  -H 'Content-Type:application/json' \
  -H 'Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>' \
  -d '{"name":"ikemen"}' \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen
```

PersonGroupにPersonを登録していきます。まずはharapanことぼくです。レスポンスにpersonIdというものが返ってきますがあとで使います。

```
curl -v -X POST \
  -H 'Content-Type:application/json' \
  -H 'Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>' \
  -d "{'name': 'harapan'}" \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/persons

 {"personId":"eb72a892-45cb-4380-9416-2acb09130039"}
```

続いて生田斗真をtomaとして登録します。

```
curl -v -X POST \
  -H 'Content-Type:application/json' \
  -H 'Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>' \
  -d "{'name': 'toma'}" \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/persons

{"personId":"6de75efe-8114-4093-b7f2-4bd711aa369b"}
```

生田斗真の顔を覚えさせます。

![生田斗真1](/images/blog/2019-08-13-face-api-ikemen/toma1.jpg)

![生田斗真2](/images/blog/2019-08-13-face-api-ikemen/toma2.jpg)

```
curl -v -X POST \
  -H 'Content-Type:application/octet-stream' \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  --data-binary @/Users/atsushiharada/Desktop/toma1.jpg \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/persons/6de75efe-8114-4093-b7f2-4bd711aa369b/persistedFaces
```

```
curl -v -X POST \
  -H 'Content-Type:application/octet-stream' \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  --data-binary @/Users/atsushiharada/Desktop/toma2.jpg \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/persons/6de75efe-8114-4093-b7f2-4bd711aa369b/persistedFaces
```

続いてはらぱんの顔を覚えさせます。実質ほぼ生田斗真ですが、果たしてFaceAPIはちゃんと認識出来るのでしょうか？

![はらぱん1](/images/blog/2019-08-13-face-api-ikemen/harapan1.jpg)
![はらぱん2](/images/blog/2019-08-13-face-api-ikemen/harapan2.jpg)


```
curl -v -X POST \
  -H 'Content-Type:application/octet-stream' \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  --data-binary @/Users/atsushiharada/Desktop/harapan1.jpg \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/persons/eb72a892-45cb-4380-9416-2acb09130039/persistedFaces
```

```
curl -v -X POST \
  -H 'Content-Type:application/octet-stream' \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  --data-binary @/Users/atsushiharada/Desktop/harapan2.jpg \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/persons/eb72a892-45cb-4380-9416-2acb09130039/persistedFaces
```

アップした画像を学習させます。202が返って来れば成功。

```
curl -v -X POST \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  -d "" \
  https:// https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/train
```

学習は非同期で実行されるらしいので、学習状況の確認はリクエストして確認出来る。

```
curl -v -X GET \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/persongroups/ikemen/training

{"status":"succeeded","createdDateTime":"2019-08-13T05:59:24.9316627Z","lastActionDateTime":"2019-08-13T05:59:25.1769198Z","message":null}
```

それでは早速顔認識させてみたいと思います。一度顔検出してfaceIdをとって来る必要があります。めんどくせ。

Wantedlyの募集記事のカバー画像に使っている弊社スタッフが3人写った写真を使います。leftの位置からすると`a656b0dd-f47c-4256-98f1-33bb8ddefacf`がはらぱんです。

![wantedly](/images/blog/2019-08-13-face-api-ikemen/wantedly.jpeg)

``` 
curl -v -X POST \
  -H 'Content-Type:application/octet-stream' \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  --data-binary @/Users/atsushiharada/Desktop/wantedly.jpeg \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/detect

  [
	{
		"faceId": "48618cb2-0939-4548-8ef0-98d2397ba484",
		"faceRectangle": {
			"top": 105,
			"left": 295,
			"width": 179,
			"height": 179
		}
	},
	{
		"faceId": "a656b0dd-f47c-4256-98f1-33bb8ddefacf",
		"faceRectangle": {
			"top": 139,
			"left": 1326,
			"width": 178,
			"height": 178
		}
	},
	{
		"faceId": "ce468758-4631-4c8d-975d-c8337cf9fef9",
		"faceRectangle": {
			"top": 261,
			"left": 885,
			"width": 149,
			"height": 149
		}
	}
]
```

いよいよ顔認識させてみます。faceIdsには上記で取得したfaceIdの配列を渡します。`a656b0dd-f47c-4256-98f1-33bb8ddefacf`の`personId`が`eb72a892-45cb-4380-9416-2acb09130039`で`confidence`が`0.80172`ということで、はらぱんであることを識別できているようですね。

```
curl -v -X POST \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  -d '{"personGroupId":"ikemen", "faceIds":["48618cb2-0939-4548-8ef0-98d2397ba484","a656b0dd-f47c-4256-98f1-33bb8ddefacf","ce468758-4631-4c8d-975d-c8337cf9fef9"]}' \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/identify

[
	{
		"faceId": "48618cb2-0939-4548-8ef0-98d2397ba484",
		"candidates": []
	},
	{
		"faceId": "a656b0dd-f47c-4256-98f1-33bb8ddefacf",
		"candidates": [
			{
				"personId": "eb72a892-45cb-4380-9416-2acb09130039",
				"confidence": 0.80172
			}
		]
	},
	{
		"faceId": "ce468758-4631-4c8d-975d-c8337cf9fef9",
		"candidates": []
	}
]
```

続いて生田斗真。Googleで検索して適当に入れてみるも、画像が小さいんだよオラって怒られまくる。

```
{"error":{"code":"InvalidImageSize","message":"Image size is too small."}}%
```

どうにか大きめの画像をみつけてきて若い頃の生田斗真を認識させてみる。同様にまずはfaceIdを取得するために顔検出する。

![若い頃の生田斗真](/images/blog/2019-08-13-face-api-ikemen/toma-young.jpeg)

```
curl -v -X POST \
  -H 'Content-Type:application/octet-stream' \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  --data-binary @/Users/atsushiharada/Desktop/toma_young.jpeg \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/detect

[
	{
		"faceId": "b01e403f-35f8-4941-996f-8de8884fa33d",
		"faceRectangle": {
			"top": 53,
			"left": 34,
			"width": 76,
			"height": 76
		}
	}
]
```

認識させる。`personId: 6de75efe-8114-4093-b7f2-4bd711aa369b, confidence: 0.6171`でかろうじて生田斗真と認識されたようだ。あやうくはらぱんと認識されてもおかしくないところだった。

```
curl -v -X POST \
  -H "Content-Type: application/json" \
  -H "Ocp-Apim-Subscription-Key:<SUBSCRIPTION_KEY>" \
  -d '{"personGroupId":"ikemen", "faceIds":["b01e403f-35f8-4941-996f-8de8884fa33d"]}' \
  https://westcentralus.api.cognitive.microsoft.com/face/v1.0/identify


[
	{
		"faceId": "b01e403f-35f8-4941-996f-8de8884fa33d",
		"candidates": [
			{
				"personId": "6de75efe-8114-4093-b7f2-4bd711aa369b",
				"confidence": 0.6171
			}
		]
	}
]
```


## まとめ

マンガの顔が認識されなかったのが残念なのと、顔認識する前に一度顔検出してfaceIdを取得しなければならないところがちょっとだるいなーと思った。

Amazon Rekognitionの方はいちいちS3を経由しなきゃいけないところが大変だるいが、マンガの顔も認識されていたしこっちを使った方がいいのだろうか。