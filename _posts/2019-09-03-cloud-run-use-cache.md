---
layout: blog
title: GCPのCloud Runでキャッシュを使ってビルドしたい
category: blog
tags: [GCP, Cloud Run, Docker]
summary: 前回Cloud Runに入門してHello Worldができるところまでやりました。思い出す意味でも一度ビルドとデプロイをやりなおしてみる。あとビルドの履歴はCloud Buildの画面を見に行けばわかる。
author: aharada
# image: TODO
---

前回Cloud Runに入門してHello Worldができるところまでやりました。

[Google Cloud PlatformのCloud Runに入門してみる](/blog/cloud-run-sample.html)

思い出す意味でも一度ビルドとデプロイをやりなおしてみる。あとビルドの履歴はCloud Buildの画面を見に行けばわかる。

[https://console.cloud.google.com/cloud-build/builds](https://console.cloud.google.com/cloud-build/builds) 

ビルドする。エラー、なんぞ？

```
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld

~

denied: Token exchange failed for project 'PROJECT_ID'. Caller does not have permission 'storage.buckets.get'. To configure permissions, follow instructions at: https://cloud.google.com/container-registry/docs/access-control
```

うーん。Cloud Stroageのbucketへのget権限がないっすよ、的なエラーぽい。前回は特に設定とかせずに動いた気がする。認証らへんが通ってないのかなと当たりをつけて適当にコマンドたたく。

```
$ gcloud auth configure-docker
$ gcloud components install docker-credential-gcr
$ gcloud config set project PROJECT_ID
$ gcloud auth login
```

再びビルドするとちゃんと成功したぞ。

```
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld
```

## キャッシュを使ってビルドする

公式のドキュメントはこちら

[https://cloud.google.com/cloud-build/docs/speeding-up-builds?hl=ja](https://cloud.google.com/cloud-build/docs/speeding-up-builds?hl=ja) 

`--cache-from`をつけたらいいんだよ的なことが書いてあったのでやってみたが、そんなオプションはねえ、とのこと。

```
$ gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld --cache-from
ERROR: (gcloud.builds.submit) unrecognized arguments: --cache-from (did you mean '--no-cache'?)
```

ドキュメントをよく読むとなんかymlで設定ファイルを書いておいておけば良いらしい。気をつける必要があるのは、公式にのっているymlのインデントがめちゃくちゃです。そのまま実行するとエラーになるのでインデント直しましょう。

どないやねんって思ったけどまあβ版だし、しゃーないか。

```
$ gcloud builds submit --config cloudbuild.yml .                                                                                                         ✘ 1
ERROR: (gcloud.builds.submit) parsing cloudbuild.yml: while parsing a block mapping
  in "cloudbuild.yml", line 1, column 1
expected <block end>, but found u'-'
  in "cloudbuild.yml", line 8, column 1
```

修正後の`cloudbuild.yml`がこちら。

```
steps:
  - name: 'gcr.io/cloud-builders/docker'
    entrypoint: 'bash'
    args:
      - '-c'
      - |
        docker pull gcr.io/PROJECT_ID/helloworld:latest || exit 0
  - name: 'gcr.io/cloud-builders/docker'
    args: [
            'build',
            '-t', 'gcr.io/PROJECT_ID/helloworld:latest',
            '--cache-from', 'gcr.io/PROJECT_ID/helloworld:latest',
            '.'
          ]
images: ['gcr.io/PROJECT_ID/helloworld:latest']
```

ビルドが通ればOK。

```
$ gcloud builds submit --config cloudbuild.yml .                                                                                                         ✘ 1
```


## どれくらい早いのか計測してみる。

timeコマンドで計測してみます。まずキャッシュを使った場合。19.022秒。

```
$ time gcloud builds submit --config cloudbuild.yml .

~

0.34s user 0.13s system 2% cpu 19.022 total
```

続いてキャッシュを使わない場合。24.117秒。

```
$ time gcloud builds submit --tag gcr.io/PROJECT_ID/helloworld

~

0.35s user 0.13s system 1% cpu 24.117 total
```

まあ5秒くらい早くなったのだろうか。何度か試してみると、ほとんど差がでないときもある。今回のDockerfileはかなり軽量だったから差が出にくかったかな？あるいはキャッシュ使ってくれてないとか。わからん。


## おまけ: Cloud Runに割り当てるメモリサイズを変更する方法

Cloud Runで新規にサービス作成するときにメモリサイズを選択できるのですが、後から変更する項目が画面に見当たらず困った。どうやらコマンドで変更できる模様なのでメモっておく(2Gに変更している例)。

```
$ gcloud beta run services update --memory 2G
```

Herokuだとメモリ1GBにすると$50/月の費用がかかるが、Cloud Runは処理時間に対して費用がかかるので、呼び出しが少なければ全然費用がかからない。厳密にはよくわからないが、アクセスが非常に少ないので2GBにしても無料で使えるみたい。すごーい。
