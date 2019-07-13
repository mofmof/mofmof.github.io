---
layout: blog
title: Heroku Container RegistryでPython + Rasa NLUを動かす
category: blog
tags: [Python, Machine Learning, 機械学習, 固有表現抽出, Rasa, Docker, Heroku]
summary: Herokuならサーバを意識する必要がないので楽チンなので、固有表現抽出のRasa NLUをHerokuで動かしてみたいと思います。
author: aharada
# image: 
---

以前にHeroku上でPythonで動かしている機械学習のAPIを立てたことがあった。Herokuならサーバを意識する必要がないので楽チンなので、固有表現抽出のRasa NLUをHerokuで動かしてみたいと思います。

## ローカルのdockerで動くようにする

Mac Book Proを買い換えたばかりでDockerが入ってなかったのでDockerをインストールするところから。

```
$ brew cask install docker
```

Rasa公式にDockerで動かす方法が書いてあるが、これが罠だということは知る由もなかった。

[Running Rasa with Docker](https://rasa.com/docs/rasa/user-guide/running-rasa-with-docker/#installing-docker)

適当にプロジェクトを作ります。

```
$ mkdir rasa-sample-docker
$ cd rasa-sample-docker
```

docker runを経由してrasa initする。

```
$ docker run -v **$(**pwd**)**:/app rasa/rasa init —no-prompt
```

docker imageのダウンロードもあるが長い時間待たされる。出来上がったディレクトリ構造は前回やったものと同じ。

```
$ tree .
.
├── __init__.py
├── __pycache__
│   ├── __init__.cpython-36.pyc
│   └── actions.cpython-36.pyc
├── actions.py
├── config.yml
├── credentials.yml
├── data
│   ├── nlu.md
│   └── stories.md
├── domain.yml
├── endpoints.yml
└── models
    └── 20190713-001000.tar.gz
```

rasa shellを実行してみる。とりあえず動いた。

```
docker run -it -v $(pwd):/app rasa/rasa shell
2019-07-13 00:15:01 INFO     root  - Connecting to channel 'cmdline' which was specified by the '--connector' argument. Any other channels will be ignored. To connect to all given channels, omit the '--connector' argument.
2019-07-13 00:15:01 INFO     root  - Starting Rasa Core server on http://localhost:5005
Bot loaded. Type a message and press enter (use '/stop' to exit):
Your input ->  hello
Hey! How are you?
```

## docker-composeで動くようにする

dockerのコマンドを直接打って使っていくスタイルはしんどいのでdocker composeを使いたい。設定していきます。

```
version: ‘3.0’
services:
  rasa:
    image: rasa/rasa:latest-full
    ports:
      - 5005:5005
    volumes:
      - ./:/app
    command:
      - run
```


```
$ docker-compose up
```

クッソ待たされるが、立ち上がったぽいのでcurlでリクエスト投げてみる。うまくいかないのでエンドポイント変えたりしてみたけど成功しない。。

```
$ curl -X POST localhost:5005/parse \
  -d '{"q":hello"}'
Error: Requested URL /parse not found%                                                   ~/s/rasa-sample-docker ❯❯❯ curl -X POST localhost:5005/parse \
  -d '{"q":hello"}'
Error: Requested URL /parse not found%                                                   ~/s/rasa-sample-docker ❯❯❯ curl -X POST localhost:5005/parse \
  -d '{"q":hello"}'
Error: Requested URL /parse not found%                                                   ~/s/rasa-sample-docker ❯❯❯ curl -X POST localhost:5005 \
  -d '{"q":hello"}'
Error: Requested URL / not found%                                                        ~/s/rasa-sample-docker ❯❯❯ curl -X POST localhost:5005/parse \
  -d '{"q":hello"}'
Error: Requested URL /parse not found%                                                   ~/s/rasa-sample-docker ❯❯❯ curl localhost:5005/model/parse -d '{"text":"hello"}'
Error: Requested URL /model/parse not found%                                             ~/s/rasa-sample-docker ❯❯❯ curl localhost:5005/model/webhooks/rest/webhook -d '{"text":"hello"}'
curl: (7) Failed to connect to localhost port 5005: Connection refused
~/s/rasa-sample-docker ❯❯❯ curl localhost:5005/model/webhooks/rest/webhook -d '{"text":"hello"}'
Error: Requested URL /model/webhooks/rest/webhook not found%                             ~/s/rasa-sample-docker ❯❯❯ curl localhost:5005/webhooks/rest/webhook -d '{"text":"hello"}'
[]%
```

commandが`run`になっているので、前回やってように`rasa run --enable-api`という風にやるのと違う模様。

公式のDockerfileを眺めてヒントを探す。

[Docker Hub](https://hub.docker.com/r/rasa/rasa_nlu/dockerfile)

どうやら`entrypoint.sh`がENDPOINTに指定されているため、それが動いているみたい。シェルの中身をみると、コマンドはstart, run, trainに限定されている。

[https://github.com/RasaHQ/rasa_core/blob/master/entrypoint.sh](https://github.com/RasaHQ/rasa_core/blob/master/entrypoint.sh)

docker-compose.ymlを変更したりして色々試す。

```
    command:
      - rasa
      - shell
```

```
$ docker-compose up
Starting rasa-sample-docker_rasa_nlu_1 ... done
Attaching to rasa-sample-docker_rasa_nlu_1
rasa_nlu_1  | Traceback (most recent call last):
rasa_nlu_1  |   File "/usr/local/bin/rasa", line 6, in <module>
rasa_nlu_1  |     from pkg_resources import load_entry_point
rasa_nlu_1  |   File "/usr/local/lib/python3.6/site-packages/pkg_resources/__init__.py", line 3191, in <module>
rasa_nlu_1  |     @_call_aside
rasa_nlu_1  |   File "/usr/local/lib/python3.6/site-packages/pkg_resources/__init__.py", line 3175, in _call_aside
rasa_nlu_1  |     f(*args, **kwargs)
rasa_nlu_1  |   File "/usr/local/lib/python3.6/site-packages/pkg_resources/__init__.py", line 3204, in _initialize_master_working_set
rasa_nlu_1  |     working_set = WorkingSet._build_master()
rasa_nlu_1  |   File "/usr/local/lib/python3.6/site-packages/pkg_resources/__init__.py", line 583, in _build_master
rasa_nlu_1  |     ws.require(__requires__)
rasa_nlu_1  |   File "/usr/local/lib/python3.6/site-packages/pkg_resources/__init__.py", line 900, in require
rasa_nlu_1  |     needed = self.resolve(parse_requirements(requirements))
rasa_nlu_1  |   File "/usr/local/lib/python3.6/site-packages/pkg_resources/__init__.py", line 786, in resolve
rasa_nlu_1  |     raise DistributionNotFound(req, requirers)
rasa_nlu_1  | pkg_resources.DistributionNotFound: The 'rasa' distribution was not found and is required by the application
rasa-sample-docker_rasa_nlu_1 exited with code 1
```

わけが分からないので一度Dockerコンテナの中に入ってみる。

```
$ docker-compose run rasa_nlu bash
```

rasa-core-sdkしか入ってないみたい。

```
root@5258eb52fcb6:/app# pip list | grep rasa
rasa-core-sdk           0.13.0
```

rasaをインストールしてみると、コンテナの中ではrasaコマンドが使えるようになった。

```
root@5258eb52fcb6:/app# pip install rasa
root@5258eb52fcb6:/app# pip list | grep rasa
rasa                    1.1.6       /usr/local/lib/python3.6/site-packages
rasa-core-sdk           0.13.0
rasa-sdk                1.1.0
```

公式のイメージを使うと、rasaコマンドが正しく入らないっぽい(あとから分かったけど、公式のイメージではpip install rasaをせず、GitHubの[rasaのリポジトリ](https://github.com/RasaHQ/rasa)を丸々持ってきているため、rasaコマンドではなくソースコードを直接実行されることを想定しているっぽい)。

## 公式のDockerイメージはやめてDockerfileを自作する

公式のDockerイメージではrasaコマンドをうまく叩けなかったのでDockerfileから自作することにする。

Dockerfile

```
FROM python:3.6-slim

ENV RASA_HOME=/app

RUN apt-get update -qq \
    && apt-get install -y --no-install-recommends build-essential git-core \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR ${RASA_HOME}
COPY . ${RASA_HOME}

RUN pip install -r requirements.txt

CMD rasa run --enable-api -p ${PORT}
```

docker-compose.yml

```
version: '3.0'
services:
  rasa_nlu:
    build: .
    ports:
      - 5005:5005
    volumes:
      - ./:/app
    command:
      - rasa
      - run
      - --enable-api
```

起動してみる。

```
$ docker-compose up
```

curlでリクエストしてみたら、期待通りに動いたー！！

Rasa NLUの正しいエンドポイントとリクエストの仕方は公式に載ってた。

[https://rasa.com/docs/rasa/nlu/using-nlu-only/](https://rasa.com/docs/rasa/nlu/using-nlu-only/)


```
curl localhost:5005/model/parse -d '{"text":"hello"}'
{"intent":{"name":"greet","confidence":0.9535727501},"entities":[],"intent_ranking":[{"name":"greet","confidence":0.9535727501},{"name":"goodbye","confidence":0.0813569427},{"name":"deny","confidence":0.0},{"name":"mood_unhappy","confidence":0.0},{"name":"mood_great","confidence":0.0},{"name":"affirm","confidence":0.0}],"text":"hello"}%
```

## Heroku Container Registyにデプロイ

[Container Registry & Runtime](https://devcenter.heroku.com/articles/container-registry-and-runtime)


Heroku Container RegistyはHeroku上でDockerコンテナを動かせるサービス。これを使えばHerokuにpushする容量制限などなく使えるので、機械学習系のライブラリを入れても動かせる(ような気がする)。

```
$ heroku container:login
Login Succeeded
```

Heroku上にアプリケーションを作る。

```
$ heroku create rasa-sample-docker
Creating ⬢ rasa-sample-docker... done
```

pushするとdockerコンテナのbuildが始まる。

```
$ heroku container:push web
```

上り回線速度の問題か、1時間くらい待たされた。つらい。オレはあと何回dockerのbuildを眺めれば良いのか。ここまで到達するのにdockerのbuildを20回くらい待っている。dockerが嫌いになりそうだ。

pushが終わったらreleaseします。

[Heroku Container Registryのcontainer:push後のcontainer:releaseの対応方法](https://casualdevelopers.com/tech-tips/how-to-adapt-new-release-process-on-heroku-container-registry/)

```
$ heroku container:release web
Releasing images web to rasa-sample-docker... done
```

リクエストしてみる。

```
$ curl https://rasa-sample-docker.herokuapp.com/model/parse -d '{"text":"hello"}'
```

と言いたいところだが、`heroku container:push`するたびに1時間くらい待たされるので、もうﾏﾁﾞﾑﾘ。たぶん動くだろう。