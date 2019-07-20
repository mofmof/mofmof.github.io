---
layout: blog
title: Rasa NLUをPythonのコードから呼び出す方法
category: blog
tags: [Python, Machine Learning, 機械学習, 固有表現抽出, Rasa]
summary: そこそこな回数の固有表現抽出処理を連続で行いたいというニーズがあったので、1リクエストで複数の固有表現抽出処理を行うAPIを追加したい。
author: aharada
# image: 
---

RasaNLUはAPIサーバが内包されているので、単純に固有表現抽出するAPIは最初からある。だがしかし、今回そこそこな回数の固有表現抽出処理を連続で行いたいというニーズがあったので、1リクエストで複数の固有表現抽出処理を行うAPIを追加したい。

たぶんFlaskで別にAPIサーバを起動すれば独自のAPIは作れると思うんだけど、pythonのコードからRasa NLUを呼び出す必要があって、どうやればいいかわからなかったので色々試してみた。


## Rasaのソースコードを読む

[https://github.com/RasaHQ/rasa](https://github.com/RasaHQ/rasa)

`rasa shell nlu`でコマンドラインでの固有表現抽出が出来るので、そのあたりのソースコードを読んで試してみた。

app/run.py

```
from rasa.nlu.model import Interpreter

interpreter = Interpreter.load(‘models’)
result = interpreter.parse(‘hello!’)

print(result)
```

実行すると、metadata.jsonがねえぞと怒られる。

```
$ python -m app.run

Traceback (most recent call last):
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/rasa/utils/io.py", line 129, in read_file
    with open(filename, encoding=encoding) as f:
FileNotFoundError: [Errno 2] No such file or directory: 'models/metadata.json'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/rasa/nlu/model.py", line 68, in load
    data = rasa.utils.io.read_json_file(metadata_file)
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/rasa/utils/io.py", line 137, in read_json_file
    content = read_file(filename)
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/rasa/utils/io.py", line 132, in read_file
    raise ValueError("File '{}' does not exist.".format(filename))
ValueError: File 'models/metadata.json' does not exist.

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/lib/python3.7/runpy.py", line 193, in _run_module_as_main
    "__main__", mod_spec)
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/lib/python3.7/runpy.py", line 85, in _run_code
    exec(code, run_globals)
  File "/Users/atsushiharada/source/rasa-sample-docker/app/run.py", line 5, in <module>
    interpreter = Interpreter.load('models')
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/rasa/nlu/model.py", line 298, in load
    model_metadata = Metadata.load(model_dir)
  File "/Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/rasa/nlu/model.py", line 73, in load
    "Failed to load model metadata from '{}'. {}".format(abspath, e)
rasa.nlu.model.InvalidModelError: Failed to load model metadata from '/Users/atsushiharada/source/rasa-sample-docker/models/metadata.json'. File 'models/metadata.json' does not exist.
```

metadata.jsonなってファイルは知らんので、いろいろググってみたらありかを見つけた。

[Metadata not found when using Rasa NLU from Python - Stack Overflow](https://stackoverflow.com/questions/56713641/metadata-not-found-when-using-rasa-nlu-from-python)

trainしたあとに`models/20190713-001000.tar.gz`のようなファイルとかが生成されるんだけど、これを展開すると中に入ってた。

```
$ tree models/20190713-001000
models/20190713-001000
├── core
│   ├── domain.json
│   ├── domain.yml
│   ├── metadata.json
│   ├── policy_0_MemoizationPolicy
│   │   ├── featurizer.json
│   │   └── memorized_turns.json
│   ├── policy_1_KerasPolicy
│   │   ├── featurizer.json
│   │   ├── keras_model.h5
│   │   ├── keras_policy.json
│   │   └── keras_policy.tf_config.pkl
│   └── policy_2_MappingPolicy
│       └── mapping_policy.json
├── fingerprint.json
└── nlu
    ├── checkpoint
    ├── component_1_RegexFeaturizer.pkl
    ├── component_4_CountVectorsFeaturizer.pkl
    ├── component_5_EmbeddingIntentClassifier.ckpt.data-00000-of-00001
    ├── component_5_EmbeddingIntentClassifier.ckpt.index
    ├── component_5_EmbeddingIntentClassifier.ckpt.meta
    ├── component_5_EmbeddingIntentClassifier_encoded_all_intents.pkl
    ├── component_5_EmbeddingIntentClassifier_inv_intent_dict.pkl
    ├── metadata.json
    └── training_data.json
```

展開されたディレクトリの下のnluディレクトリに入っていることがわかる。

このあたりのソースコードを読むと、実行時にmodelのtar.gzをtempfileを使って一時ディレクトリに展開しているのがわかる。

[rasa/model.py at 2d7092536aaadc130a33374c515b8529505b8b81 · RasaHQ/rasa · GitHub](https://github.com/RasaHQ/rasa/blob/2d7092536aaadc130a33374c515b8529505b8b81/rasa/model.py#L92)

ので、こんな感じに修正。

app/run.py

```
from rasa.nlu.model import Interpreter
from rasa.model import get_model, get_model_subdirectories

model_path = get_model()
interpreter = Interpreter.load(model_path + ‘/nlu’)
result = interpreter.parse(‘hello!’)

print(result)
```

やったあ動いた！

```
$ python -m app.run
python running!
2019-07-20 09:17:58.065753: I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 FMA
WARNING:tensorflow:From /Users/atsushiharada/.pyenv/versions/3.7.3/envs/rasa-sample/lib/python3.7/site-packages/tensorflow/python/training/saver.py:1266: checkpoint_exists (from tensorflow.python.training.checkpoint_management) is deprecated and will be removed in a future version.
Instructions for updating:
Use standard file APIs to check for files with this prefix.
{'intent': {'name': 'greet', 'confidence': 0.9535727500915527}, 'entities': [], 'intent_ranking': [{'name': 'greet', 'confidence': 0.9535727500915527}, {'name': 'goodbye', 'confidence': 0.081356942653656}, {'name': 'deny', 'confidence': 0.0}, {'name': 'mood_unhappy', 'confidence': 0.0}, {'name': 'mood_great', 'confidence': 0.0}, {'name': 'affirm', 'confidence': 0.0}], 'text': 'hello!'}
```