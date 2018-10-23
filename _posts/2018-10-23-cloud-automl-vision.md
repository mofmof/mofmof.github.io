---
layout: blog
title: 機械学習の知識なしでカスタムモデルが作れる画像分類API、Google Cloud AutoML Visionでリンゴとイチゴを分類してみる
category: blog
tags: [機械学習,machine learning,画像分類,Cloud AutoML Vision]
summary: 前回IBM Visual RecognitionとCloud AutoML Visionどっちにしようか検討していたのですが、Cloud AutoML Visionの方を使うことに決定したので、実際に試してみますです。
author: aharada
image: /images/blog/2018-10-23-cloud-automl-vision/labeling.png
---

![ラベリング](/images/blog/2018-10-23-cloud-automl-vision/labeling.png)

前回IBM Visual RecognitionとCloud AutoML Visionどっちにしようか検討していたのですが、Cloud AutoML Visionの方を使うことに決定したので、実際に試してみますです。

[画像分類API、Google Cloud AutoML VisionとIBM Watson Visual Recognitionを比較してみる](/blog/image-classify-api.html)

まだβ版らしい。

[https://cloud.google.com/vision/](https://cloud.google.com/vision/)

Google Cloud Platformとは独立したサービスみたい。

## チュートリアル的にまずはやってみる

![TOP](/images/blog/2018-10-23-cloud-automl-vision/top.png)

どうやらGoogle Cloud Platformでプロジェクトを作成する必要があるみたい。GCPのコンソールを開いて適当にプロジェクトを作っておきましょう。

![プロジェクト選択](/images/blog/2018-10-23-cloud-automl-vision/project.png)

カード情報などの請求の設定が必要っぽい。ぼくは既にしてあったのでスルー。

2.の権限設定は、SETUP NOWボタンを押せばいい感じに設定してくれるんだろう。ポチッと。

![セットアップ](/images/blog/2018-10-23-cloud-automl-vision/setup.png)

`Customer bucket missing`と怒られたので、再びGCPのコンソールから適当にバケットを作っておこう。

このときのバケット名は`プロジェクト名-vcm`って名前じゃないとダメっぽい(画像は間違って作成してしまったやつなので参考にしないこと)。今回は`cloud-automl-vision-sample-vcm`という名前。`Multi-Regional`が一番安かったのでそれにした。作成できたら再びSETUP NOWをクリック。

![バケット作成](/images/blog/2018-10-23-cloud-automl-vision/bucket.png)

セットアップは出来たので、データセットを突っ込む。

![セットアップ完了](/images/blog/2018-10-23-cloud-automl-vision/completed.png)


## データセットを作る

とりあえず普通に分類できそうなテーマとして、リンゴとイチゴの分類をやってみる。

1ラベルごとに最低10枚画像が必要なので、Google画像検索でリンゴとイチゴを検索して、10枚ずつくらいの画像を入れる。イチゴの画像は1個ではなく複数写っているのも多くノイズになりそうだが、前処理とか何もせず突っ込んでみる。

アップロードはjpg,png,zipに対応しているみたいなので、全部zipにしてアップする

Classification typeはリンゴorイチゴの1ラベルだけ返せばいいので、`Enable multi-label classification`はオフでOK。一度の予測で複数ラベルを返したい場合はチェックするだけでいいっぽい。

日本語名のファイルがあるとエラーが出る。

![日本語名ファイルのエラー](/images/blog/2018-10-23-cloud-automl-vision/error-ja.png)

## トレーニング
画面上からポチポチとラベル付け出来る。リンゴの画像には`apple`をイチゴの画像には`strawberry`というラベルをつける。

![ラベリング](/images/blog/2018-10-23-cloud-automl-vision/labeling.png)

トレーニングいくぜ！

![トレーニング](/images/blog/2018-10-23-cloud-automl-vision/train1.png)

`Invalid arguments of request`でちょっとハマる。

機械学習ではデータセットを学習用・検証用・評価用の3つ分けて使用することが一般的で、各用途ごとに今回のイチゴとリンゴの2ラベル付きのサンプルが含まれている必要があります。

![Invalid arguments of request](/images/blog/2018-10-23-cloud-automl-vision/invalid-error.png)

AutoMLでは自動的にこの用途3つに分けてくれるのですが、なにぶんデータセットが少なすぎるので、それぞれの用途にサンプルちょうどよく分散されていなかった模様。

![csv](/images/blog/2018-10-23-cloud-automl-vision/csv1.png)

なので、手動で振り分けてアップロードします。

![修正後csv](/images/blog/2018-10-23-cloud-automl-vision/csv2.png)

適当な名前をつけてGCSにアップロードして、csvファイルをインポートします。ちなみにこのcsv手動でゼロから作成するのはだるいので、上記のラベリング後にエクスポートしたものを利用しました。

![インポート](/images/blog/2018-10-23-cloud-automl-vision/import.png)

改めてトレーニング再開。

![トレーニング](/images/blog/2018-10-23-cloud-automl-vision/train2.png)

15分くらいかかるぽい。4回やったら無料枠こえてしまうが、モデルごとに1時間分が無料なので、別のモデルを作った場合にはまた無料枠がある。とりあえずいろんなデータセットを試してみる上では費用はかからず済みそうだ。

## 予測してみる
適当に拝借してきた、どうみてもリンゴの画像で予測してみる。100%リンゴだろって結果。まあ当然ですな。

![リンゴ画像を予測1](/images/blog/2018-10-23-cloud-automl-vision/apple1.png)

ちょっとノイズがありそうな背景画像もついて複数写っているリンゴの画像で予測してみる。こちらも楽勝で正答。

![リンゴ画像を予測2](/images/blog/2018-10-23-cloud-automl-vision/apple2.png)

続いてこちらもノイズがありそうなイチゴの画像で予測。こちらは残念ながら94.6%リンゴだろ？と誤答。オイちげーぞこれはイチゴだ。データセットが少なすぎるかな。

![イチゴ画像を予測1](/images/blog/2018-10-23-cloud-automl-vision/ichigo1.png)

明らかにイチゴだろと思える画像で予測。あら、これも99.9%リンゴと判断されちゃいましたね。やはりデータセットが少なすぎるのと前処理何もやっていないからダメっぽいかな。

![イチゴ画像を予測2](/images/blog/2018-10-23-cloud-automl-vision/ichigo2.png)

ちなみに、トレーニングするだけで予測APIのエンドポイントが生成されるみたい。ステキ。

![エンドポイント](/images/blog/2018-10-23-cloud-automl-vision/endpoint.png)

## まとめ
画像のアップロードからラベリング、トレーニング、評価、予測まで一気通貫で出来るので非常に使い勝手がよい。

以前自分でtensorflowを使って画像分類モデルを作ったことがあるが、プロトタイプ的に動かすのであれば、こっちでやったほうが圧倒的に簡単だし早い。難しい機械学習の知識もほとんどいらないので、正直神レベルのサービスなのではないかと思った。
