---
layout: blog
title: 画像分類API，Google Cloud AutoML Visionは本当に松坂桃李と佐藤健を分類出来るのか？
category: blog
tags: [機械学習,machine learning,画像分類,Cloud AutoML Vision]
summary: GoogleにはCloud Vision APIっていう画像分類APIがある。基本的な画像分類はそれで十分なんだけど、実際に画像分類をソリューションとして利用したいシーンって、一般的なものの分類だけでは実現できなかったりする。
author: aharada
image: /images/blog/2018-10-23-cloud-automl-ikemen/kamenashi.png
---

![亀梨和也](/images/blog/2018-10-23-cloud-automl-ikemen/kamenashi.png)

前回はGoogleのCloud AutoML Visionを使って、イチゴとリンゴを分類するモデルを構築した(精度は著しくポンコツだったけど)。

[機械学習の知識なしでカスタムモデルが作れる画像分類API、Google Cloud AutoML Visionでリンゴとイチゴを分類してみる](/blog/cloud-automl-vision.html)

## なぜCloud VisionではなくAutoML Visionなのか？

そういえば根本的な話をしていなかった。

GoogleにはCloud Vision APIっていう画像分類APIがある。基本的な画像分類はそれで十分なんだけど、実際に画像分類をソリューションとして利用したいシーンって、一般的なものの分類だけでは実現できなかったりする。

例) きゅうり農家の事例

[https://persol-tech-s.co.jp/i-engineer/product/cucumber](https://persol-tech-s.co.jp/i-engineer/product/cucumber)

この例では元々きゅうりの等級の仕分けを人の手でやっていたのだけど、画像的に判別して等級を分類するAIを構築した例。

例えばこういうの実現しようと思った場合、Cloud Vision APIは一般的なものを識別することは出来るけど、「きゅうりの等級」という問題を学習したことはないので、あくまでもきゅうりは「きゅうり」としか認識できないわけ。

そこでAutoML Visionのようなカスタムデータセットを使って学習モデルを構築することで、ニッチな分類や、一般的ではない固有のものを分類することが出来るようになるのである。

## トレーニングする
こんな感じにサブディレクトリを切った構成でzipをアップロードすると、ディレクトリ名でラベリングされるらしいのでやってみる。

```
$ ikemen tree
.
├── takeru_sato
│   ├── 001.jpg
│   ├── 002.jpg
│   ├── 003.jpg
└── touki_matsuzaka
    ├── 001.jpg
    ├── 002.jpg
    └── 003.jpg
```

今回も一旦前処理は何もせずそのまま`ikemen.zip`にしてアップロード。

![データセットをアップロード](/images/blog/2018-10-23-cloud-automl-ikemen/dataset.png)

ちゃんとラベリングが済んだ状態でアップされてる！ステキ！！

トレーニングも済ませちゃいます。

![ラベリング済みデータセット](/images/blog/2018-10-23-cloud-automl-ikemen/labeled.png)

トレーニングが終わったので評価(EVALUATE)を見てみます。Precision/Recallが表示されているけどAccuracy(正答率)がない。どうやって評価すればいいんだろう。

![評価](/images/blog/2018-10-23-cloud-automl-ikemen/evaluate.png)

## 分類してみる

比較的あたりそうな松坂桃李を分類。99.9%佐藤健という誤答。ちげーぞオラ。

![松坂桃李1](/images/blog/2018-10-23-cloud-automl-ikemen/touki1.png)

ちょっとノイズのある松坂桃李は、88.9%佐藤健という誤答。

![松坂桃李2](/images/blog/2018-10-23-cloud-automl-ikemen/touki2.png)

ノイズのある佐藤健を分類。91.5%松坂桃李という誤答、見事に逆を行くポンコツ具合。

![佐藤健](/images/blog/2018-10-23-cloud-automl-ikemen/takeru.png)

データセットを集めている最中にゲシュタルト崩壊を起こして区別がつかなくなった亀梨和也をぶっこんでみる。100%佐藤健。分かるー区別つかないよね。

![亀梨和也](/images/blog/2018-10-23-cloud-automl-ikemen/kamenashi.png)

ほんの出来心でワイを分類してみる。99. 4%松坂桃李とのこと。まあ概ね正解と言って差し支えないだろう。

![原田敦](/images/blog/2018-10-23-cloud-automl-ikemen/harapan.png)

## まとめ

前回データセットが少なかったので、1.5倍くらいにした。2クラスの分類だし、ある程度いけるかな？と思ったけどやはりポンコツだった。やっぱり前処理ちゃんとやらなきゃダメかね。人物ではなく周辺の背景画像とかにかなりひっぱられてる気がする。

次回はちゃんと前処理にチャレンジしてみよう。
