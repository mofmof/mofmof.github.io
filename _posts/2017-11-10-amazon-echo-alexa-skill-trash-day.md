---
layout: blog
title: 【Amazon Echo入門】Alexaちゃんにゴミ出しの曜日を教えてもらう(Lambda Python3.6)
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー]
summary: 前回、Amazon Echo Alexaのカスタムスキルを作る入門をやってみました。今回は少しコードやインテントをいじってカスタマイズしてみます。
author: aharada
---

前回、Amazon Echo Alexaのカスタムスキルを作る入門をやってみました。今回は少しコードやインテントをいじってカスタマイズしてみます。

[【Amazon Echo入門】Alexaちゃんに今日履いているパンツの色を答えさせる](/blog/amazon-echo-alexa-skill.html)

ぼくはNode.jsよりPythonの方がいくらか得意なので、LambdaのPython3.6で開発してみます。

## Lambdaの開発

AWSのLabmdaダッシュボード画面から「Create Function」します。

Blueprintsからalexaで検索したが、Python3.6のものがなかった。仕方ないので今回もAuthor from scratchで実装します。

![Blueprints](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/blueprints.png)


前回はどうもAlexaとやりとりするための規定のリクエストとレスポンスのフォーマットがわからなかったのだけど、そこまで複雑なものではなく、単にレスポンスでJSONを返せばいいだけみたい。

参考：[https://medium.com/@jacquelinewilson/amazon-alexa-skill-recipe-1444e6ee45a6](https://medium.com/@jacquelinewilson/amazon-alexa-skill-recipe-1444e6ee45a6)

```
def lambda_handler(event, context):
    intent = event['request']['intent']
    value = intent['slots']['TrashType']['value']

    if value == '燃える':
        day = '燃えるゴミの日は火曜日です。'
    elif value == '燃えない':
        day = '燃えないゴミの日は水曜日です。'
    elif value == '資源':
        day = '資源ごみの日は木曜日です。'


    response = {
        'version': '1.0',
        'response': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': day
            }
        }
    }
    return response
```    

![Pythonの実装](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/lambda-python.png)

前回同様にTriggersにAlexa Skills Kitを指定します。

![トリガー](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/trigger.png)

ちゃんと動作するか確認したいので、Lambda Functionのテストを設定してみます。リクエスト内容は、intentのslots配下のみ書き換えます。

- Event template: Alext Intent - GetNewFact
- Event name: GetNewFact

```
{
  "session": {
    "new": false,
    "sessionId": "amzn1.echo-api.session.[unique-value-here]",
    "attributes": {},
    "user": {
      "userId": "amzn1.ask.account.[unique-value-here]"
    },
    "application": {
      "applicationId": "amzn1.ask.skill.[unique-value-here]"
    }
  },
  "version": "1.0",
  "request": {
    "locale": "en-US",
    "timestamp": "2016-10-27T21:06:28Z",
    "type": "IntentRequest",
    "requestId": "amzn1.echo-api.request.[unique-value-here]",
    "intent": {
      "slots": {
        "TrashType": {
          "name": "TrashType",
          "value": "燃えない"
        }
      },
      "name": "GetNewFactIntent"
    }
  },
  "context": {
    "AudioPlayer": {
      "playerActivity": "IDLE"
    },
    "System": {
      "device": {
        "supportedInterfaces": {
          "AudioPlayer": {}
        }
      },
      "application": {
        "applicationId": "amzn1.ask.skill.[unique-value-here]"
      },
      "user": {
        "userId": "amzn1.ask.account.[unique-value-here]"
      }
    }
  }
}
```

![Lambdaのテスト](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/lambda-test.png)

実行すると、`outputSpeech`の`text`に「燃えないゴミの日は水曜日です。」と返ってきていることが確認出来る。

![Lambdaのテスト結果](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/lambda-test-result.png)


## Alexa Skill側の設定

[開発者コンソール](https://developer.amazon.com/edw/home.html#/)から前回同様に新しいSkillを登録していきます。

- スキル名: TrashDay
- 呼び出し名: ゴミ出しの日

![スキル情報](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/alexa-info.png)

対話モデルのインテントスキーマの設定。

```
{
  "intents": [
    {
      "slots": [
        {
          "name": "TrashType",
          "type": "LIST_OF_TRASH_TYPE"
        }
      ],
      "intent": "GetNewFactIntent"
    }
  ]
}
```

カスタムスロットタイプにゴミの種別を設定する。

- タイプ: LIST_OF_TRASH_TYPE
- 値:
  - 燃える
  - 燃えない
  - 資源


サンプル発話に以下を設定。

```
GetNewFactIntent {TrashType} ゴミの日
GetNewFactIntent {TrashType} ゴミの日教えて
GetNewFactIntent {TrashType} ごみの日
GetNewFactIntent {TrashType} ごみの日教えて
```

![対話モデル](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/conversation-model.png)


前回同様に、エンドポイントのデフォルトにLambdaのARN IDを指定する。

![設定](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/preference.png)

シミュレータでテストしてみる。

「燃えるゴミの日教えてください」と入力して呼び出すと、インテントのTrashTypeが「燃える」と認識され、レスポンスで「燃えるゴミの日は火曜日です。」と返って来ていることが確認出来る。

![シミュレータでテスト](/images/blog/2017-11-10-amazon-echo-alexa-skill-trash-day/last-test.png)

## まとめ

無事できました！

インテント周りの概念がちょっと慣れるまでわかりづらいですが、一度やってみると感覚的に理解できるようになりますね。

今回もLambdaで実装しましたが、どうやら普通にエンドポイントを指定して、HTTPでリクエストしてもいけるみたいです。これを利用すれば外部のサーバーにリクエストしたりできるので、既存のサービスとの連携も色々できそうですね。

次回はゴミ出しの日を、プログラムに直接入れるのではなく、どこか別のデータソースから取ってくる実装をしてみたいなーと思ってます。
