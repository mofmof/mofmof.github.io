---
layout: blog
title: Amazon EchoのAlexaちゃんに今日履いているパンツの色を答えさせる
category: blog
tags: [Amazon Echo, Alexa, アマゾンエコー]
summary: ただ発売を待っているだけもアレなので、先にスキル作ってみます。Alexaちゃんに今はいているパンツの色を聞いたら、何色か答えてくれるスキルを作る。
author: aharada
image: /images/blog/2017-11-09-amazon-echo-alexa-skill/exec-test.png
---

いよいよAmazon Echo日本語対応版の発売が今日2017/11/09に発表されました！！実際の発売は来週以降になるようで、今日買う気満々だったのですが、もうちょい待たねばならぬ。

Google Homeを見送ってEcho一択勝負だったので待ち遠しい。

ところで、EchoのAlexaは既に開発者向けに日本語対応のコンソールが公開されており、Alexa用のSkillが開発出来るようになってます。ただ発売を待っているだけもアレなので、先にスキル作ってみます。

Alexaちゃんに今はいているパンツの色を聞いたら、何色か答えてくれるスキルを作る。

このあたりの公式情報を参考に進めていきます。またしてもクラスメソッドさんか。

[https://developer.amazon.com/ja/alexa-skills-kit/training/building-a-skill](https://developer.amazon.com/ja/alexa-skills-kit/training/building-a-skill)

## Amazon Developer登録

AWS Lambdaを使うのでAWSアカウント取得が必要なのですが、長くなるのでそのあたりの手順は割愛します。

スキル開発を進めるには、[Alexa Skills Kit画面](https://developer.amazon.com/ja/alexa-skills-kit)から行きます。

開発者ログイン画面に行くと思うので、登録を進めていきます。普通に買い物用のAmazonのアカウントをそのまま使える。

プロフィール情報などの入力が求められるので、普通に必要な項目を埋めていく。

![profile](/images/blog/2017-11-09-amazon-echo-alexa-skill/profile.png)

App Distribution Agreementは、利用規約のようなもので、合意を求められます。内容問題なければ、「承認して続行」ボタンをクリック。

![App Distribution Agreement](/images/blog/2017-11-09-amazon-echo-alexa-skill/app-distribution-agreement.png)

次は「支払い」画面。よく分からんけど、なんとなく口座情報入力とか出てきたらめんどそうだし、現時点で収益化は狙っていないので、デフォルトの「いいえ」のままで進める。

![支払い](/images/blog/2017-11-09-amazon-echo-alexa-skill/pay.png)

ダッシュボードがでたら開発者アカウントの登録は完了。「ALEXA」を選択します。

![ダッシュボード](/images/blog/2017-11-09-amazon-echo-alexa-skill/dashboard.png)

## Alexa Skillの設定

Alexa Skill Kitの方を選択。

![Alexaで開発を始める1](/images/blog/2017-11-09-amazon-echo-alexa-skill/start-dev-alexa-1.png)
![Alexaで開発を始める2](/images/blog/2017-11-09-amazon-echo-alexa-skill/start-dev-alexa-2.png)

スキルの設定を入力していく。

- スキルの種類: カスタム対話モデル
- 言語: Japanese
- スキル名: WhatColor
- 呼び出し名: パンツ何色
- グローバルフィールド: 全てデフォルトの「いいえ」

![スキルを登録](/images/blog/2017-11-09-amazon-echo-alexa-skill/regist-skill.png)

対話モデル画面。インテントスキーマは以下をコピペ。現時点ではなんだかよく分かってない。公式情報のまんまです。スキルビルダーを起動すんなと公式ブログに書いてあるので気をつけよう。


```
{ "intents": [
  { "intent": "GetNewFactIntent" },
  { "intent": "AMAZON.HelpIntent" },
  { "intent": "AMAZON.StopIntent" },
  { "intent": "AMAZON.CancelIntent" }
]}
```

![インテントスキーマ](/images/blog/2017-11-09-amazon-echo-alexa-skill/conversation-model.png)

カスタムスロットタイプは空でOK。

サンプル発話は、実際に起動したい時の対話で使われそうな文章を複数パターン入れておく。

オレは一体何をやっているのだろうか、という気分になってくる。


```
GetNewFactIntent パンツは何色ですか
GetNewFactIntent 今日は何色のパンツを履いていますか
GetNewFactIntent パンツ何色
GetNewFactIntent パンティ何色
GetNewFactIntent 何色のパンティ履いてるの
GetNewFactIntent 何色のパンツはいてるんですか
GetNewFactIntent 今どんな色のパンツはいてるの
```

ここまでいったら一旦スキルの設定を保留。

## Lambdaの設定

続いて[AWS Lambda](https://ap-northeast-1.console.aws.amazon.com/lambda/home)の設定。

ダッシュボード画面からCreate Functionボタンをクリック。

![ダッシュボード](/images/blog/2017-11-09-amazon-echo-alexa-skill/lambda-dashboard.png)

テンプレート的なのは使わず Author from scratchボタンをクリック。

![Blueprints](/images/blog/2017-11-09-amazon-echo-alexa-skill/blueprints.png)

Lambdaの基本設定画面。

- Name: WhatColor
- Role: Create custom role

![Lambdaの基本情報](/images/blog/2017-11-09-amazon-echo-alexa-skill/basic-information.png)

Create custom roleを選択すると別画面が開くので、特に値を変更せずAllowする。

![Create custom role](/images/blog/2017-11-09-amazon-echo-alexa-skill/create-custom-role.png)

Lambdaにコードを実装していきます。公式ブログからとってきた実装済みのコードのzipファイルをダウンロード。

[https://m.media-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/jp/tutorials/fact/lambda.zip](https://m.media-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-skills-kit/jp/tutorials/fact/lambda.zip)

ファイルを展開して、`index.js`内の変数`data`に配列を格納している箇所を以下のように変更する。

```
var data = [
    "今日のパンツの色はピンクです",
    "今履いているパンツの色は白です",
    "私のパンツの色は水色です",
    "今日のパンツの色は黒です",
    "実は今パンツ履いてないんです",
    "今日のパンツはグレーです",
];
```

再び圧縮してzipファイルにするのですが、macの右クリック圧縮ではなぜかうまくいかなくて少しハマった。下記ブログに記載されているのと同じようにコマンドで圧縮したらうまくいった。

ソースコードが置いてあるカレントディレクトリにcdしてからzipしないとダメっぽい。

[https://www.business-on-it.com/2003/techblog/aws-lambda-create-function/](https://www.business-on-it.com/2003/techblog/aws-lambda-create-function/)

```
$ cd lambda
$ zip -r hoge.zip index.js node_modules/
```

ちなみに本当はpythonで実装したかったのですが、どうやらAlexaでLambdaを使う場合、規定のインターフェースというかルールみたいのにそってfunctionを定義しないと、シミュレータテスト時に`The response is invalid`というエラーが出てしまいうまくいかないため、既存のコードを利用した。

zipしたファイルをLambdaにアップロードします。

![zipをアップロード](/images/blog/2017-11-09-amazon-echo-alexa-skill/lambda-code.png)

Triggersタブで、Alexa Skills KitをAddする。

![Triggerを追加](/images/blog/2017-11-09-amazon-echo-alexa-skill/add-triger.png)

テストイベントを作成して実行。succeededが表示されればOK。

- Event template: Alexa Start Session
- Event name: AlexaStartSession

![テストイベントを作成](/images/blog/2017-11-09-amazon-echo-alexa-skill/test-event.png)

テストが成功したら画面右上のARNってところに表示されているIDをコピーしておく。

```
arn:aws:lambda:ap-northeast-1:00000000000:function:WhatColor
```

## LambdaとAlexaを接続する

[開発者コンソール](https://developer.amazon.com/edw/home.html#/skills)に戻って、lambdaと接続させる。

メニューの「設定」の「エンドポイント」の「デフォルト」に先程コピーしらARNのIDをペーストします。

![エンドポイントの設定](/images/blog/2017-11-09-amazon-echo-alexa-skill/endpoint.png)

「パンツ何色ですか」と発話欄に入力して呼び出してみます。

![テスト結果](/images/blog/2017-11-09-amazon-echo-alexa-skill/exec-test.png)

できたあああああああああああああ！！

Alexaちゃんは今日はグレーのパンツを履いているようですね！！

## まとめ

とりあえず今回はスキルを作る流れだけだったので、難しいことはやりませんでしたが、大体つかめましたね。

独自のコードはほとんど書いていないのですが、Lambdaにコードを実装していくあたり今後ハマりそうな気がする。次はその辺やれたらいいな。
