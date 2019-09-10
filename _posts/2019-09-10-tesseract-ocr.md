---
layout: blog
title: Tesseractでサクッと日本語OCRを試してみる
category: blog
tags: [OCR, Tesseract]
summary: TesseractはPythonからオープンソースで使えるOCRエンジンで、テッセラクトと読むらしい。とりあえずインストールしたらサクッとOCRを試せるみたいなのでやってみる。
author: aharada
# image:
---

TesseractはPythonからオープンソースで使えるOCRエンジンで、テッセラクトと読むらしい。とりあえずインストールしたらサクッとOCRを試せるみたいなのでやってみる。

こちらの記事を参考にした。

[Tesseract+PyOCRで簡易OCRを試してみる - Qiita](https://qiita.com/nabechi6011/items/3a367ca94dbd208efcc7)

## とりあえずOCRをしてみる

まずはインストールする。`brew`で一発。

```
$ brew install tesseract
```

デフォルトでは日本語対応していないので、日本語の学習データを手動で入れる。

データは三種類のリポジトリが用意されている。

- tessdata_bestは最も精度が高いデータ
- tessdata_fastはたぶん最も速度が速いデータ
- tessdataは通常のデータ

[Data Files · tesseract-ocr/tesseract Wiki · GitHub](https://github.com/tesseract-ocr/tesseract/wiki/Data-Files)

ひとまず`tessdata`で試してみることにする。

[GitHub - tesseract-ocr/tessdata](https://github.com/tesseract-ocr/tessdata)

GitHubリポジトリ内の`jpn.traineddata`をダウンロードする。

学習データ置き場を確認する。ここで合ってるぽい。4.1.0の部分はtesseractのバージョンが入るので各自の環境で異なります。

```
ls /usr/local/Cellar/tesseract/4.1.0/share/tessdata

configs          eng.traineddata  osd.traineddata  pdf.ttf          snum.traineddata tessconfigs
```

学習データを移動します。

```
$ mv ~/Downloads/jpn.traineddata /usr/local/Cellar/tesseract/4.1.0/share/tessdata
```

この画像をOCRしてみます。

![テキスト画像](/images/blog/2019-09-10-tesseract-ocr/text.png)

```
$ tesseract text.png result
Tesseract Open Source OCR Engine v4.1.0 with Leptonica
Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 175
```

ふむ。日本語が有効になっていないようだ。

```
$ cat result.txt
Type Belk CHS Db LNB 123.
BUDABS, RHRUVKLVE RH
```

オプションが必要みたい。

```
$ tesseract text.png result -l jpn                                                                                                                                  Tesseract Open Source OCR Engine v4.1.0 with Leptonica
Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 175
```

ふーむ。とんでもないポンコツ具合である。これだと使いものにならなそう。

```
$ cat result.txt
Type 我 華 は 殖 で あ る か も し れ な い ⑫③。
あ い う え お S、&⑧ 楽 し い 楽 し い 夏 会 み
```

## 日本語のOCR精度を上げる
色々調べてみるとTesseract4からはLSTMになって精度が上がっているらしい。

だがしかし、すでに4系が入っていた。

```
$ tesseract -v
tesseract 4.1.0
 leptonica-1.78.0
  libgif 5.1.4 : libjpeg 9c : libpng 1.6.37 : libtiff 4.0.10 : zlib 1.2.11 : libwebp 1.0.3 : libopenjp2 2.3.1
 Found AVX2
 Found AVX
 Found SSE
```

そういえば、学習データには精度が高い版があったはず。そっちを試してみる。同様にリポジトリから`jpn.traineddata`をダウンロードしてくる。

 [tessdata_best](https://github.com/tesseract-ocr/tessdata_best) 

ファイルを移動する。

```
$ mv ~/Downloads/jpn.traineddata /usr/local/Cellar/tesseract/4.1.0/share/tessdata
```

OCRしてみる。

```
~/Desktop ❯❯❯ tesseract text.png result -l jpn                                                                                                                           ✘ 1
Tesseract Open Source OCR Engine v4.1.0 with Leptonica
Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 175
```

だいぶ改善した。我輩の「輩」以外は正しく認識されている(このスペースどうにかなんねえかな)。

```
$ cat result.txt
Type 我 斉 は 猫 で ある か も し れ な い 123。
あい うえ お $、&& 楽 し い 楽 し い 夏 休み
```

ついでに、`jpn_vert.traineddata`というのも見つけたのでよくわからんが試してみる。

同様にファイルを移動しておく(コマンド割愛)。

```
$ tesseract text.png result -l jpn_vert
Tesseract Open Source OCR Engine v4.1.0 with Leptonica
Warning: Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 175
```

なんじゃこりゃあああ。

```
$ cat result.txt
っoo洪映再庶わっWeyジせてさや
きいSuい>トおe 9の湯漁てS光謀
```

ちょっとだけ検索してみたが、`jpn_vert.traineddata`の用途はわからないままだった。

## まとめ
サクッとOCRができる点が良かった。精度もまあ用途によっては使えるのではないだろうか。

あと学習はLSTMなのでFine Tuning(転移学習)ができる。ので、固有の学習データを追加で学習してあげれば、うまいこと使えるのではないだろうか。

[Tesseract 4.1にLSTMを使って日本語を再学習させる - Qiita](https://qiita.com/aki_abekawa/items/418e069038fbdb77c59e)