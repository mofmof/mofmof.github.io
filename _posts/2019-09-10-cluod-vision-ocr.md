---
layout: blog
title: Google Cloud Vision APIでOCRを試してみる
category: blog
tags: [Google Cloud Vision, OCR, GCP]
summary: 前回、Tesseractを使ったOCRを試してみました。続いて今度はGoogle Cloud VisionにもOCRする機能がある模様なので試してみます。
author: aharada
image: /images/blog/2019-09-10-cluod-vision-ocr/article-ocr.png
---

前回、Tesseractを使ったOCRを試してみました。続いて今度はGoogle Cloud VisionにもOCRする機能がある模様なので試してみます。

[Tesseractでサクッと日本語OCRを試してみる](/blog/tesseract-ocr.html)


## デモやってみる

GCPのAPIは大体デモが用意してあってどんなことが出来るかサクッと理解しやすくなってる。やはりあったのでやってみた。

[ドラッグ＆ドロップ Cloud Vision API ドキュメント Google Cloud](https://cloud.google.com/vision/docs/drag-and-drop)

誤認識なし。優秀だ。

![デモOCR](/images/blog/2019-09-10-cluod-vision-ocr/text-ocr.png)

弊社で運営している[my-logue](https://www.my-ope.net/my-logue/)というチャットボットのメディアの記事をスクショしてOCRかけてみる。テキストのブロックを認識出来るため、同じ水平位置のテキストでも別のテキストだと判断出来るみたい。これは優秀なのでは。 

![My-logueのOCR](/images/blog/2019-09-10-cluod-vision-ocr/article-ocr.png)

## curlでCloud Vision APIを叩いてみる

公式のドキュメントにしたがって進める。

[クイックスタート: クライアント ライブラリの使用 Cloud Vision API Google Cloud](https://cloud.google.com/vision/docs/quickstart-client-libraries?hl=ja)

たぶん事前にプロジェクトを作っておく必要があるので、作っておきましょう。

APIとサービスを開いて、Cloud  Visionを検索して有効化する。もしかしたら先に課金情報を入れないといけないかも知れん。

![APIとサービス](/images/blog/2019-09-10-cluod-vision-ocr/api-service.png)

![APIとサービスの有効化](/images/blog/2019-09-10-cluod-vision-ocr/api-enable.png)

つづいてサービス アカウントキーの作成。以前に作ったのがあるのでとりあえず使いまわしてみるか。正しく設定できたらキー情報の入ったjsonファイルをダウンロード出来るので、大事に保管しておく。

jsonファイルのパスを環境変数に設定。

```
$ export GOOGLE_APPLICATION_CREDENTIALS=/path/to/gcp.key.json
```

OCRする画像ファイルはCloud StorageかWEB上のURLを指定する模様(たぶんbase64エンコードされた文字列でもいける？)。ひとまずCloud Storageでやりたいので、適当にcloud-vision-ocr-sampleというbucketを作成しておく。

上記同様にMy-logueの記事のスクショを使う。ファイル名を`article.png`とした。

![My-logueのOCR](/images/blog/2019-09-10-cluod-vision-ocr/article-ocr.png)

それではcurlでpostリクエスト。

```
$ curl -X POST \
     -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
     -H "Content-Type: application/json; charset=utf-8" \
     --data "{
      'requests': [
        {
          'image': {
            'source': {
              'imageUri': 'gs://cloud-vision-ocr-sample/article.png'
            }
          },
          'features': [
            {
              'type': 'TEXT_DETECTION'
            }
          ]
        }
      ]
    }" "https://vision.googleapis.com/v1/images:annotate"
```

レスポンスなげええええ。iTerm2のログ上限を突破してしまったので全貌が見えないが、流し見すると、ちょいちょい一文字ずつや一単語ずつに座標情報などを付与して返しているみたい？こまけえな。

```
{
  "responses": [
    {
      "textAnnotations": [
        {
          "locale": "ja",
          "description": "チャットボットが働き方改革に効果的な10の理\n2019年7月5日\n由\nで、どんなチャットボットが\nいいの?有名チャットボット\nサービス9つまとめ!\nf\nLINE\nB!\n労働生産性の向上を目指す働き方改革。人手不足等の要因も重なり、\n小企業ではITを活用した業務効率化が急務になっています。\nとくに中\n資料請求\nRPAとともに業務改善に役立つツールが\nチャットボットです。そこでチャット\nボットが働き方改革に効果的な理由について列挙していきます。\nMy-ope office\n詳しくはこちらへ\n働き方改革とは\n働き方改革は、労働生産性の低下の改善を目的とした政府主導の取り組みで\nす。長時間労働を是正し、単位時間当たりの労働生産性を高めるためには、\nOJTだけでは限界があります。RPAやチャットボットなどITを導入すること\nで、労働生産性の向上を図らねばなりません。\n企業が\n取り組む働き方改革の実際\n",
          "boundingPoly": {
            "vertices": [
              {
                "x": 171,
                "y": 16
              },
              {
                "x": 1119,
                "y": 16
              },
              {
                "x": 1119,
                "y": 648
              },
              {
                "x": 171,
                "y": 648
              }
            ]
          }
        },
        {
          "description": "チャット",
          "boundingPoly": {
            "vertices": [
              {
                "x": 181,
                "y": 20
              },
              {
                "x": 275,
                "y": 20
              },
              {
                "x": 275,
                "y": 41
              },
              {
                "x": 181,
                "y": 41
              }
            ]
          }
        },
        {
          "description": "ボット",
          "boundingPoly": {
            "vertices": [
              {
                "x": 284,
                "y": 19
              },
              {
                "x": 353,
                "y": 19
              },
              {
                "x": 353,
                "y": 41
              },
              {
                "x": 284,
                "y": 41
              }
            ]
          }
        },

~~~

        "text": "チャットボットが働き方改革に効果的な10の理\n2019年7月5日\n由\nで、どんなチャットボットが\nいいの?有名チャットボット\nサービス9つまとめ!\nf\nLINE\nB!\n労働生産性の向上を目指す働き方改革。人手不足等の要因も重なり、\n小企業ではITを活用した業務効率化が急務になっています。\nとくに中\n資料請求\nRPAとともに業務改善に役立つツールが\nチャットボットです。そこでチャット\nボットが働き方改革に効果的な理由について列挙していきます。\nMy-ope office\n詳しくはこちらへ\n働き方改革とは\n働き方改革は、労働生産性の低下の改善を目的とした政府主導の取り組みで\nす。長時間労働を是正し、単位時間当たりの労働生産性を高めるためには、\nOJTだけでは限界があります。RPAやチャットボットなどITを導入すること\nで、労働生産性の向上を図らねばなりません。\n企業が\n取り組む働き方改革の実際\n"
      }
    }
  ]
}
```

デモでやったようにブロック情報が欲しいので、`TEXT_DETECTION`ではなく`DOCUMENT_TEXT_DETECTION`でリクエストしてみる。構造化されたテキストを取得するにはこっちの方が良さそう。

> TEXT_DETECTION は、任意の画像からテキストを検出、抽出します。たとえば、写真に道路名や交通標識が写っていれば、抽出された文字列全体、個々の単語、それらの境界ボックスが JSON レスポンスに含まれます。

> DOCUMENT_TEXT_DETECTION も画像からテキストを抽出しますが、高密度のテキストやドキュメントに応じてレスポンスが最適化され、ページ、ブロック、段落、単語、改行の情報が JSON に含まれます。


```
$ curl -X POST \
     -H "Authorization: Bearer "$(gcloud auth application-default print-access-token) \
     -H "Content-Type: application/json; charset=utf-8" \
     --data "{
      'requests': [
        {
          'image': {
            'source': {
              'imageUri': 'gs://cloud-vision-ocr-sample/article2.png'
            }
          },
          'features': [
            {
              'type': 'DOCUMENT_TEXT_DETECTION'
            }
          ]
        }
      ]
    }" "https://vision.googleapis.com/v1/images:annotate"
```

相変わらずどデカイjsonがかえってくる。`fullTextAnnotation.pages.blocks`にブロックごとの境界ボックスやテキストが入っているみたい。

```
{
  "responses": [
    {
      "textAnnotations": [
        {
          "locale": "ja",
          "description": "働き方改革は、労働生産性の低下の改善を目的とした政府主導の取り組みで\nす。長時間労働を是正し、単位時間当たりの労働生産性を高めるためには、\nOJTだけでは限界があります。 RPAやチャットボットなどITを導入すること\nで、労働生産性の向上を図らねばなりません。\n企業が取り組む働き方改革の実際\n",
          "boundingPoly": {
            "vertices": [
              {
                "x": 74,
                "y": 14
              },
              {
                "x": 616,
                "y": 14
              },
              {
                "x": 616,
                "y": 198
              },
              {
                "x": 74,
                "y": 198
              }
            ]
          }
        },
        {
          "description": "働き方改革は",
          "boundingPoly": {
            "vertices": [
              {
                "x": 77,
                "y": 14
              },
              {
                "x": 169,
                "y": 14
              },
              {
                "x": 169,
                "y": 37
              },
              {
                "x": 77,
                "y": 37
              }
            ]
          }
        },
        {

~~~

     "fullTextAnnotation": {
        "pages": [
          {
            "property": {
              "detectedLanguages": [
                {
                  "languageCode": "ja",
                  "confidence": 1
                }
              ]
            },
            "width": 719,
            "height": 237,
            "blocks": [
              {
                "boundingBox": {
                  "vertices": [
                    {
                      "x": 74,
                      "y": 14
                    },
                    {
                      "x": 616,
                      "y": 14
                    },
                    {
                      "x": 616,
                      "y": 124
                    },
                    {
                      "x": 74,
                      "y": 124
                    }
                  ]
                },
                "paragraphs": [
                  {
                    "boundingBox": {
                      "vertices": [
                        {
                          "x": 74,
                          "y": 14
                        },
                        {
                          "x": 616,
                          "y": 14
                        },
                        {
                          "x": 616,
                          "y": 124
                        },
                        {
                          "x": 74,
                          "y": 124
                        }
                      ]
                    },
                    "words": [
                      {
                        "property": {
                          "detectedLanguages": [
                            {
                              "languageCode": "ja"
                            }
                          ]
                        },
                        "boundingBox": {
                          "vertices": [
                            {
                              "x": 77,
                              "y": 14
                            },
                            {
                              "x": 169,
                              "y": 14
                            },
                            {
                              "x": 169,
                              "y": 37
                            },
                            {
                              "x": 77,
                              "y": 37
                            }
                          ]
                        },
                        "symbols": [
                          {
                            "property": {
                              "detectedLanguages": [
                                {
                                  "languageCode": "ja"
                                }
                              ]
                            },
                            "boundingBox": {
                              "vertices": [
                                {
                                  "x": 77,
                                  "y": 14
                                },
                                {
                                  "x": 86,
                                  "y": 14
                                },
                                {
                                  "x": 86,
                                  "y": 37
                                },
                                {
                                  "x": 77,
                                  "y": 37
                                }
                              ]
                            },
                            "text": "働",
                            "confidence": 0.98
                          },

~~~
```

## まとめ

前回やったTesseractに比べると、精度も高くブロック情報も解析してくれるので非常に使い勝手が良いのでは？という印象だった。次は`Amazon Textract`を試してみようと思ったけど、もうこれでいいんじゃないかなって気分。あ、でも`Amazon Textract`は表形式の情報も認識できるみたい。気になる。





