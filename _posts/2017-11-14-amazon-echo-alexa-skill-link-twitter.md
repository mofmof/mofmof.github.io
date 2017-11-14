---
layout: blog
title: 【Amazon Echo入門#6】AlexaちゃんとTwitterアカウントを連携してみる
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー, Twitter]
summary: 外部アプリケーションと連携してアカウント認証の仕組みが必要になるので、そのあたりを調べてみたい。今回は一旦はAlexaちゃんとTwitterアカウントを紐付けるところまでやってみます。
author: aharada
image: /images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/success.png
---

前々回くらいにやったゴミ出しの日をAlexaちゃんに教えてもらうとしたりとかをちゃんと作ろうとすると、当然住んでいる地域でゴミ出しの曜日はバラバラなので、ユーザーごとに異なる結果を返す必要があります。

ともすると、外部アプリケーションと連携してアカウント認証の仕組みが必要になるので、そのあたりを調べてみたい。今回は一旦はAlexaちゃんとTwitterアカウントを紐付けるところまでやってみます。

参考: [Amazon AlexaのAccount Linkingを使ってAmazon EchoからTwitterに書き込ませてみる](https://dev.classmethod.jp/cloud/aws/tweet-from-echo-using-by-alexa-account-linking/)


## サーバ側の準備

認証を通すには、AlexaちゃんとTwitterの間でアカウント連携を仲介する必要があります。サンプルコードがそのまま使えるのでセットアップしていきましょう。

[https://github.com/bignerdranch/developing-alexa-skills-solutions/tree/master/5_accountLinking](https://github.com/bignerdranch/developing-alexa-skills-solutions/tree/master/5_accountLinking)

とりあえずソースコードをローカルに持ってくる。

```
$ git clone git@github.com:bignerdranch/developing-alexa-skills-solutions.git
```

ローカルで起動してみて正しく動くことを確認しておく。

```
$ cd developing-alexa-skills-solutions/5_accountLinking/solution/server
$ ruby -v
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-darwin14]
$ bundle install
$ thin start --ssl -p 3000
```

画面を開く

https://localhost:3000/app

thinはじめて使ったけど、sslも出来るみたい。ただしssl警告とかキーチェーン認証とかガンガンでてくる。とりあえず無視して進めば画面は開けた。

![サーバ側画面](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/local-server.png)

次はこれをHerokuにデプロイして、Alexaちゃんからも見えるようにします。Herokuアプリ(alexa-twitter-connect)は既に登録済みの状態。

```
$ cd ..
$ cp -r server ~/source_code/alexa-twitter-connect-server
$ cd ~/source_code/alexa-twitter-connect-server

$ git init
$ git add .
$ git commit

$ git remote add heroku https://git.heroku.com/alexa-twitter-connect.git
$ git push heroku master

$ heroku open
```

![サーバ画面(Heroku)](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/heroku-server.png)

## Twitter Appの準備

OAuth認証の仕組みを使うので、[Twitter Application Management](https://apps.twitter.com/)からApplicationを作る必要があります。普通にOAuth認証と同じ感じでOK。

- Website: https://alexa-twitter-connect.herokuapp.com/app
- Callback URL: https://alexa-twitter-connect.herokuapp.com/oauth/callback

![Twitter Settings](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/twitter.png)

登録が出来たら、Keys and Access Tokensタブを開き、Consumer Key (API Key)とConsumer Secret (API Secret)をメモっておく。


## Alexa側

いつものようにAlexaのスキルを作成します。基本的な作成の仕方は[【Amazon Echo入門#1】Alexaちゃんに今日履いているパンツの色を答えさせる](/amazon-echo-alexa-skill.html)でやっているのでそちらを参考に。

[https://developer.amazon.com/](開発者コンソール)

インテントスキーマ・カスタムスロットタイプ・サンプル発話の設定は、上記GitHubのサンプルコードの`solution/skill/airportinfo/resources`配下のtxtファイルをコピペすればOK。

インテントスキーマ

```
{
  "intents": [
    {
      "slots": [
        {
          "name": "AIRPORTCODE",
          "type": "FAACODES"
        }
      ],
      "intent": "tweetAirportStatusIntent"
    },
    {
      "slots": [
        {
          "name": "AIRPORTCODE",
          "type": "FAACODES"
        }
      ],
      "intent": "airportInfoIntent"
    }
  ]
}
```

カスタムスロット

- タイプ: FAACODES
- 値:
  - AAC
  - AAE
  - AAF
  - ...省略

サンプル発話

```
tweetAirportStatusIntent  tweet {AIRPORTCODE}
tweetAirportStatusIntent  tweet delay {AIRPORTCODE}
tweetAirportStatusIntent  tweet status {AIRPORTCODE}
tweetAirportStatusIntent  tweet info {AIRPORTCODE}
tweetAirportStatusIntent  tweet delay info {AIRPORTCODE}
tweetAirportStatusIntent  tweet status info {AIRPORTCODE}
tweetAirportStatusIntent  tweet for {AIRPORTCODE}
tweetAirportStatusIntent  tweet delay for {AIRPORTCODE}
tweetAirportStatusIntent  tweet status for {AIRPORTCODE}
tweetAirportStatusIntent  tweet info for {AIRPORTCODE}
tweetAirportStatusIntent  tweet delay info for {AIRPORTCODE}
tweetAirportStatusIntent  tweet status info for {AIRPORTCODE}
airportInfoIntent {AIRPORTCODE}
airportInfoIntent flight {AIRPORTCODE}
airportInfoIntent airport {AIRPORTCODE}
airportInfoIntent delay {AIRPORTCODE}
airportInfoIntent flight delay {AIRPORTCODE}
airportInfoIntent airport delay {AIRPORTCODE}
airportInfoIntent status {AIRPORTCODE}
airportInfoIntent flight status {AIRPORTCODE}
airportInfoIntent airport status {AIRPORTCODE}
airportInfoIntent info {AIRPORTCODE}
airportInfoIntent flight info {AIRPORTCODE}
airportInfoIntent airport info {AIRPORTCODE}
airportInfoIntent delay info {AIRPORTCODE}
airportInfoIntent flight delay info {AIRPORTCODE}
airportInfoIntent airport delay info {AIRPORTCODE}
airportInfoIntent status info {AIRPORTCODE}
airportInfoIntent flight status info {AIRPORTCODE}
airportInfoIntent airport status info {AIRPORTCODE}
airportInfoIntent for {AIRPORTCODE}
airportInfoIntent flight for {AIRPORTCODE}
airportInfoIntent airport for {AIRPORTCODE}
airportInfoIntent delay for {AIRPORTCODE}
airportInfoIntent flight delay for {AIRPORTCODE}
airportInfoIntent airport delay for {AIRPORTCODE}
airportInfoIntent status for {AIRPORTCODE}
airportInfoIntent flight status for {AIRPORTCODE}
airportInfoIntent airport status for {AIRPORTCODE}
airportInfoIntent info for {AIRPORTCODE}
airportInfoIntent flight info for {AIRPORTCODE}
airportInfoIntent airport info for {AIRPORTCODE}
airportInfoIntent delay info for {AIRPORTCODE}
airportInfoIntent flight delay info for {AIRPORTCODE}
airportInfoIntent airport delay info for {AIRPORTCODE}
airportInfoIntent status info for {AIRPORTCODE}
airportInfoIntent flight status info for {AIRPORTCODE}
airportInfoIntent airport status info for {AIRPORTCODE}
```

続いてアカウントリンクの設定。

- 認証 URL: `https://alexa-twitter-connect.herokuapp.com/oauth/request_token?vendor_id=<VENDOR_ID>&consumer_key=<TWITTER_CONSUMER_KEY>&consumer_secret=<TWITTER_CONSUMER_SECRET>`
  - VENDOR_IDは、直後の「リダイレクトURL」に表示されているvendorId=xxxxxxxの部分をコピペする。
- ドメインリスト: `api.twitter.com`を追加しておく
- 認可の承諾タイプ: 簡易版のImplicit Grantにしておく

エンドポイントのデフォルトは、今回はアカウントリンクだけ試したいので、とりあえず適当なLambdaのARN IDを入れておく。カイジのとか入れておいた。

![アカウントリンク](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/account-link.png)

## サーバに環境変数を設定する

Herokuの環境変数にTwitterアプリのキー情報など、必要な値をセットします。

```
$ heroku config:set CONSUMER_KEY=<Twitterで取得したConsumer key>
$ heroku config:set CONSUMER_SECRET=<Twitterで取得したConsumer secret>
$ heroku config:set VENDOR_ID=<Alexa側で自動的に発行されるID、上記参照>
$ heroku config:set CALLBACK_URL=https://alexa-twitter-connect.herokuapp.com/oauth/callback
$ heroku config:set SESSION_SECRET=<サンプルコードのままでOK>
```

## 試してみる

[Alexaのポータル](https://alexa.amazon.com/)へ行ってスキルを開くと言語が違うと言われて先に進めず試せない。。

そう、なにしろ、そもそもぼくはAmazon Echo実機を持っていないのである。

![Alexa言語が一致しない](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/lang-failed.png)

実機がなければ日本語設定も出来ないのだけど、[echosim.io](https://echosim.io/)というWEB上でAlexaのシミュレーションが出来るサービスがある。Alexaのポータルではこれも有効みたい。

だがしかし、このサービスは`amazon.jp`のアカウントでは利用できず、`amazon.com`のアカウントが必要。仕方ないので、Amazonアカウントも、開発者コンソールも、`amazon.com`の方で取り直して、Alexaのスキルも作り直した。

※この手順はAmazon Echo実機が入手出来れば必要ないです。

よし、いけたいけた。この画面では開発中のスキルも試すことが出来ます。

![Alexa Skills](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/skills.png)

諸々問題なくいけばこのようにリンク成功の画面が表示されます。

![Twitterリンク成功](/images/blog/2017-11-14-amazon-echo-alexa-skill-link-twitter/success.png)

## ハマった点

どうしてもAlexaポータルからTwitter連携をしようとすると、認可は通るものの、なぜかPINコードが表示されてしまい、どうにも連携出来ない状態でハマりました。

結論から言うとHeroku側の環境変数設定忘れでした。`heroku config:set HOGEHOGE=xxxxxxxxxxxxxxx`ってやつ。参考元の記事にはこの手順は書いてなかったのでご注意を。
