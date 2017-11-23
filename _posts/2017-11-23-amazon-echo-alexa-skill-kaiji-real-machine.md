---
layout: blog
title: 【Amazon Echo入門#7】Alexaちゃんにカイジに「ざわざわ」を演出してもらうスキルを実機で動かす
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー]
summary: ついに ねんがんの アマゾンエコーをてにいれたぞ！というわけで、このAmazon Echo Alexa入門シリーズで実際に開発した、カイジのざわざわを演出するスキルをついに実機で動かしてみようと思います。
author: aharada
image: /images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/zawa.jpg
---

![ざわざわ](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/zawa.jpg)

ついに ねんがんの アマゾンエコーをてにいれたぞ！

正確に言うと、こんな募集を出したところ

![Amazon Echo貸して欲しい](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/rental.png)

友人であるちくちゅうさんが貸してくださいました。本当にありがとうございます！！

![ちくちゅうさんが貸してくれた](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/chikuchu.png)

というわけで、このAmazon Echo Alexa入門シリーズで実際に開発した、カイジのざわざわを演出するスキルをついに実機で動かしてみようと思います。

こちらが完成品。

<iframe width="560" height="315" src="https://www.youtube.com/embed/uGwa3Y6zXMw" frameborder="0" allowfullscreen></iframe>

- [【Amazon Echo入門#4】Alexaちゃんにカイジの「ざわざわ」をやらせて焦燥感を演出するスキルを作る](/blog/amazon-echo-alexa-skill-kaiji.html)
- [【Amazon Echo入門#5】Alexaちゃんでカイジの「ざわざわ」を演出するスキルを公開申請してみる](amazon-echo-alexa-skill-publish.html)

## 初期設定して動くようにする

とりあえずUSBケーブルをつなげてONにして話しかけてみたが反応しない。しばらく待つとAlexaアプリをインストールせよと言われる。だが実はPCでも設定できるのを知っているので無視してPCから設定を進める。

まず、https://alexa.amazon.co.jp/ を開いて、Amazonアカウントでログイン。

こんな画面が開くので、「新しいデバイスをセットアップ」をクリック。

![新しいデバイスをセットアップ](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/start-setup.png)

どのEchoかを選択する画面。今回はdotをお借りしたのでdotを選択。

![どのEchoか選択](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/select-type.png)

言語を日本語に設定し、wifiの設定に続きます。

一旦Echoに直接接続する。

![Echoに直接接続](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/amazon-gbw.png)

PCの画面からwifiの設定が出来るので、自宅やら職場やらのwifi接続情報を入力。

![WiFiの設定](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/wifi-setup.png)

完了画面が出ればOK。

![設定完了](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/complete.png)

その後動画を見ろと行ってくるが、永遠にぐるぐるしてやがるので無視して[ホーム](https://alexa.amazon.co.jp/spa/index.html)に戻ろう。別に見なくても大丈夫。

![動画見れない](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/movie.png)

これで既に動くようになっているので、「Alexa、こんにちは」とか話しかけてみよう。認識精度めっちゃいい。これで何度もHey Hey言わなくて済みますね！

## カスタムスキルが動くようにする

Alexaの開発者コンソールで既にスキル開発していれば、自動的に「有効なスキル」に入ってるみたい。

![スキル](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/skills.png)

![有効なスキル](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/enable-skills.png)

Alexaに話しかけてみる。

わし「アレクサ、ざわざわするやつ開いて」
Alexaたん「スキルからの応答に問題があります」

ふーむ。とりあえずLambdaの実行ログがCloudWatchに入ってるみたいなので、エラーログを探すと該当箇所見つかった。

```
'intent': KeyError
Traceback (most recent call last):
File "/var/task/lambda_function.py", line 5, in lambda_handler
intent = intent_request['intent']
KeyError: 'intent'
```

![CloudWatch](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/cloud-watch.png)

Alexaからのリクエストに`intent`が含まれていないのが原因ということが分かった。どうやら`LaunchRequest`というものがあって、Alexaに最初に話しかけて、どのスキルを発動するのかを選択されたときのリクエストらしい。そして`LaunchRequest`には`intent`が含まれない。

解決方法としては、このざわざわスキルは特にスロット(変数)を必要とせず、起動したらざわざわするだけだから、`intent`が含まれないリクエストが来た場合も、ざわざわするようにLambdaのコードを修正した。

lambda_function.py

```python
def lambda_handler(event, context):
    intent_request = event['request']
    intent = intent_request.get('intent')
    intent_name = 'GetNewFactIntent'
    if intent != None:
        intent_name = intent_request['intent']['name']

    if intent_name == "GetNewFactIntent":
        response = {
            'version': '1.0',
            'response': {
                "outputSpeech": {
                    "type": "SSML",
                    "ssml": "<speak> <audio src=\"https://s3-ap-northeast-1.amazonaws.com/for-alexa/zawazawa_conv.mp3\" /> </speak>"
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
```

ちなみにLambdaのテストのテンプレートで、`Alexa Start Session`というのがあるので、これで`LaunchRequest`で飛んでくるリクエストっぽいのをテスト出来ます。このテストを実行してエラー吐かなければOK。

![Start Sessionテスト](/images/blog/2017-11-23-amazon-echo-alexa-skill-kaiji-real-machine/start-session-test.png)

一通り動作することを確認したら実機に再挑戦。

わし「アレクサ、ざわざわするやつスタート」
Alexaたん「ざわ・・・ざわ・・・」

できたあああああああああああ！！！

## 呼び出し名について

Alexaたんにはスキルを開始するときにの呼び出し名というのがあって、「{呼び出し名}を開始」とか「{呼び出し名}をスタート」という風に話しかけます。この後ろの「開始」とか「スタート」とかはAlexa側で固定で設定されてるみたい。

どんなワードで起動できるのは以下サイトに一覧が書いてあった。

[http://redorereadblog.hatenablog.jp/entry/2017/10/31/090312](http://redorereadblog.hatenablog.jp/entry/2017/10/31/090312)

Amazonの公式情報はこっち。

[ユーザーによるカスタムスキルの呼び出し](https://developer.amazon.com/ja/docs/custom-skills/understanding-how-users-invoke-custom-skills.html)
