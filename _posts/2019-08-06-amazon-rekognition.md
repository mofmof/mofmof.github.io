---
layout: blog
title: Amazon Rekognitionでアミバとトキの違いを認識出来るか
category: blog
tags: [Rekognition, AI, 顔認識]
summary: ちょっと作ってみたいプロダクトがあって、顔認識APIを調べてみたいと思った。ざっと調べて、Microsoft Azure Face API, Amazon Rekognitionのどちらかにしようと思った
author: aharada
image: /images/blog/2019-08-06-amazon-rekognition/ken-juuza.png
---

ちょっと作ってみたいプロダクトがあって、顔認識APIを調べてみたいと思った。

ざっと調べて、

- Microsoft Azure Face API
- Amazon Rekognition

のどちらかにしようと思ったが、Face APIは既に社内で遊んでた人がいて便利そうなのが分かっていたのでAmazon Rekognitionにした。

## AWSの画面からデモを動かしてみる

参考URL

[ステップ 4: Amazon Rekognition コンソールの使用開始 - Amazon Rekognition](https://docs.aws.amazon.com/ja_jp/rekognition/latest/dg/getting-started-console.html)

AWSのコンソールにサインインしときます。

Rekognitionで検索すると、とりあえずサクッとAWSの画面上から試せるデモがあるのでまずはこれを触ってみる。

![デモ](/images/blog/2019-08-06-amazon-rekognition/demo.png)

顔の比較面白そうなので試す。

[演習 3: イメージ内の顔を比較する (コンソール) - Amazon Rekognition](https://docs.aws.amazon.com/ja_jp/rekognition/latest/dg/compare-faces-console.html)

まずはケンシロウとジュウザを比較してみる。当然一致しないだろうと思ったら、しょっぱなから予想外の展開。

せつこ、それ顔やない拳やぞ。

![ケンシロウとジュウザ](/images/blog/2019-08-06-amazon-rekognition/ken-juuza.png)

仕方がないのでラオウとジュウザを比較。当然一致しませんね。

![ラオウとジュウザ](/images/blog/2019-08-06-amazon-rekognition/raou-juuza.png)

ここまでくれば賢い皆さんは分かっていると思います。お待ちかね、アミバとトキです。

楽勝でしたね。よくよくみるとアミバとトキって全然違う顔してますね。

![トキとアミバ](/images/blog/2019-08-06-amazon-rekognition/toki-amiba.png)

ここまで、不一致という結果しか見ていないので、一致するであろう比較も試す。問題ないですね。

![トキとトキ](/images/blog/2019-08-06-amazon-rekognition/toki-toki.png)

## Amazon Rekognition コレクション
さて、デモを動かしてどんなことが実現できるか分かりました。次は1:1の画像同士の比較ではなく、コレクションに登録した画像と、1つの画像を比べて、該当するものを見つける処理をやってみます。画像による検索みたいな感じですね。

参考URL

- [コレクションの作成 - Amazon Rekognition](https://docs.aws.amazon.com/ja_jp/rekognition/latest/dg/create-collection-procedure.html)
- [Amazon Rekognitionで「コレクション」を操作する ｜ DevelopersIO](https://dev.classmethod.jp/cloud/rekognition-handling-collection/)

まずはRekognitionコレクションなるものを作ります。

```
$ aws rekognition create-collection --collection-id "hokutono-ken"

aws:rekognition:ap-northeast-1:017826181293:collection/hokutono-ken	4.0	200
```

S3を経由する必要があるっぽいので、適当なバケット作ってさっき使用した画像を一通りアップします。

![バケット](/images/blog/2019-08-06-amazon-rekognition/bucket.png)

続いてS3にアップした画像をコレクションに追加してみます。アミバ。

```
$ aws rekognition index-faces --collection-id 'hokutono-ken' --image '{"S3Object":{"Bucket":"rekognition-sample-1","Name":"amiba1.png"}}' --external-image-id 'amiba'

4.0
FACE	98.46493530273438	amiba	aa844226-ab19-44d4-8794-b3fe2661b3e6	41a39206-e8a5-3275-83f9-0e39d7cf3c82
BOUNDINGBOX	0.5311169624328613	0.2021254599094391	0.30127328634262085	0.4585578441619873
FACEDETAIL	98.46493530273438
BOUNDINGBOX	0.5311169624328613	0.2021254599094391	0.30127328634262085	0.4585578441619873
LANDMARKS	eyeLeft	0.208028644323349	0.5075579881668091
LANDMARKS	eyeRight	0.33020567893981934	0.48216381669044495
LANDMARKS	mouthLeft	0.25633931159973145	0.6738607883453369
LANDMARKS	mouthRight	0.35594677925109863	0.6540347933769226
LANDMARKS	nose	0.2332862913608551	0.5932871699333191
POSE	6.496764183044434	-15.008339881896973	-38.05028533935547
QUALITY	90.58085632324219	99.54999542236328
```

次はトキ。

```
$ aws rekognition index-faces --collection-id 'hokutono-ken' --image '{"S3Object":{"Bucket":"rekognition-sample-1","Name":"toki1.jpg"}}' --external-image-id 'toki'
4.0
FACE	99.99978637695312	toki	002dc264-43d9-4f98-917d-6c51a7f771c4	9a2696fc-6186-3da1-9c1a-b8f389880584
BOUNDINGBOX	0.713051438331604	0.20599114894866943	0.13474750518798828	0.7362598776817322
FACEDETAIL	99.99978637695312
BOUNDINGBOX	0.713051438331604	0.20599114894866943	0.13474750518798828	0.7362598776817322
LANDMARKS	eyeLeft	0.3723050355911255	0.39738720655441284
LANDMARKS	eyeRight	0.6741913557052612	0.3866893947124481
LANDMARKS	mouthLeft	0.4313778579235077	0.6607327461242676
LANDMARKS	mouthRight	0.6803216338157654	0.6538835167884827
LANDMARKS	nose	0.4899027347564697	0.5459250211715698
POSE	-8.109090805053711	-5.07843017578125	-23.702438354492188
QUALITY	75.8443603515625	99.66374969482422
```

トキをもう一個。

```
$ aws rekognition index-faces --collection-id 'hokutono-ken' --image '{"S3Object":{"Bucket":"rekognition-sample-1","Name":"toki2.jpg"}}' --external-image-id 'toki'
4.0
FACE	99.99512481689453	toki	57ea2aba-d590-413c-93cd-4c5352f9d278	dd98dbad-bb98-32dc-89a1-b8032ccf85e7
BOUNDINGBOX	0.7523548007011414	0.3014705777168274	0.07518408447504044	0.36288216710090637
FACEDETAIL	99.99512481689453
BOUNDINGBOX	0.7523548007011414	0.3014705777168274	0.07518408447504044	0.36288216710090637
LANDMARKS	eyeLeft	0.3504350185394287	0.33039596676826477
LANDMARKS	eyeRight	0.4949299395084381	0.28910547494888306
LANDMARKS	mouthLeft	0.39206045866012573	0.6262251138687134
LANDMARKS	mouthRight	0.5097337365150452	0.5933699607849121
LANDMARKS	nose	0.398319274187088	0.4656740725040436
POSE	4.154842853546143	-15.775106430053711	-42.40241241455078
QUALITY	92.02243041992188	99.54999542236328
```

これでアミバとトキx2がコレクションに入りました。

``` 
$ aws rekognition list-faces --collection-id 'hokutono-ken'

4.0
FACES	99.99979400634766	toki	002dc264-43d9-4f98-917d-6c51a7f771c4	9a2696fc-6186-3da1-9c1a-b8f389880584
BOUNDINGBOX	0.7130510210990906	0.2059909999370575	0.13474799692630768	0.7362599968910217
FACES	99.99510192871094	toki	57ea2aba-d590-413c-93cd-4c5352f9d278	dd98dbad-bb98-32dc-89a1-b8032ccf85e7
BOUNDINGBOX	0.7523549795150757	0.3014709949493408	0.07518409937620163	0.36288198828697205
FACES	98.46489715576172	amiba	aa844226-ab19-44d4-8794-b3fe2661b3e6	41a39206-e8a5-3275-83f9-0e39d7cf3c82
BOUNDINGBOX	0.5311170220375061	0.20212499797344208	0.301272988319397	0.45855799317359924
```

新たに見つけてきたアミバの画像で検索してみます。

```
$ aws rekognition search-faces-by-image --collection-id hokutono-ken --image '{"S3Object":{"Bucket": "rekognition-sample-1","Name":"amiba2.jpg"}}'

4.0	98.81639099121094
FACEMATCHES	98.49234008789062
FACE	98.46489715576172	amiba	aa844226-ab19-44d4-8794-b3fe2661b3e6	41a39206-e8a5-3275-83f9-0e39d7cf3c82
BOUNDINGBOX	0.5311170220375061	0.20212499797344208	0.301272988319397	0.45855799317359924
SEARCHEDFACEBOUNDINGBOX	0.454548180103302	0.35454151034355164	0.2800408899784088	0.29142048954963684
```

`amiba`がヒットしてるっぽいけど、うーん、なんか読みにくいなあ。ドキュメントによるとJSONが返るはずなのになー。

ふと思い出しました。`aws configure`したときに`text`を指定してしまっていました。JSONに変更します。

```
$ aws configure
AWS Access Key ID [****************aaaa]:
AWS Secret Access Key [****************aaaa]:
Default region name [ap-northeast-1]:
Default output format [text]: json
```

これで結果がJSONになりました。前の方の出力もJSONで見た方がわかりやすいっす。

```
$ aws rekognition search-faces-by-image --collection-id hokutono-ken --image '{"S3Object":{"Bucket": "rekognition-sample-1","Name":"amiba2.jpg"}}'

{
    "SearchedFaceBoundingBox": {
        "Width": 0.29142048954963684,
        "Height": 0.454548180103302,
        "Left": 0.35454151034355164,
        "Top": 0.2800408899784088
    },
    "SearchedFaceConfidence": 98.81639099121094,
    "FaceMatches": [
        {
            "Similarity": 98.49234008789062,
            "Face": {
                "FaceId": "aa844226-ab19-44d4-8794-b3fe2661b3e6",
                "BoundingBox": {
                    "Width": 0.45855799317359924,
                    "Height": 0.5311170220375061,
                    "Left": 0.20212499797344208,
                    "Top": 0.301272988319397
                },
                "ImageId": "41a39206-e8a5-3275-83f9-0e39d7cf3c82",
                "ExternalImageId": "amiba",
                "Confidence": 98.46489715576172
            }
        }
    ],
    "FaceModelVersion": "4.0"
}
```

これで、`Similarity": 98.49234008789062`で`"ExternalImageId": "amiba"`が検索されているのがわかりますね.

これで大体の使い方が分かりました！S3を経由しなきゃいけないのが少々めんどくさいですねえ。