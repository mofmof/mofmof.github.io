---
layout: blog
title: 【Amazon Echo入門#9】SESを使ってAlexaちゃんからAmazonアカウントのメールアドレスにメールを送る
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー, Link with Amazon, SES]
summary: TODO
author: aharada
image: /images/blog/2017-12-20-alexa-link-with-amazon/...jpg
---

前回、Login with Amazonを使って、Alexa側でメールアドレスを取得するところまでやりました。続いて、そのメールアドレスにメールを送信する実装をしてみます。

前回

[【Amazon Echo入門#8】AlexaちゃんとAmazonアカウントをリンクしてメールアドレスを取得する(Serverless Frameworkを使ってみた)](/blog/2017-12-21-alexa-lwa-ses)

こちらの記事を参考に進めます。

[https://dev.classmethod.jp/cloud/aws/lambda-to-ses/](https://dev.classmethod.jp/cloud/aws/lambda-to-ses/)

## SES側の設定

今回は一つのメールアドレス宛にメールを送信していきたいので、送信先のメールアドレスを事前に検証しておけばOK。SESの`Email Addresses`を開きます。

![SES - Email Addresses](/images/blog/2017-12-21-alexa-lwa-ses/verify-email.png)

宛先のメールアドレスを入力すると、`pending verification`状態になります。

![宛先メールアドレスを登録](/images/blog/2017-12-21-alexa-lwa-ses/verify-email2.png)

![pending verification](/images/blog/2017-12-21-alexa-lwa-ses/verify-email3.png)

登録したメールアドレス宛に検証メールが届くので、本文内のURLをクリックしたら検証完了になります。

![pending verification](/images/blog/2017-12-21-alexa-lwa-ses/verify-email4.png)

## Lambda側

前回に引き続き`Serverless Framework`を使っていきます。前回の実装に、メール送信する実装を追加します。

`handler.py`

```
import json
import urllib.request
import boto3

MAIL_TO = 'from@exmaple.com'  # SESで登録したメールアドレス

def hello(event, context):
    access_token = event['session']['user'].get('accessToken')
    print(f'access_token: {access_token}')
    data = fetch_profile(access_token)

    send_email(data['email'], MAIL_TO, '件名ほげほげ', 'メール送ります')

    response = {
        'version': '1.0',
        'response': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': 'メールを送信しました'
            }
        }
    }
    return response


def fetch_profile(access_token):
    with urllib.request.urlopen(f"https://api.amazon.com/user/profile?access_token={access_token}") as res:
        body = res.read().decode("utf-8")
        print(body)

        data = json.loads(body)
        return data


def send_email(source, to, subject, body):
    client = boto3.client('ses', region_name='us-west-2')

    response = client.send_email(
        Source=source,
        Destination={
            'ToAddresses': [
                to,
            ]
        },
        Message={
            'Subject': {
                'Data': subject,
            },
            'Body': {
                'Text': {
                    'Data': body,
                },
            }
        }
    )
    return response
```

`serverless.yml`

`iamRoleStatements`の記述が追加されています。

```
service: link-with-amazon

provider:
  name: aws
  runtime: python3.6
  region: ap-northeast-1
  iamRoleStatements:
    - Effect: "Allow"
      Action:
        - ses:SendEmail
        - ses:SendRawEmail
      Resource: "*"

functions:
  hello:
    handler: handler.hello
    events:
      - alexaSkill
```

Alexa実機からスキルを呼び出してみると、以下のようなメールが届きます。

![受信メール](/images/blog/2017-12-21-alexa-lwa-ses/result.png)

## ハマったエラー

途中、こんなエラーにハマった。要するに、Lambda Functionにses:SendMailのroleがないよってエラーなんですが、`serverless.yml`にもちゃんと`iamRoleStatements`したのになぁと色々いじってた。

結論、`iamRoleStatements`は`provider`の下にいれなきゃいけなかったのが原因でした。同列に書いていたのでroleに反映されなかった模様。

```
{
    "errorMessage": "An error occurred (AccessDenied) when calling the SendEmail operation: User `arn:aws:sts::0000000000:assumed-role/link-with-amazon-dev-ap-northeast-1-lambdaRole/link-with-amazon-dev-hello' is not authorized to perform `ses:SendEmail' on resource `arn:aws:ses:us-west-2:000000000:identity/hoge@example.com'",
    "errorType": "ClientError",
    "stackTrace": [
        [
            "/var/task/handler.py",
            21,
            "hello",
            "send_email(MAIL_TO, MAIL_TO, '件名ほげほげ', 'メール送ります')"
        ],
        [
            "/var/task/handler.py",
            50,
            "send_email",
            "'Data': body,"
        ],
        [
            "/var/runtime/botocore/client.py",
            314,
            "_api_call",
            "return self._make_api_call(operation_name, kwargs)"
        ],
        [
            "/var/runtime/botocore/client.py",
            612,
            "_make_api_call",
            "raise error_class(parsed_response, operation_name)"
        ]
    ]
}
```
