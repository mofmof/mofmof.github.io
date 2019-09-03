---
layout: blog
title: Google Cloud PlatformのCloud Runに入門してみる
category: blog
tags: [GCP, Cloud Run, FaaS, Docker, Knative]
summary: mofmof社は個人サービス開発が盛んで、そこでは普段の業務ではあまり使っていないような新しい技術を使ってサービス開発してます。どうやらGCPのCloud Runというサービスが熱いということを知ったので入門してみます。
author: aharada
# image: TODO
---

mofmof社は個人サービス開発が盛んで、そこでは普段の業務ではあまり使っていないような新しい技術を使ってサービス開発してます。どうやらGCPのCloud Runというサービスが熱いということを知ったので入門してみます。

## まずは設定・準備

まずはチュートリアルをやる。英語と日本語混じりのページを読みながら進めます。

[Setting Up Your Environment Cloud Run Documentation Google Cloud](https://cloud.google.com/run/docs/setup)

> Google Cloud Platform プロジェクトを選択または作成します。

言われた通り、適当なサンプル用プロジェクトを作成します。

> ENABLE THE CLOUD RUN API

ボタンを押してAPIを有効化しようと思ったらなんか課金の有効化エラーが出た。

![課金の有効化エラー](/images/blog/2019-08-27-cloud-run-sample/bill-error.png)

![課金を有効にする](/images/blog/2019-08-27-cloud-run-sample/bill-enable.png)

よくわからないけど、ごちゃごちゃやってたら進めるようになったので次へ行く(ブログの後ろの方でちゃんと解決しました)。

> Install and initialize the Cloud SDK.

Google Cloud SDKをインストールする。

```
$ brew cask install google-cloud-sdk

==> Source [/usr/local/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/completion.zsh.inc] in your profile to enable shell command completion for gcloud.
==> Source [/usr/local/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/path.zsh.inc] in your profile to add the Google Cloud SDK command line tools to your $PATH.
```

指示された通り`.zshrc`に設定を追記します。bashの人は`.bashrc`へ。

```
$ vim .zshrc

# google-cloud-sdk
source '/usr/local/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/completion.zsh.inc'
source '/usr/local/Caskroom/google-cloud-sdk/latest/google-cloud-sdk/path.zsh.inc'
```

再読み込みさせる。

```
$ source .zshrc                                                                                                                                                             
```

使えるか確かめる。ヘルプが表示されたので問題ないようだ。

```
$ gcloud --help
```

> Install the gcloud beta component:

よくわからんがコンポーネントをインストールする。

```
$ gcloud components install beta
```

> Update components:

コンポーネントを更新する

```
$ gcloud components update
```

> Optionally, set your platform and default Cloud Run region with the gcloud properties to avoid prompts from the command line:

なんか設定値をセットするみたい。現在β版なのでリージョンは`us-central1`一択。

```
$ gcloud config set run/platform managed
$ gcloud config set run/region us-central1
```

> Optionally, install Docker locally. If you do so, you must invoke the following gcloud commands to add the gcloud Docker credential helper:

Dockerのなんかをする。よくわかってない。

```
gcloud auth configure-docker
gcloud components install docker-credential-gcr
```

## ビルド・デプロイ

準備できたっぽいので、ビルドとデプロイの手順へ進む。
[https://cloud.google.com/run/docs/quickstarts/build-and-deploy](https://cloud.google.com/run/docs/quickstarts/build-and-deploy) 

サンプルの言語はGo, Node.js, Python, Java, C#, PHP, Rubyなどが用意されているが、まあコンテナなのでなんでも動かせるのだろう。今回はPythonでやってみます。

app.py

```
import os

from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
  target = os.environ.get('TARGET', 'World')
  return 'Hello {}!\n'.format(target)

if __name__ == "__main__":
  app.run(debug=True,host='0.0.0.0',port=int(os.environ.get('PORT', 8080)))
```

Dockerfile

```
# Use the official Python image.
# https://hub.docker.com/_/python
FROM python:3.7

# Copy local code to the container image.
ENV APP_HOME /app
WORKDIR $APP_HOME
COPY . .

# Install production dependencies.
RUN pip install Flask gunicorn

# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 app:app
```

.dockerignore

```
Dockerfile
README.md
*.pyc
*.pyo
*.pyd
__pycache__
```

GCR(Google Container Registry)にコンテナをビルドしてアップしようと思ったらプロジェクトをセットしろと怒られた。コマンドにプロジェクトID入れてる設定を求められるの謎。PROJECT_IDは各自作成したプロジェクトIDに置き換えてください。

```
~/s/cloud-run-sample ❯❯❯ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
ERROR: (gcloud.builds.submit) The required property [project] is not currently set.
You may set it for your current workspace by running:

  $ gcloud config set project VALUE

or it can be set temporarily by the environment variable [CLOUDSDK_CORE_PROJECT]
```

言われた通りにします。

```
gcloud config set project PROJECT_ID
```

もう一回ビルドしてアップ。ぐぬぬ。。

```
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
ERROR: (gcloud.builds.submit) You do not currently have an active account selected.
Please run:

  $ gcloud auth login

to obtain new credentials, or if you have already logged in with a
different account:

  $ gcloud config set account ACCOUNT

to select an already authenticated account to use.
```

gcloud sdkを使って認証しておく。ブラウザが開きます。

```
gcloud auth login
```

気を取り直してもう一回。くっそおおお。

```
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
ERROR: (gcloud.builds.submit) HTTPError 403: The account for bucket "PROJECT_ID_cloudbuild" has not enabled billing.
```

これに関しては冒頭でサボったワイが悪いな。ちゃんと対応します。このエラーな。支払い情報に紐付けられるプロジェクトは5つまでの模様。

```
課金の有効化エラー
課金を有効にできるプロジェクトの上限に達しています。他のプロジェクトで課金を有効にする必要がある場合は、 [課金の割り当ての増加をリクエスト](https://support.google.com/code/contact/billing_quota_increase?hl=ja) できます。
```

お支払い > アカウント管理画面からリンクされているプロジェクトを解除できる。

![アカウント管理画面](/images/blog/2019-08-27-cloud-run-sample/account-admin.png)

再度実行。SUCCESSと出た。

```
gcloud builds submit --tag gcr.io/cloud-run-sample-251104/helloworld

~

ID                                    CREATE_TIME                DURATION  SOURCE                                                                                             IMAGES                                               STATUS
bb1cca07-ba44-4cba-96d1-c3ee1dc50e80  2019-08-27T05:25:50+00:00  24S       gs://PROJECT_ID_cloudbuild/source/1566883512.43-0f856163e06a43df8281e5b71a96adc9.tgz  gcr.io/PROJECT_ID/helloworld (+1 more)  SUCCESS
```

ようやくデプロイだぜ。なんかいろいろ聞かれますがとりあえず思考停止してyをタイプした(良い子はちゃんと読め)。

```
gcloud beta run deploy --image gcr.io/PROJECT_ID/helloworld --platform managed

~

Deploying container to Cloud Run service [helloworld] in project [cloud-run-sample-251104] region [us-central1]
✓ Deploying new service... Done.
  ✓ Creating Revision...
  ✓ Routing traffic...
  ✓ Setting IAM Policy...
Done.
Service [helloworld] revision [helloworld-00001] has been deployed and is serving traffic at https://helloworld-lhhsx6leoq-uc.a.run.app
```

URLが発行されるのでブラウザで開いてみる。Hello World!できた！

![Hello Wolrd](/images/blog/2019-08-27-cloud-run-sample/helloworld.png)

## 別の関数を生やしてみる

関数を追加してみたらどうなるだろうか。

app.py

```
@app.route('/harapan')
def hello_harapan():
    target = 'harapan'
    return 'Hello {}!\n'.format(target)
```

再びデプロイして`/harapan`にアクセスしてみる。

```
gcloud beta run deploy --image gcr.io/PROJECT_ID/helloworld --platform managed
```

Not Foundとな。

![Not Found](/images/blog/2019-08-27-cloud-run-sample/not-found.png)

コンテナのビルドとアップが必要っぽい。そりゃそうだな。

```
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
```

もっかいデプロイ。

```
gcloud beta run deploy --image gcr.io/PROJECT_ID/helloworld --platform managed
```

再び`/harapan`にアクセス。表示された！

![Hello Harapan](/images/blog/2019-08-27-cloud-run-sample/hello-harapan.png)

## まとめ

Cloud Runとは一体何か。Cloud FunctionsとかLambdaの自由度が高い版かな？と思っていたが、どちらかというとAWS Fargateや、Heroku Container Resigtryとかみたいな感じか？

マネージド環境上で、コンテナで動くアプリケーションならなんでも動かせるってことかな。ということは、Railsでもなんでも動かせるんだろう。

メリットのもう一つは、稼働時間ではなく実行時間に対して課金されるので、個人サービスや立ち上げ期のサービスなど、ユーザー利用が少ない時期のコストが最適化される点も良さげ。