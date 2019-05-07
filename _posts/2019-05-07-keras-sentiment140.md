---
layout: blog
title: KerasでSentiment140のネガポジ分類をCNNで解こうとして挫折した
category: blog
tags: [機械学習,machine learning,TensorFlow,Keras,Sentiment140]
summary: 前回、KerasでMNISTの手書き数字をCNNで分類する問題をやってみたので、今度は自然言語処理らへんをKerasで動かしてみたいと思った。
author: aharada
# image:
---

前回、KerasでMNISTの手書き数字をCNNで分類する問題をやってみたので、今度は自然言語処理らへんをKerasで動かしてみたいと思った。

[KerasでMNIST分類問題をCNNで解く](/blog/keras-mnist-cnn.html)

同様にKerasのリポジトリのサンプルコードを眺めていたら`imdb_snn.py`というのを見つけた。IMDbデータセットは、映画に関するデータで、映画のレビューテキストとかが入っていたはず。こいつをそのまま使えばKerasでとりあえず自然言語処理タスクが動くのでは？と思ったのでやってみました。

[https://github.com/keras-team/keras/tree/master/examples](https://github.com/keras-team/keras/tree/master/examples)

が、結果断念したので、断念までのログを書いておきます。

## データセット

機械学習で何かやってみようと思うと、まずはデータが必要になります。`arXivTimes/datasets`を見るとよく使われているデータセットが日本語で紹介されているのでオススメ。

[arXivTimes/datasets at master · arXivTimes/arXivTimes · GitHub](https://github.com/arXivTimes/arXivTimes/tree/master/datasets)

で、今回使ってみることにした`Sentiment140`は英語ツイートにネガ・ポジのラベルを付与してある約160万件データです。下記リンクから落とせます。

[For Academics - Sentiment140 - A Twitter Sentiment Analysis Tool](http://help.sentiment140.com/for-students/)

zipファイルがダウンロードされますが展開したファイルの`testdata.manual.2009.06.14.csv`は498件だけ入っていて、`training.1600000.processed.noemoticon.csv`の方に約16万件入ってます。小さい方のデータで動作確認してから、大きいデータで学習させるとデバッグが楽です。

## データを取得する

sentiment140のテストデータを開いて、学習用とテスト用に分割して返すスクリプト。最初はダウンロードして展開するところもコードで書いていたけど、文字コード関連のエラーの問題で諦めた。

sentiment140.py

```
from keras.utils.data_utils import get_file
import os, csv, subprocess
import zipfile
import numpy as np
import pandas as pd
from util import preprocess
from io import StringIO


def load_data():
    train_ratio = 0.7

    cols = ['sentiment','id','date','query_string','user','text']
    df = pd.read_csv('~/.keras/datasets/sentiment/training.1600000.processed.noemoticon.csv', header=None, names=cols)
    df = df.sample(frac=1, random_state=0)

    # コーパスを作成
    all_text = ' '.join(df['text'])
    corpus, word_to_id, id_to_word = preprocess(all_text)
    # print('corpus: ', corpus)

    # テキストを単語IDに変換する
    df['text'] = df['text'].apply(lambda t: text2word_id_list(t, word_to_id))

    # データを学習用とテスト用に分割する    
    p = int(train_ratio * len(df))
    train_data = df.iloc[:p, :]
    test_data = df.iloc[p:, :]

    return (train_data['text'], train_data['sentiment']), (test_data['text'], test_data['sentiment'])


def text2word_id_list(text, word_to_id):
    text = text.lower()
    text = text.replace('.', ' .')
    words = text.split(' ')
    return [word_to_id[w] for w in words]
```

`text2word_id_list`部分は書籍のゼロからディープラニングのコードの`preprocess`をコピペしてます。

[https://github.com/oreilly-japan/deep-learning-from-scratch](https://github.com/oreilly-japan/deep-learning-from-scratch)


## 学習させる

kerasのサンプルコードの`imdb_cnn.py`そのまんまでやってみました。語彙数がめっちゃ多いので`max_features`だけ`800000`に増やしてます。

```
from __future__ import print_function
from keras.preprocessing import sequence
from keras.models import Sequential
from keras.layers import Dense, Dropout, Activation
from keras.layers import Embedding
from keras.layers import Conv1D, GlobalMaxPooling1D
from datasets import sentiment140

# set parameters:
max_features = 800000
maxlen = 400
batch_size = 32
embedding_dims = 50
filters = 250
kernel_size = 3
hidden_dims = 250
epochs = 2

print('Loading data...')

(x_train, y_train), (x_test, y_test) = sentiment140.load_data()
print(len(x_train), 'train sequences')
print(len(x_test), 'test sequences')

# word_idのリストのサイズを400に揃える
print('Pad sequences (samples x time)')
x_train = sequence.pad_sequences(x_train, maxlen=maxlen)
x_test = sequence.pad_sequences(x_test, maxlen=maxlen)

print('x_train shape:', x_train.shape)
print('x_test shape:', x_test.shape)

print('Build model...')
model = Sequential()

# we start off with an efficient embedding layer which maps
# our vocab indices into embedding_dims dimensions
model.add(Embedding(max_features,
                    embedding_dims,
                    input_length=maxlen))
model.add(Dropout(0.2))

# we add a Convolution1D, which will learn filters
# word group filters of size filter_length:
model.add(Conv1D(filters,
                 kernel_size,
                 padding='valid',
                 activation='relu',
                 strides=1))
# we use max pooling:
model.add(GlobalMaxPooling1D())

# We add a vanilla hidden layer:
model.add(Dense(hidden_dims))
model.add(Dropout(0.2))
model.add(Activation('relu'))

# We project onto a single unit output layer, and squash it with a sigmoid:
model.add(Dense(1))
model.add(Activation('sigmoid'))

model.compile(loss='binary_crossentropy',
              optimizer='adam',
              metrics=['accuracy'])
model.fit(x_train, y_train,
          batch_size=batch_size,
          epochs=epochs,
          validation_data=(x_test, y_test))
```

実行するとUnicode関連のエラー。日本語じゃないのになあ。

```
UnicodeDecodeError: 'utf-8' codec can't decode bytes in position 80-81: invalid continuation byte
```

ファイルの文字コードを調べると、utf-8みたい？

```
$ file --mime ~/.keras/datasets/sentiment/training.1600000.processed.noemoticon.csv
/Users/harada/.keras/datasets/sentiment/training.1600000.processed.noemoticon.csv: text/plain; charset=utf-8
```

sentiment140データセットを使っているこの人のコードを読むと`latin-1`なのかな？

[sunny-side-up/data_utils.py at master · Lab41/sunny-side-up · GitHub](https://github.com/Lab41/sunny-side-up/blob/master/src/datasets/data_utils.py)

あれこれコードを変えてみてもエラーになるので、諦めて手動でBOMつけるコマンド叩いてみたら読み込めるようになった。

```
$ nkf --overwrite --oc=UTF-8-BOM ~/.keras/datasets/sentiment/training.1600000.processed.noemoticon.csv
```

気を取り直して実行。

```
$ python sentiment140.py

5024/1119962 [..............................] - ETA: 9:26:55 - loss: -13.5096 - acc: 0.0016
```

`loss`が増え続けて行きました。なんかダメっぽい。

文字コード関連のところでハマりまくってツラくなってきたので一旦諦めるわ。
