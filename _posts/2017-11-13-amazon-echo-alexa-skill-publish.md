---
layout: blog
title: 【Amazon Echo入門#5】Alexaちゃんでカイジの「ざわざわ」を演出するスキルを公開申請してみる
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー]
summary: 公開してみるとは言ったもののどこに公開されるんだろうか。日本語のスキルストアみたいのは見たことない。Amazon.comのスキルストアはあるみたいだけど。
author: aharada
image: /images/blog/2017-11-13-amazon-echo-alexa-skill-publish/global.png
---

前回の、[【Amazon Echo入門#4】Alexaちゃんにカイジの「ざわざわ」をやらせて焦燥感を演出するスキルを作る](/blog/amazon-echo-alexa-skill-kaiji.html)を実際に公開してみる。

公開してみるとは言ったもののどこに公開されるんだろうか。日本語のスキルストアみたいのは見たことない。[Amazon.comのスキルストア](https://www.amazon.com/b?node=13727921011)はあるみたいだけど。

まあとりあえずやってみましょう。

## Amazon 開発者コンソールから公開する

開発者コンソールからスキル「Kaiji」を選択し、「公開情報」を開きます。

公開情報を入力していきます。

- カテゴリー: Nevelty & Humor
- テストの手順: Alexaに「アレクサ、カイジやって」と話しかけます。
- 国と地域: Amazon がスキルを配布するすべての国と地域
- スキルの簡単な説明: Alexaに頼むと、カイジのあのざわざわする感じを演出してくれます
- スキルの詳細な説明: 長いので割愛
- サンプルフレーズ1: アレクサ、カイジやって
- サンプルフレーズ2: アレクサ、ざわざわやって
- サンプルフレーズ3: アレクサ、カイジ
- キーワード: カイジ, ざわざわ

![グローバルフィールド](/images/blog/2017-11-13-amazon-echo-alexa-skill-publish/global.png)

![スキル情報](/images/blog/2017-11-13-amazon-echo-alexa-skill-publish/skill-info.png)

![アイコン](/images/blog/2017-11-13-amazon-echo-alexa-skill-publish/icon.png)

続いて「プライバシーとコンプライアンス」画面を開いてそれぞれ入力。

- このスキルを使って何かを購入をしたり、実際にお金を支払うことができますか？: いいえ
- このスキルはユーザーの個人情報を収集しますか？: いいえ
- このスキルは13歳未満の子供を対象にしていますか？: いいえ
- 輸出コンプライアンス: チェック
- このスキルは広告を含みますか？: いいえ

![プライバシーとコンプライアンス](/images/blog/2017-11-13-amazon-echo-alexa-skill-publish/privacy.png)

必要情報を全て埋めたら申請ボタンが押せるようになるのでポチっとな。だいぶ雑に書いてみたから通るかどうか分からないけど、結果はまたのちほど載せるかも知れない。

## 2017/11/15 Amazonによる審査結果

Amazonから結果メールが飛んできました。結論から言うと、まあ当然リジェクトされました。そりゃダメだよね。

それぞれ対応していきます。

### 1.の対応

> \1.   Amazon Alexaスキルカタログに申請されたスキルに関して、第三者の商標またはブランドが使用されていたため、認定を却下いたしました。
>
> 使用されている第三者の商標またはブランド: カイジ
> 第三者の商標/ブランドが使用されていたメタデータ: スキル名、呼び出し名、スキルアイコン、スキルの詳細（このスキルについて）

すいません。カイジの顔をスキルアイコンにしていたのはさすがにダメですよね。アイコン画像をカイジの顔からオリジナルの「ざわざわ」に変更。

![ざわざわ](/images/blog/2017-11-13-amazon-echo-alexa-skill-publish/zawa108.png)

### 2,3.の対応

> \2.   スキルは、呼び出し名の要件を満たしていません。1単語だけの呼び出し名は使用できません（ただし、その単語が自分のブランド/知的財産である場合、または1単語が複数の単語を組み合わせたものの場合は例外です）。  必要な変更を行い、新しい呼び出し名が、スキルのサンプルフレーズ、スキルの説明、スキルの応答（あれば）で実際に使われていることを確認してください。
> https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/choosing-the-invocation-name-for-an-alexa-skill#invocation-name-requirements

> \3.   スキルは、呼び出し名の要件人名や地名（「molly」、「seattle」など）の呼び出し名は、それ以外の語と組み合わせ（「molly’s horoscope」など）ない限り使用できないという要件を満たしていません。 必要な変更を行い、新しい呼び出し名が、スキルのサンプルフレーズ、スキルの説明、スキルの応答（あれば）で実際に使われていることを確認してください。
> https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/choosing-the-invocation-name-for-an-alexa-skill#invocation-name-requirements

[https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/choosing-the-invocation-name-for-an-alexa-skill#invocation-name-requirements](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/choosing-the-invocation-name-for-an-alexa-skill#invocation-name-requirements)

貼ってあるURLの先を読んでもどう直せばよいのかイマイチ分からん。とりあえず単語一つの呼び出し名がダメっぽいので、スキル情報-呼び出し名を「カイジ」から「ざわざわするやつ」に変更してみた。

公開情報-テストの手順を『Alexaに「アレクサ、ざわざわするやつをスタートして」と話しかけます。』に変更。

### 4.の対応

> \4. ユーザーがスキル内の「help」をリクエストした際、スキルはユーザーにスキルのコア機能の操作方法を示すプロンプトを返す必要があります。また、ヘルププロンプトはユーザーへの質問で終わり、応答を受信するまでセッションを開いたままにしておく必要があります。
>
> 再現手順
> ユーザー：カイジ を 開い て ヘルプ.
> スキル：ざわざわ

どう対応したら良いのかわからないけど、「アレクサ、ざわざわするやつ ヘルプ」みたいな発言でHelpIntentも受け取って処理を返せるようにする。

インテントスキーマにHelpIntentを追加。

```
{
  "intents": [
    {
      "intent": "GetNewFactIntent"
    },
    {
      "intent": "HelpIntent",
      "slots": []
    }
  ]
}
```

サンプル発話にもHelpIntentを追加。

```
GetNewFactIntent ざわざわするやつ開いて
GetNewFactIntent ざわざわするやつをスタートして
GetNewFactIntent ざわざわするやつを実行して
HelpIntent ヘルプ
HelpIntent どうやってつかうの
HelpIntent 使い方
```

Lambda側の`lambda_function.py`もHelpIntentに対応する。この[Gistのコード](https://gist.github.com/n8henrie/3db1205331d0f6195b01)の一部をコピペして使わせてもらう。

```
def lambda_handler(event, context):
    intent_request = event['request']
    intent = intent_request['intent']
    intent_name = intent_request['intent']['name']

    if intent_name == "GetNewFactIntent":
        response = {
            'version': '1.0',
            'response': {
                "outputSpeech": {
                    "type": "SSML",
                    "ssml": "<speak> <audio src=\"https://s3-ap-northeast-1.amazonaws.com/for-alexa/xxxxxxxxxxx.mp3\" /> </speak>"
                },
            }
        }
    elif intent_name == "HelpIntent":
        session_attributes = {}
        card_title = "Welcome"
        speech_output = "ざわざわするやつをご利用いただきありがとうございます。"
        reprompt_text = "ざわざわするやつをスタート、と話しかけてください。"
        should_end_session = False
        response = build_response(session_attributes, build_speechlet_response(
            card_title, speech_output, reprompt_text, should_end_session))
    else:
        raise ValueError("Invalid intent")

    return response


def build_speechlet_response(title, output, reprompt_text, should_end_session):
    return {
        'outputSpeech': {
            'type': 'PlainText',
            'text': output
        },
        'card': {
            'type': 'Simple',
            'title': 'SessionSpeechlet - ' + title,
            'content': 'SessionSpeechlet - ' + output
        },
        'reprompt': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': reprompt_text
            }
        },
        'shouldEndSession': should_end_session
    }


def build_response(session_attributes, speechlet_response):
    return {
        'version': '1.0',
        'sessionAttributes': session_attributes,
        'response': speechlet_response
    }
```

GetNewFactIntentとHelpIntentの両方のテストを実行しエラーにならないことを確認しておく。

### 5.の対応

> \5.   コンパニオンアプリでユーザーへの提示用に選ばれたサンプルフレーズは、現在、サポートされていない起動フレーズを使用しています。
>
> 該当箇所
> アレクサ、カイジやって
> アレクサ、ざわざわやって
> アレクサ、カイジ
>
> 変更例
> アレクサ、カイジを開いて
> アレクサ、カイジをスタートして
> アレクサ、カイジを実行して

公開-サンプルフレーズを変更例を参考に修正する。

1. ざわざわするやつ開いて
2. ざわざわするやつをスタートして
3. ざわざわするやつを実行して

以上で出来上がり。再度申請してみる。結果が出たらまた追記していくかも。
