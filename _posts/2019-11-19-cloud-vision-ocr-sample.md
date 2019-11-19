---
layout: blog
title: Google Cloud Visionを使ってPythonでOCRしてみる
category: blog
tags: [Cloud Vision, OCR, Python]
summary: 以前、Google Cloud Visionを使ってcurl経由でOCRを試した。今回はPythonで実装するとどんな感じになるのか試してみる。ひとまずVision APIの使い方のドキュメントを見直すところから。
author: aharada
image: 
---

以前、Google Cloud Visionを使ってcurl経由でOCRを試した。

[Google Cloud Vision APIでOCRを試してみる](/blog/cluod-vision-ocr.html)

今回はPythonで実装するとどんな感じになるのか試してみる。ひとまずVision APIの使い方のドキュメントを見直すところから。

[https://cloud.google.com/vision/docs/libraries?hl=ja#client-libraries-install-python](https://cloud.google.com/vision/docs/libraries?hl=ja#client-libraries-install-python)

```
$ mkdir python-cloud-vision-sample
$ cd python-cloud-vision-sample
$ pip install --upgrade google-cloud-vision
```

ドキュメントによるとサービスアカウントキーの作成せよとある。[前回](/blog/cluod-vision-ocr.html)つくったプロジェクトを使ってサービスアカウントを作成する。サービスアカウントは役割なしで良いみたい。

![サービスアカウントの作成](/images/blog/2019-11-19-cloud-vision-ocr-sample/service-account.png)

credentialファイルがダウンロードされるので、プロジェクト内に移動。

```
$ mv ~/Downloads/cloud-vision-sample-8019658d4eb3.json .
```

このcredentialファイルを`export`コマンドでパスを設定しろと書いてあるが、使い勝手悪そうだったのでソースコード内で指定するように変更した。今回はローカルのファイルからテキスト検出するサンプルを使ったけど、WEB上においてある画像を使うことも出来る。

ソースコードはこちらからコピペ。

[テキスト検出のサンプル Cloud Vision API ドキュメント Google Cloud](https://cloud.google.com/vision/docs/detecting-text?hl=ja)

main.py

```
import io
import os
from google.cloud import vision
from google.cloud.vision import types
from google.oauth2 import service_account

def detect_text(path):
    """Detects text in the file."""
    from google.cloud import vision
    # client = vision.ImageAnnotatorClient()
    credentials = service_account.Credentials.from_service_account_file('cloud-vision-sample-8019658d4eb3.json')
    client = vision.ImageAnnotatorClient(credentials=credentials)

    with io.open(path, 'rb') as image_file:
        content = image_file.read()

    image = vision.types.Image(content=content)

    response = client.text_detection(image=image)
    texts = response.text_annotations
    print('Texts:')

    for text in texts:
        print('\n"{}"'.format(text.description))

        vertices = (['({},{})'.format(vertex.x, vertex.y)
                    for vertex in text.bounding_poly.vertices])

        print('bounds: {}'.format(','.join(vertices)))


detect_text('g4.png')
```

実行するとエラーになった。GCPの方で支払い情報を設定する必要がある模様。

```
$ python main.py

Traceback (most recent call last):
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/google/api_core/grpc_helpers.py", line 57, in error_remapped_callable
    return callable_(*args, **kwargs)
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/grpc/_channel.py", line 690, in __call__
    return _end_unary_response_blocking(state, call, False, None)
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/grpc/_channel.py", line 592, in _end_unary_response_blocking
    raise _Rendezvous(state, None, None, deadline)
grpc._channel._Rendezvous: <_Rendezvous of RPC that terminated with:
	status = StatusCode.PERMISSION_DENIED
	details = "This API method requires billing to be enabled. Please enable billing on project #851590779607 by visiting https://console.developers.google.com/billing/enable?project=851590779607 then retry. If you enabled billing for this project recently, wait a few minutes for the action to propagate to our systems and retry."
	debug_error_string = "{"created":"@1574144877.044046000","description":"Error received from peer ipv4:216.58.197.202:443","file":"src/core/lib/surface/call.cc","file_line":1055,"grpc_message":"This API method requires billing to be enabled. Please enable billing on project #851590779607 by visiting https://console.developers.google.com/billing/enable?project=851590779607 then retry. If you enabled billing for this project recently, wait a few minutes for the action to propagate to our systems and retry.","grpc_status":7}"
>

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "main.py", line 24, in <module>
    response = client.label_detection(image=image)
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/google/cloud/vision_helpers/decorators.py", line 101, in inner
    response = self.annotate_image(request, retry=retry, timeout=timeout)
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/google/cloud/vision_helpers/__init__.py", line 72, in annotate_image
    r = self.batch_annotate_images([request], retry=retry, timeout=timeout)
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/google/cloud/vision_v1/gapic/image_annotator_client.py", line 274, in batch_annotate_images
    request, retry=retry, timeout=timeout, metadata=metadata
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/google/api_core/gapic_v1/method.py", line 143, in __call__
    return wrapped_func(*args, **kwargs)
  File "/Users/atsushiharada/.pyenv/versions/python-cloud-vision-sample/lib/python3.7/site-packages/google/api_core/grpc_helpers.py", line 59, in error_remapped_callable
    six.raise_from(exceptions.from_grpc_error(exc), exc)
  File "<string>", line 3, in raise_from
google.api_core.exceptions.PermissionDenied: 403 This API method requires billing to be enabled. Please enable billing on project #851590779607 by visiting https://console.developers.google.com/billing/enable?project=851590779607 then retry. If you enabled billing for this project recently, wait a few minutes for the action to propagate to our systems and retry.
```

支払い情報を設定する。

[https://console.cloud.google.com/billing/](https://console.cloud.google.com/billing/)

ポモドーロテクニックで作業をしていくとレベルが上がっていくWEBサービスの[g4](https://g-g-g-g.games/)のスクショからテキスト抽出する。

![g4](/images/blog/2019-11-19-cloud-vision-ocr-sample/g4.png)

```
$ python main.py
Texts:

"e status
Y bar
E menu
quest
現夫世界で25分間修行しよう
* 25分間集中してなにかに打ち込みましょう。
達成するとMP(Maturity point)を獲得できます。
. MPは主にキャラクターのレベルアップに使用します。
メモは他の人からは見えませんが、経験に変換すると公開されます。
86 37
Пеmо (beta)
process ing
"
bounds: (134,3),(552,3),(552,624),(134,624)

"e"
bounds: (316,3),(325,3),(325,19),(316,19)

"status"
bounds: (327,7),(365,7),(365,17),(327,17)

"Y"
bounds: (404,3),(412,3),(412,18),(404,18)

"bar"
bounds: (415,7),(433,7),(433,16),(415,16)

"E"
bounds: (163,6),(170,6),(170,16),(163,16)

"menu"
bounds: (171,9),(196,10),(196,18),(171,17)

"quest"
bounds: (246,7),(277,7),(277,19),(246,19)

"現"
bounds: (139,103),(150,103),(150,114),(139,114)

"夫"
bounds: (152,102),(162,102),(162,114),(152,114)

"世界"
bounds: (164,102),(188,102),(188,114),(164,114)

"で"
bounds: (190,105),(201,105),(201,114),(190,114)

"25"
bounds: (203,99),(214,99),(214,117),(203,117)

"分間"
bounds: (216,103),(239,103),(239,114),(216,114)

"修行"
bounds: (241,102),(265,102),(265,114),(241,114)

"しよ"
bounds: (268,102),(289,102),(289,114),(268,114)

"う"
bounds: (294,103),(302,103),(302,114),(294,114)

"*"
bounds: (145,138),(161,138),(161,146),(145,146)

"25"
bounds: (166,132),(175,132),(175,151),(166,151)

"分間"
bounds: (185,137),(196,137),(196,147),(185,147)

"集中"
bounds: (198,135),(222,135),(222,147),(198,147)

"し"
bounds: (225,137),(233,137),(233,147),(225,147)

"て"
bounds: (236,138),(247,138),(247,147),(236,147)

"なにか"
bounds: (249,135),(286,135),(286,147),(249,147)

"に"
bounds: (289,137),(291,137),(291,147),(289,147)

"打ち込み"
bounds: (293,135),(350,135),(350,147),(293,147)

"ましょ"
bounds: (352,132),(385,132),(385,151),(352,151)

"う"
bounds: (391,137),(400,137),(400,147),(391,147)

"。"
bounds: (404,143),(409,143),(409,147),(404,147)

"達成"
bounds: (160,157),(183,157),(183,169),(160,169)

"する"
bounds: (185,157),(208,157),(208,169),(185,169)

"と"
bounds: (213,158),(220,158),(220,169),(213,169)

"MP"
bounds: (224,160),(235,160),(235,168),(224,168)

"("
bounds: (243,157),(246,157),(246,171),(243,171)

"Maturity"
bounds: (249,158),(299,158),(299,171),(249,171)

"point"
bounds: (303,154),(336,154),(336,172),(303,172)

")"
bounds: (340,157),(343,157),(343,171),(340,171)

"を"
bounds: (353,157),(362,157),(362,169),(353,169)

"獲得"
bounds: (364,157),(388,157),(388,169),(364,169)

"でき"
bounds: (390,157),(414,157),(414,169),(390,169)

"ます"
bounds: (416,154),(437,154),(437,172),(416,172)

"。"
bounds: (443,165),(447,165),(447,169),(443,169)

"."
bounds: (145,185),(148,185),(148,188),(145,188)

"MP"
bounds: (159,176),(173,176),(173,194),(159,194)

"は"
bounds: (175,176),(185,176),(185,194),(175,194)

"主"
bounds: (187,176),(201,176),(201,194),(187,194)

"に"
bounds: (203,176),(211,176),(211,194),(203,194)

"キャラクター"
bounds: (213,176),(293,176),(293,194),(213,194)

"の"
bounds: (295,176),(299,176),(299,194),(295,194)

"レベル"
bounds: (303,180),(337,180),(337,191),(303,191)

"アップ"
bounds: (339,176),(375,176),(375,194),(339,194)

"に"
bounds: (377,176),(388,176),(388,194),(377,194)

"使用"
bounds: (390,176),(420,176),(420,194),(390,194)

"し"
bounds: (417,181),(425,181),(425,191),(417,191)

"ます"
bounds: (427,176),(451,176),(451,194),(427,194)

"。"
bounds: (455,187),(460,187),(460,191),(455,191)

"メモ"
bounds: (161,202),(183,202),(183,213),(161,213)

"は"
bounds: (185,201),(196,201),(196,213),(185,213)

"他"
bounds: (198,201),(209,201),(209,213),(198,213)

"の"
bounds: (211,204),(222,204),(222,213),(211,213)

"人"
bounds: (224,202),(234,202),(234,213),(224,213)

"から"
bounds: (236,201),(259,201),(259,213),(236,213)

"は"
bounds: (262,203),(264,203),(264,213),(262,213)

"見え"
bounds: (266,201),(298,201),(298,213),(266,213)

"ませ"
bounds: (300,201),(324,201),(324,213),(300,213)

"ん"
bounds: (327,203),(337,203),(337,213),(327,213)

"が"
bounds: (339,201),(350,201),(350,213),(339,213)

"、"
bounds: (353,211),(355,211),(355,213),(353,213)

"経験"
bounds: (358,198),(387,198),(387,216),(358,216)

"に"
bounds: (391,202),(393,202),(393,213),(391,213)

"変換"
bounds: (395,201),(426,201),(426,213),(395,213)

"する"
bounds: (428,201),(451,201),(451,213),(428,213)

"と"
bounds: (457,203),(464,203),(464,213),(457,213)

"公開"
bounds: (467,202),(490,202),(490,213),(467,213)

"さ"
bounds: (492,201),(503,201),(503,213),(492,213)

"れ"
bounds: (505,201),(516,201),(516,213),(505,213)

"ます"
bounds: (518,198),(542,198),(542,216),(518,216)

"。"
bounds: (545,209),(549,209),(549,213),(545,213)

"86"
bounds: (378,367),(457,364),(460,432),(381,435)

"37"
bounds: (473,364),(549,361),(552,429),(476,432)

"Пеmо"
bounds: (134,532),(155,532),(155,539),(134,539)

"(beta)"
bounds: (163,531),(193,531),(193,540),(163,540)

"process"
bounds: (447,616),(484,615),(484,623),(447,624)

"ing"
bounds: (487,614),(501,614),(501,624),(487,624)
```

無事抽出できたけど精度はイマイチだな。前回はもっと精度良かったので特殊なフォントは苦手なのかも知れない。