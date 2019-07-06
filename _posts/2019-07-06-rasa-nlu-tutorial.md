---
layout: blog
title: Rasa NLUを使って固有表現抽出器を作りたいので入門してみた
category: blog
tags: [Python, Machine Learning, 機械学習, 固有表現抽出, Rasa]
summary: Rasa-xなるよく分からんもんをインストールしろと書いてあるけどいらなそうなのでrasaだけインスコします。
author: aharada
# image: 
---

そういえば最近Mac Book Proを新しくしたのでPython環境が整っていなかった。今まではpyenv + pyenv-virtualenvでバージョン管理してたんですが、最近ではPython公式がPipenvを推してるらしいのでpyenv + pipenvという構成にしてみた。

[Macにpipenv環境作ってみた](https://qiita.com/MasanoriIwakura/items/e9011e49bd225553ff47)

```
$ mkdir rasa-sample
$ cd rasa-sample
$ pipenv install
```

ぼくの環境ではpyenvでpython3.7.3を先にインストールしていたので、pipenvの仮想環境も3.7.3で構築されたみたい。

## 公式のチュートリアル動かしてみる

公式ドキュメント

[Installation](https://rasa.com/docs/rasa/user-guide/installation/)

Rasa-xなるよく分からんもんをインストールしろと書いてあるけどいらなそうなのでrasaだけインスコします。

--skip-lockはPipfile.lockを更新しないオプションです。これをつけないとパッケージ一個インストールするのに Locking... って表示されて数分待たせてしまう。

チーム開発するときは必ず`pipenv lock`してからpushしましょう。

```
$ pipenv install rasa --skip-lock
```

pipenvを使う場合、仮想環境に入る必要があるっぽい。これでrasaコマンドとかも使えるようになる。

```
$ pipenv shell
```

公式のチュートリアル

[Rasa Tutorial](https://rasa.com/docs/rasa/user-guide/rasa-tutorial/)

だーっとファイルが生成されます。rasaを使うために必要なファイルやデータファイルが一通り生成される模様。

```
$ rasa init --no-prompt
```

```
$ tree .
.
├── Pipfile
├── Pipfile.lock
├── __init__.py
├── actions.py
├── config.yml
├── credentials.yml
├── data
│   ├── nlu.md
│   └── stories.md
├── domain.yml
├── endpoints.yml
├── models
│   ├── 20190706-102404.tar.gz
└── rasa_core.log
```

教師データっぽいのをのぞいてみる。markdownで書かれてるのが面白い。

```
$ cat data/nlu.md
## intent:greet
- hey
- hello
- hi
- good morning
- good evening
- hey there

## intent:goodbye
- bye
- goodbye
- see you around
- see you later

## intent:affirm
- yes
- indeed
- of course
- that sounds good
- correct

## intent:deny
- no
- never
- I don't think so
- don't like that
- no way
- not really

## intent:mood_great
- perfect
- very good
- great
- amazing
- wonderful
- I am feeling very good
- I am great
- I'm good

## intent:mood_unhappy
- sad
- very sad
- unhappy
- bad
- very bad
- awful
- terrible
- not very good
- extremely sad
- so sad%
```

なんかよくわからんけど会話できた。すげえ。

```
$ rasa shell
2019-07-06 10:26:27 INFO     root  - Starting Rasa Core server on http://localhost:5005
Bot loaded. Type a message and press enter (use '/stop' to exit):
Your input ->  hello
Hey! How are you?
```

Intent分類と固有表現抽出をしたかったので、別に会話できる必要はないんだが。

## Rasa NLUのみを動かす

公式ドキュメントに書いてあった。

 [https://rasa.com/docs/rasa/nlu/using-nlu-only/](https://rasa.com/docs/rasa/nlu/using-nlu-only/) 

これでNLUだけ動かせる。`hello! nice to meet you.`と入れたらintent:greetであると判別された。

```
$ rasa shell nlu
NLU model loaded. Type a message and press enter to parse it.
Next message:
hello! nice to meet you.
{
  "intent": {
    "name": "greet",
    "confidence": 0.656333863735199
  },
  "entities": [],
  "intent_ranking": [
    {
      "name": "greet",
      "confidence": 0.656333863735199
    },
    {
      "name": "goodbye",
      "confidence": 0.5116747617721558
    },
    {
      "name": "mood_unhappy",
      "confidence": 0.026658378541469574
    },
    {
      "name": "affirm",
      "confidence": 0.0
    },
    {
      "name": "deny",
      "confidence": 0.0
    },
    {
      "name": "mood_great",
      "confidence": 0.0
    }
  ],
  "text": "hello! nice to meet you."
}
Next message:
```

ふむ大体のカラクリはわかった。今度は飲食店のおすすめを聞くintent分類と固有表現抽出をしてみたい。

data/rlu.mdに下記を追加。

```
## intent:restaurant
- [渋谷](location)で美味しい[イタリアン](restaurant_type)ない？
- [和食](restaurant_type)食べたいんだけど、[六本木](location)におすすめある？
- 今度[麻布](location)行くんだけど、[フレンチ](restaurant_type)のお店教えて
```

config.ymlをちょっと修正。

こちらを参考にした。

 [https://www.ogis-ri.co.jp/otc/hiroba/technical/similar-document-search/part2.html](https://www.ogis-ri.co.jp/otc/hiroba/technical/similar-document-search/part2.html) 

残念ながらこの辺はまだあんまり理解できてないが、固有表現抽出するにあたってどんなモジュールを使って処理するかを選択してる。これ変えるだけで色々調整できるのは素敵。

```
# Configuration for Rasa NLU.
# https://rasa.com/docs/rasa/nlu/components/
language: en
# pipeline: supervised_embeddings

pipeline:
  - name: "tokenizer_whitespace"
  - name: "ner_crf"
  - name: "ner_synonyms"
  - name: "intent_featurizer_count_vectors"
  - name: "intent_classifier_tensorflow_embedding"

# Configuration for Rasa Core.
# https://rasa.com/docs/rasa/core/policies/
policies:
  - name: MemoizationPolicy
  - name: KerasPolicy
  - name: MappingPolicy

```

学習して試してみる。

```
$ rasa train nlu
```

ぴゃーーー

```
$ rasa shell nlu
NLU model loaded. Type a message and press enter to parse it.
Next message:
渋谷の美味しい和食
{
  "intent": {
    "name": null,
    "confidence": 0.0
  },
  "entities": [],
  "intent_ranking": [],
  "text": "\u6e0b\u8c37\u306e\u7f8e\u5473\u3057\u3044\u548c\u98df"
}
```

日本語対応するにはどっかしらいじらないといけないようだ。仕方ないので今回は英語で。

data/nlu.md

```
**## intent:restaurant**
- Isn’t it delicious Italian in Shibuya?
- I would like to eat Japanese food, but is it recommended for Roppongi?
- I’m going to Azabu next time, but tell me a French shop
```

```
$ rasa train nlu
```

ちゃんと`intent: restaurant`と認識しましたね！

```
$ rasa shell nlu
NLU model loaded. Type a message and press enter to parse it.
Next message:
I want to eat Italian in Shibuya
{
  "intent": {
    "name": "restaurant",
    "confidence": 0.9497830271720886
  },
  "entities": [],
  "intent_ranking": [
    {
      "name": "restaurant",
      "confidence": 0.9497830271720886
    },
    {
      "name": "affirm",
      "confidence": 0.09771481156349182
    },
    {
      "name": "goodbye",
      "confidence": 0.08525433391332626
    },
    {
      "name": "greet",
      "confidence": 0.05800335854291916
    },
    {
      "name": "mood_great",
      "confidence": 0.0
    },
    {
      "name": "deny",
      "confidence": 0.0
    },
    {
      "name": "mood_unhappy",
      "confidence": 0.0
    }
  ],
  "text": "I want to eat Italian in Shibuya"
}
```

次回は日本語対応してみたいと思います。