---
layout: blog
title: Rasa NLUで日本語の固有表現抽出をしてみる
category: blog
tags: [Python, Machine Learning, 機械学習, 固有表現抽出, Rasa]
summary: 前回Rasa NLUを使って英語文章のintentを分類できるところまで出来たので、今度は日本語でやってみる。
author: aharada
# image: 
---

前回Rasa NLUを使って英語文章のintentを分類できるところまで出来たので、今度は日本語でやってみる。

[Rasa NLUを使って固有表現抽出器を作りたいので入門してみた](/blog/rasa-nlu-tutorial.html)

まず、`data/nlu.md`に日本語の教師データを追加する。

```
## intent:restaurant_ja
- 渋谷で美味しいイタリアンない？
- 和食食べたいんだけど、六本木におすすめある？
- 今度麻布行くんだけど、フレンチのお店教えて
``` 

こちらの記事を参考に、`mecab_tokenizer.py`を作って`config.yml`に追加。

 [https://www.ogis-ri.co.jp/otc/hiroba/technical/similar-document-search/part2.html](https://www.ogis-ri.co.jp/otc/hiroba/technical/similar-document-search/part2.html) 

```
pipeline:
  # - name: "tokenizer_whitespace"
  - name: "MecabTokenizer"
  - name: "ner_crf"
  - name: "ner_synonyms"
  - name: "intent_featurizer_count_vectors"
  - name: "intent_classifier_tensorflow_embedding"
```

学習してみる。うーん、なんかレジストリに登録しないと使えないっぽい。やり方よくわからんから別のやり方を模索するか。

```
$ rasa train nlu                                                                                                         Training NLU model...
Traceback (most recent call last):
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/registry.py", line 141, in get_component_class
    return class_from_module_path(component_name)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/utils/common.py", line 201, in class_from_module_path
    raise ImportError("Cannot retrieve class from path {}.".format(module_path))
ImportError: Cannot retrieve class from path mecab_tokenizer.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/bin/rasa", line 10, in <module>
    sys.exit(main())
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/__main__.py", line 70, in main
    cmdline_arguments.func(cmdline_arguments)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/cli/train.py", line 147, in train_nlu
    fixed_model_name=args.fixed_model_name,
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/train.py", line 363, in train_nlu
    fixed_model_name=fixed_model_name,
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/train.py", line 382, in _train_nlu_with_validated_data
    config, nlu_data_directory, _train_path, fixed_model_name="nlu"
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/train.py", line 81, in train
    trainer = Trainer(nlu_config, component_builder)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/model.py", line 148, in __init__
    components.validate_requirements(cfg.component_names)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/components.py", line 36, in validate_requirements
    component_class = registry.get_component_class(component_name)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/registry.py", line 150, in get_component_class
    "in a module.".format(component_name)
Exception: Failed to find component class for 'mecab_tokenizer'. Unknown component name. Check your configured pipeline and make sure the mentioned component is not misspelled. If you are creating your own component, make sure it is either listed as part of the `component_classes` in `rasa.nlu.registry.py` or is a proper name of a class in a module.
```

どっかで、`spaCy`が日本語対応したらしい？って噂を聞いていたので自前でクラス書くのはやめて`spaCy`のトークナイザを使う方向でやってみる。

configy.yml

```
pipeline:
  # - name: “tokenizer_whitespace”
  - name: “nlp_spacy”
  - name: “tokenizer_spacy”
  - name: “CRFEntityExtractor”
  - name: “ner_crf”
  - name: “ner_synonyms”
  - name: “intent_featurizer_count_vectors”
  - name: “intent_classifier_tensorflow_embedding”
```

`spacy`入ってねえぞと怒られた。

``` 
$ rasa train nlu 
Training NLU model...
2019-07-09 13:13:36 WARNING  rasa.nlu.registry  - DEPRECATION warning: your nlu config file contains old style component name `nlp_spacy`, you should change it to its class name: `SpacyNLP`.
2019-07-09 13:13:36 WARNING  rasa.nlu.registry  - DEPRECATION warning: your nlu config file contains old style component name `tokenizer_spacy`, you should change it to its class name: `SpacyTokenizer`.
2019-07-09 13:13:36 WARNING  rasa.nlu.registry  - DEPRECATION warning: your nlu config file contains old style component name `ner_crf`, you should change it to its class name: `CRFEntityExtractor`.
2019-07-09 13:13:36 WARNING  rasa.nlu.registry  - DEPRECATION warning: your nlu config file contains old style component name `ner_synonyms`, you should change it to its class name: `EntitySynonymMapper`.
2019-07-09 13:13:36 WARNING  rasa.nlu.registry  - DEPRECATION warning: your nlu config file contains old style component name `intent_featurizer_count_vectors`, you should change it to its class name: `CountVectorsFeaturizer`.
2019-07-09 13:13:36 WARNING  rasa.nlu.registry  - DEPRECATION warning: your nlu config file contains old style component name `intent_classifier_tensorflow_embedding`, you should change it to its class name: `EmbeddingIntentClassifier`.
Traceback (most recent call last):
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/bin/rasa", line 10, in <module>
    sys.exit(main())
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/__main__.py", line 70, in main
    cmdline_arguments.func(cmdline_arguments)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/cli/train.py", line 147, in train_nlu
    fixed_model_name=args.fixed_model_name,
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/train.py", line 363, in train_nlu
    fixed_model_name=fixed_model_name,
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/train.py", line 382, in _train_nlu_with_validated_data
    config, nlu_data_directory, _train_path, fixed_model_name="nlu"
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/train.py", line 81, in train
    trainer = Trainer(nlu_config, component_builder)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/model.py", line 148, in __init__
    components.validate_requirements(cfg.component_names)
  File "/Users/atsushiharada/.local/share/virtualenvs/rasa-sample-RFdosdgb/lib/python3.7/site-packages/rasa/nlu/components.py", line 47, in validate_requirements
    + "Please install {}".format(", ".join(failed_imports))
Exception: Not all required packages are installed. To use this pipeline, you need to install the missing dependencies. Please install spacy
```

`spaCy`をインスコします。`pip`を使っている人は普通に`pip install`でOK。

```
$ pipenv install spacy --skip-lock
```

`spaCy`の日本語モデルを取ってくる必要があるっぽいのでコマンド叩いてみるが。そんなものはないと怒られる。どないやねん。

```
$ python -m spacy download ja
✘ No compatible model found for 'ja' (spaCy v2.1.4).
```

`spaCy`の公式を参照する。

[Models & Languages · spaCy Usage Documentation](https://spacy.io/usage/models)

none yetだと？

どうやら`GiNZA`という日本語自然言語処理ライブラリが`spaCy`で動かせる形で配布されているらしいのでインスコする。

```
$ pipenv install "https://github.com/megagonlabs/ginza/releases/download/latest/ginza-latest.tar.gz" --skip-lock
Installing https://github.com/megagonlabs/ginza/releases/download/latest/ginza-latest.tar.gz…
Adding ginza to Pipfile's [packages]…
✔ Installation Succeeded
Installing dependencies from Pipfile…
  🐍   ▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉▉ 4/4 — 00:00:0
```

cofing.ymlの言語指定を修正

```
language: ja_ginza
```

できたーーー！！

```
$ rasa train nlu
....

$ rasa shell nlu
2019-07-09 13:46:44 INFO     rasa.nlu.components  - Added 'SpacyNLP' to component cache. Key 'SpacyNLP-ja_ginza'.
NLU model loaded. Type a message and press enter to parse it.
Next message:
六本木のイタリアン教えて
{
  "intent": {
    "name": "restaurant_ja",
    "confidence": 0.9044981598854065
  },
  "entities": [],
  "intent_ranking": [
    {
      "name": "restaurant_ja",
      "confidence": 0.9044981598854065
    },
    {
      "name": "mood_great",
      "confidence": 0.17773786187171936
    },
    {
      "name": "greet",
      "confidence": 0.07361753284931183
    },
    {
      "name": "goodbye",
      "confidence": 0.055908896028995514
    },
    {
      "name": "affirm",
      "confidence": 0.0
    },
    {
      "name": "restaurant",
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
  "text": "\u516d\u672c\u6728\u306e\u30a4\u30bf\u30ea\u30a2\u30f3\u6559\u3048\u3066"
}
```

## 文章から固有表現抽出してみる
`data/nlu.md`の日本語教師データを以下のように編集する。

```
## intent:restaurant_ja
- [渋谷](location)で美味しい[イタリアン](restaurant_type)ない？
- [和食](restaurant_type)食べたいんだけど、[六本木](location)におすすめある？
- 今度[麻布](location)行くんだけど、[フレンチ](restaurant_type)のお店教えて
```

出来たーーー！

```
$ rasa train nlu
....

$ rasa shell nlu

Next message:
渋谷で美味しいイタリアンない？
{
  "intent": {
    "name": "restaurant_ja",
    "confidence": 0.9514173865318298
  },
  "entities": [
    {
      "start": 0,
      "end": 2,
      "value": "\u6e0b\u8c37",  // 渋谷
      "entity": "location", 
      "confidence": 0.727149998248404,
      "extractor": "CRFEntityExtractor"
    },
    {
      "start": 7,
      "end": 12,
      "value": "\u30a4\u30bf\u30ea\u30a2\u30f3",  // イタリアン
      "entity": "restaurant_type",
      "confidence": 0.7463336276563652,
      "extractor": "CRFEntityExtractor"
    },
    {
      "start": 0,
      "end": 2,
      "value": "\u6e0b\u8c37",
      "entity": "location",
      "confidence": 0.727149998248404,
      "extractor": "CRFEntityExtractor"
    },
    {
      "start": 7,
      "end": 12,
      "value": "\u30a4\u30bf\u30ea\u30a2\u30f3",
      "entity": "restaurant_type",
      "confidence": 0.7463336276563652,
      "extractor": "CRFEntityExtractor"
    }
  ],
  "intent_ranking": [
    {
      "name": "restaurant_ja",
      "confidence": 0.9514173865318298
    },
    {
      "name": "greet",
      "confidence": 0.09692550450563431
    },
    {
      "name": "mood_great",
      "confidence": 0.06003577262163162
    },
    {
      "name": "mood_unhappy",
      "confidence": 0.02446659654378891
    },
    {
      "name": "deny",
      "confidence": 0.01749199628829956
    },
    {
      "name": "affirm",
      "confidence": 0.0
    },
    {
      "name": "check_balance",
      "confidence": 0.0
    },
    {
      "name": "restaurant",
      "confidence": 0.0
    },
    {
      "name": "goodbye",
      "confidence": 0.0
    }
  ],
  "text": "\u6e0b\u8c37\u3067\u7f8e\u5473\u3057\u3044\u30a4\u30bf\u30ea\u30a2\u30f3\u306a\u3044\uff1f"
}
```

出来たはいいが、なぜかユニコードで表記されていて直接読めない。UTF-16BEで変換すれば日本語として読めるのだけど、できれば最初から日本語で表示されて欲しいところ。

まあ実際には受け取ったプログラム側でも制御できるしとりあえずいいか。

## ソースコード
GitHubにアップしてあるので参照くださいませ。

[GitHub - harada4atsushi/rasa-sample](https://github.com/harada4atsushi/rasa-sample)
