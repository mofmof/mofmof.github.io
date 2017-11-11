---
layout: blog
title: 【Amazon Echo入門#3】Lambdaの環境変数を使って、ゴミ出しの曜日を変数にする
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー]
summary: 今回はベタ打ちだったゴミ出しの曜日を環境変数化してみます。簡単だったので一瞬で終わった。
author: aharada
image: /images/blog/2017-11-11-amazon-echo-alexa-skill-lambda-environment-var/unburnable.png
---

![もやせないごみ](/images/blog/2017-11-11-amazon-echo-alexa-skill-lambda-environment-var/unburnable.png)

前回は、Amazon Echo Alexaちゃんにゴミ出しの曜日を教えてもらうスキルを開発しました。

[【Amazon Echo入門#2】Alexaちゃんにゴミ出しの曜日を教えてもらう(Lambda Python3.6)](/blog/amazon-echo-alexa-skill-trash-day.html)

今回はベタ打ちだったゴミ出しの曜日を環境変数化してみます。簡単だったので一瞬で終わった。

## Lambda側

環境変数を設定する。

- plastic: 木曜日
- recyclable: 土曜日
- burnable: 火曜日、金曜日
- unburnable: 第2第4土曜日

![環境変数](/images/blog/2017-11-11-amazon-echo-alexa-skill-lambda-environment-var/env.png)


`lambda_function.py`を下記のように更新します。

```python
import os

def lambda_handler(event, context):
    intent = event['request']['intent']
    value = intent['slots']['TrashType']['value']

    if value == '燃える':
        day = f"燃えるゴミの日は{os.environ['burnable']}です。"
    elif value == '燃えない':
        day = f"燃えないゴミの日は{os.environ['unburnable']}です。"
    elif value == '資源':
        day = f"資源ごみの日は{os.environ['recyclable']}です。"
    elif value == 'プラスチック':
        day = f"プラスチックごみの日は{os.environ['plastic']}です。"

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

![lambda_function.py](/images/blog/2017-11-11-amazon-echo-alexa-skill-lambda-environment-var/python-code.png)

ちょっと余談ですが、python3.6から`f-strings`なるものが使えるようになりました。いわゆる文字列内で変数展開したいときの記法です。

以前の記法は

```
weather = '晴'
print('今日の天気: %s' % weather)
```

とか

```
weather = '晴'
print('今日の天気: {}'.format(weather))
```

みたいな書き方をしていたのですが、直感的でないし、長いしで不満でしたがついに解消されました。

新記法`f-strings`

```
weather = '晴'
print(f"今日の天気: {weather}")
```

素晴らしい。

## Alexa側

前回プラスチックごみの曜日を入れ忘れてたので、`lambda_function.py`にしれっと追加してます。Alexa側も設定を追加する。

対話モデル画面のカスタムスロットタイプ`LIST_OF_TRASH_TYPE`にプラスチックも追加します。

![カスタムスロットタイプ](/images/blog/2017-11-11-amazon-echo-alexa-skill-lambda-environment-var/custom-slot-type.png)

テストしてみる。

![テスト結果](/images/blog/2017-11-11-amazon-echo-alexa-skill-lambda-environment-var/result.png)

できたああああああ！！
