---
layout: blog
title: 画像分類API、Google Cloud AutoML VisionとIBM Watson Visual Recognitionを比較してみる
category: blog
tags: [機械学習,machine learning,画像分類,Cloud AutoML Vision,Visual Recognition]
summary: 新規事業で使いたい技術の調査で、GoogleのCloud AutoML VisionかIBMのVisual Recognitionか、どちらを使うべきか検討中なので、まずは料金の試算をしてみる。
author: aharada
image: /images/blog/2018-10-23-image-classify-api/main.png
---

![AutoML Vision](/images/blog/2018-10-23-image-classify-api/main.png)

新規事業で使いたい技術の調査で、GoogleのCloud AutoML VisionかIBMのVisual Recognitionか、どちらを使うべきか検討中なので、まずは料金の試算をしてみる。

Cloud AutoML Visionの料金について

[https://cloud.google.com/vision/automl/pricing?hl=ja](https://cloud.google.com/vision/automl/pricing?hl=ja)

Visual Recognitionの料金について

[https://console.bluemix.net/catalog/services/visual-recognition](https://console.bluemix.net/catalog/services/visual-recognition)

## 前提条件

- プロトタイプ的なアプリケーションの開発がゴール
- モデルのデータセットは小さく、精度向上には取り組まない

## Cloud AutoML Vision

- モデルのトレーニングは、1時間分は無料だが、それ以降は$20/時がかかる。
	- 今回はプロトタイプなので、多分トレーニングは2時間以内でいけるんじゃないかな。0円で。
- 画像へのラベリングは100枚/月のラベリングは無料
	- 今回は2クラス分類程度になると思うので100枚でいけると思う。0円。
- 予測は1,000回まで無料、それ以降は$3/1,000枚
	- 無料の範囲におさまると思うけど、
	- 10万回リクエストされたとしても$3,000。そんなに恐い数字じゃない。
- 画像を保存するストレージ
	- [https://cloud.google.com/storage/pricing?hl=JA](https://cloud.google.com/storage/pricing?hl=JA)
	- 5GB/月までは無料。画像100枚いかない予定なので解像度を数MBにおさえれば無料でいけそう。
- APIリクエストの上限設定は出来るか？
	- ちょっと調べた限り上限設定出来る旨は見つけられなかった
	- Google Cloud PlatformのAPIは通常、APIリクエスト数の上限を設定できるはずだが不明

### 結論

基本的には0円で試せる。仮に、ある程度の規模のアクセスがあったとしても上限は数万円程度で済ませられそう。

## Visual Recognition

- 1,000イベント/1ヶ月までは無料らしいが、イベントってなんだろうか。とりあえずリクエストと読み替えて検討する
- モデルのトレーニング
	- 1イベントとカウントされるのであれば無料の範囲でいけそうだが、本当にそうなのか？ラベリングとトレーニングではコンピューティングの度合いは全然違うと思うのだが
- 画像へのラベリング
	- 無料プランでは特に記述がないので、やはり1ラベリング作業は1イベントと考えればいいのか？
	- だとすると1,000イベントなので、無駄にラベリングのミスとかしない限りは無料の範囲でいけるはず
- 予測も上記同様
- 画像を保存するストレージ
	- 特に記述はないので分からない
- APIリクエストの上限設定は出来るか？
	- 特に記述はないので分からない

### 結論
たぶんお試しでやる範囲では無料で出来そうだが、料金設定が不明確なのでちょっとビビる。Cloud AutoML Visionの方を使おうかな。
