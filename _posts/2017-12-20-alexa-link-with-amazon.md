---
layout: blog
title: 【Amazon Echo入門#8】AlexaちゃんとAmazonアカウントをリンクしてメールアドレスを取得する(Serverless Frameworkを使ってみた)
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー, Link with Amazon]
summary: 久しぶりにAlexaちゃん入門をアップします。今回はAlexaちゃんとAmazonアカウントをリンクするところを実装してみました。
author: aharada
image: /images/blog/2017-12-20-alexa-link-with-amazon/login-with-amazon.png
---

ちょっと時間が空きましたが、久しぶりにAlexaちゃん入門をアップします。今回はAlexaちゃんとAmazonアカウントをリンクするところを実装してみました。Amazonアカウントにはメールアドレスが紐付いているので、リンクしたユーザーにメールで何かアクションしたいときに使えると思います。

今回、Lambdaの開発にServerless Frameworkを使ってみました。便利な気もする。ちょっと`deploy`に時間がかかるのでデバッグがしんどいけど、local実行も出来るらしい。今度調べてみよう。

こちらの記事を参考にしてます。

[https://dev.classmethod.jp/cloud/alexa-account-linking-login-with-amazon/](https://dev.classmethod.jp/cloud/alexa-account-linking-login-with-amazon/)

## Login with Amazon(LWA)の設定

Amazonの[開発者コンソール画面](https://developer.amazon.com/home.html)から、「アプリ&サービス」の「Amazonでログイン」画面を開きます。

![Login with Amazon](/images/blog/2017-12-20-alexa-link-with-amazon/login-with-amazon.png)

どうやらセキュリティプロファイルを作らなければならないみたい。進めて行きます。

- Security Profile Name: AlexaLoginSample
- Security Profile Description: Sample Account Login with Amazon
- Consent Privacy Notice URL: 任意のURL(ユーザーにリンクが表示されるだけなので適当でもとりあえず大丈夫)

![セキュリティプロファイル](/images/blog/2017-12-20-alexa-link-with-amazon/security-profile.png)

これだけで出来たっぽい。簡単や。「Show Client ID and Client Secret」の部分をクリックするとIDとSecretが表示される。あとで使うので手元にコピペしておく。

![Login with Amazon2](/images/blog/2017-12-20-alexa-link-with-amazon/login-with-amazon2.png)

## スキルの作成とリダイレクトURLの設定

とりあえずAmazonアカウントとリンクできれば良いので、スキルざっと作っておく。作り方は以前にやってるので割愛します。

参考

- [【Amazon Echo入門#1】Alexaちゃんに今日履いているパンツの色を答えさせる](/blog/amazon-echo-alexa-skill.html)
- [【Amazon Echo入門#6】AlexaちゃんとTwitterアカウントを連携してみる](/blog/amazon-echo-alexa-skill-link-twitter.html)

設定 - アカウントリンクで、「ユーザーにアカウントの作成や既存のアカウントへのリンクを許可しますか？」に「はい」を設定すると、自動的にリダイレクトURLが表示されます。これをコピーして、Login with Amazonの方に設定します。

![リダイレクトURL](/images/blog/2017-12-20-alexa-link-with-amazon/redirect-url.png)

![リターンURLs](/images/blog/2017-12-20-alexa-link-with-amazon/allowed-return-urls.png)

アカウントリンクの設定項目を埋めていきます。

- 認証URL: https://www.amazon.com/ap/oa
- クライアントID: Login with Amazonで取得したClientID
- スコープ: profile
- アクセストークンURL: https://api.amazon.com/auth/o2/token
- クライアントシークレット: Login with Amazonで取得したClientSecret

![アカウントリンク](/images/blog/2017-12-20-alexa-link-with-amazon/account-link.png)

## Lambdaの実装

Serverless Frameworkを使用してLambdaを実装していきます。

こちらの記事を参考にしました。

[https://dev.classmethod.jp/cloud/aws/easy-deploy-of-lambda-with-serverless-framework/](https://dev.classmethod.jp/cloud/aws/easy-deploy-of-lambda-with-serverless-framework/)

`aws-cli`のセットアップは済んでいるものとして割愛。

npmでserverless frameworkをインストール。

```
$ npm install serverless -g
```

python3.6でLambdaのプロジェクトを作る。

```
$ sls create -t aws-python3 -p link-with-amazon
```

デプロイして実行してみる。

```
$ cd link-with-amazon
$ sls deploy
$ sls invoke -f hello
{
    "body": "{\"input\": {}, \"message\": \"Go Serverless v1.0! Your function executed successfully!\"}",
    "statusCode": 200
}
```

これだけでLambdaにFunctionが追加されます。コマンドベースでバチバチやれるので楽ですね。

次に、リージョンを東京に変更、TriggerをAlexaに設定変更します。

`serverless.yml`

```
service: link-with-amazon

provider:
  name: aws
  runtime: python3.6
  region: ap-northeast-1

functions:
  hello:
    handler: handler.hello
    events:
      - alexaSkill
```

Functionの実装。

`handler.py`

```
import json
import urllib.request

def hello(event, context):
    access_token = event['session']['user'].get('accessToken')
    print(f'access_token: {access_token}')
    fetch_profile(access_token)
    response = {
        'version': '1.0',
        'response': {
            'outputSpeech': {
                'type': 'PlainText',
                'text': 'ほげほげ'
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
```

## 実機から試してみる

動作確認のためLambdaのログをtailしておきます。

```
$  sls logs -f hello -t
```

[Alexaポータル](https://alexa.amazon.co.jp/spa/index.html)から開発中のスキル、`LinkWithAmazon`でAmazonアカウントとリンクします。

リンクが成功したら、呼び出し名を話しかけて、以下のようにアカウント情報がログに出力されることを確認。ちゃんとメールアドレスが取得できてますね！

```
START RequestId: 3aafe005-e4ce-11e7-a42f-a13c00653d3a Version: $LATEST
{"user_id":"amzn1.account.AEUZQT37CEFY4NPZGRMNJPMCZ6OA","name":"原田敦","email":"hogehoge@example.com"}
END RequestId: 3aafe005-e4ce-11e7-a42f-a13c00653d3a
REPORT RequestId: 3aafe005-e4ce-11e7-a42f-a13c00653d3a  Duration: 1012.57 ms    Billed Duration: 1100 ms  Memory Size: 1024 MB    Max Memory Used: 23 MB
```
