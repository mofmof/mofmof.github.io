---
layout: blog
title: 【Amazon Echo入門#4】Alexaちゃんにカイジの「ざわざわ」をやらせて焦燥感を演出するスキルを作る
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー]
summary: Alexaちゃんに「Alexa, カイジやって」とか「Alexa, ざわざわして」とかお願いすると、こんな感じにざわざわしてくれます。
author: aharada
image: /images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/zawazawa.jpg
---

![ざわざわ](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/zawazawa.jpg)

## 完成品

<iframe width="560" height="315" src="https://www.youtube.com/embed/3-0lFGXOwks" frameborder="0" gesture="media" allowfullscreen></iframe>

Alexaちゃんに「Alexa, カイジやって」とか「Alexa, ざわざわして」とかお願いすると、こんな感じにざわざわしてくれます。

例えば仕事中にヤバいトラブルが発生して焦っているときとか、嫁に浮気を追及されてバレそうな場面とかで、すかさずざわざわさせると、ただでさえ焦っているのに、さらにその場を焦燥感あふれる舞台にしてくれます。これは大変重宝するスキルですね！

まだ実物が入手できないのでシミュレーターでしか再生できないのがアレですが、Alexaちゃん待ち遠しいですね。

## 前回の記事
[【Amazon Echo入門#3】Lambdaの環境変数を使って、ゴミ出しの曜日を変数にする](/blog/amazon-echo-alexa-skill-lambda-environment-var.html)

## 音声の準備

Alexaではテキストを読み上げたり、音声ファイルを再生したりする際は、SSMLというマークアップ言語を使用します。スキル発動のレスポンスに、SSMLを渡せばその通り動いてくれます。

まず音声はYoutubeからmp3ファイルでダウンロードして変換をかけます。元動画。

<iframe width="560" height="315" src="https://www.youtube.com/embed/2BfNLm0Vj-Y" frameborder="0" allowfullscreen></iframe>

ダウンロードするやりかたはWEBで調べればすぐ分かるので割愛。

ダウンロードしたmp3ファイルをEchoで再生出来る形に変換します。MPEGのバージョンとかビットレートとか細かく指定されていて、ちゃんと守らないとテスト再生時に `Error: Please make sure that "Alexa Skills Kit" is selected for the event source type of arn:aws:lambda:ap-northeast-1:000000000:function:Kaiji` みたいになってしまうのでちゃんと間違いなく変換しましょう。

再生できる形式はこちらのエントリを参考。

[https://kotodama.today/?p=251](https://kotodama.today/?p=251)

> 使用するMP3にはいくつかの条件があります。
>　・ インターネット上でホスティングされた HTTPSエンドポイントで提供されていること。
>　・ SSL証明書がオレオレ証明書ではなくAmazonの基準を満たす証明書であること。（公式サイトではAWS S3の使用を提案しています）
>　・ カスタマーの個人情報やその他デリケートな情報を含んでいないこと
>　・ 有効なMP3ファイルであること（MPEG version 2）
>　・ 90秒以内であること
>　・ ビットレートが48KB/sであること
>　・ サンプルレートが16,000Hzであること

ffmpegをインストールします。

```
$ brew install ffmpeg
```

コマンドラインで音声ファイルを変換。変換のコマンドは以下エントリを参考にした。

[https://dev.classmethod.jp/cloud/alexa-ssml-audio/](https://dev.classmethod.jp/cloud/alexa-ssml-audio/)

```
$ ffmpeg -i zawazawa.mp3 -ac 2 -codec:a libmp3lame -b:a 48k -ar 16000 zawazawa_conv.mp3
```

変換が成功したらitunesからビットレートとかが確認できます。

![曲の詳細](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/itunes.png)

## 音声ファイルをS3にアップする

Alexaちゃんが読み込める場所に音声ファイルを置く必要があるので、手軽にアップできるAWS S3に置くことにします。

AWS S3を開き、Create bucketします。

- Bucket name: for-alexa
- Region: Asia Pacific(Tokyo)
- その他は全てデフォルトで

![S3 Create bucket](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/create-bucket.png)

さきほど変換した音声ファイルをアップロード。設定値は全てデフォルトでOK。

![S3 upload](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/s3-upload.png)

アップロードが完了したら、publicアクセスを許可する。

![S3 make public](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/s3-make-public.png)

Overviewに表示されているURLを開いてざわざわが再生されることを確認できたら準備OK．

## Lambda側

いつものようにAWS Lambdaのダッシュボード画面を開き、Create functionする。Blueprintsは使わずにAuthor from scratch。

- Name: kaiji
- Role: Choose an existing role
- Existing role: lambda_basic_execution

![Lambda設定](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/lambda-basic-info.png)

`lambda_functin.py`にSSMLを返す関数を定義する。Testを実行してみて、問題なく実行できることを確認しておく。

![lambda_functin.py](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/function.png)

`lambda_functin.py`

```
def lambda_handler(event, context):
    response = {
        'version': '1.0',
        'response': {
            "outputSpeech": {
                "type": "SSML",
                "ssml": "<speak> <audio src=\"https://s3-ap-northeast-1.amazonaws.com/for-alexa/zawazawa_conv.mp3\" /> </speak>"
            },
        }
    }
    return response
```

TriggersにAlexa Skills Kitを設定する(これいつも忘れる)。

![Triggers](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/trigger.png)

## Alexa側

いつも通りスキルの設定をしていきます。

- スキルの種類: Custom
- 言語: Japanese
- スキル名: Kaiji
- 呼び出し名: カイジ

![スキル情報](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/skill-info.png)

インテントスキーマの設定。カスタムスロットは空でOK。

![インテントスキーマ](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/intent-schema.png)

```
{
  "intents": [
    {
      "intent": "GetNewFactIntent"
    }
  ]
}
```

サンプル発話。

![サンプル発話](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/sample-text.png)

```
GetNewFactIntent カイジやって
GetNewFactIntent ざわざわして
GetNewFactIntent ざわざわやって
GetNewFactIntent カイジ
```

設定でエンドポイトにLambdaのARN IDを設定。

![エンドポイント](images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/endpoint.png)

テスト実行すると、レスポンスにSSMLが返ってきてることが確認出来ます。

![テスト実行](/images/blog/2017-11-11-amazon-echo-alexa-skill-kaiji/test-result.png)

聴くを押してみたらざわざわがはじまることを確認。
できたああああああああ！！
