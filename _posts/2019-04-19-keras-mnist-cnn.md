---
layout: blog
title: KerasでMNIST分類問題をCNNで解く
category: blog
tags: [機械学習,machine learning,TensorFlow,Keras,MNIST]
summary: Kerasはあくまでも機械学習フレームワークのインタフェースを簡便なものにするラッパーなので、バックエンドが必要。基本的にはtensor flowを推奨してる模様。
author: aharada
image: /images/blog/2019-04-19-keras-mnist-cnn/MNIST.png
---

Kerasはあくまでも機械学習フレームワークのインタフェースを簡便なものにするラッパーなので、バックエンドが必要。基本的にはtensor flowを推奨してる模様。

[Home - Keras Documentation](https://keras.io/ja/#_2)

Kerasのリポジトリ。

[GitHub - keras-team/keras: Deep Learning for humans](https://github.com/keras-team/keras)

## インストール
ぼくはpyenv使ってます。今回はpython 3.6.4でやってみます。

```
$ pyenv local 3.6.4
$ python --version
Python 3.6.4
```

tensorflowとkerasをインストール。

```
$ pip install tensorflow
$ pip install keras
```

## MNISTを学習
Kerasのリポジトリ内にいろんなサンプルが用意されているので適当なのやってみます。

手書き数字認識で有名なMNISTのデータをCNNで解くサンプルがあったので、これでいいや。

![MNIST](/images/blog/2019-04-19-keras-mnist-cnn/MNIST.png)

[keras/mnist_cnn.py at master · keras-team/keras · GitHub](https://github.com/keras-team/keras/blob/master/examples/mnist_cnn.py)

おもむろに`mnist_cnn.py`を無思考でコピペして動かしてみる。

```
$ python mnist_cnn.py
```

lossが下がり、accが上がっていく様子がリアルタイムに見られるので楽しい。

```
22656/60000 [==========>...................] - ETA: 1:42 - loss: 0.4622 - acc: 0.8562
```

だがしかしデフォがepoch: 12になっているのでCPUで動かしているとめっちゃ待つ。いったんそれっぽく動けばいいのでepoch: 1にしてやりなおすとサクッと終わる。

```
60000/60000 [==============================] - 189s 3ms/step - loss: 0.2654 - acc: 0.9184 - val_loss: 0.0592 - val_acc: 0.9812
Test loss: 0.059171129403682424
Test accuracy: 0.9812
```

続いて予測させようと思ったのですが、試行錯誤しようとすると毎回学習処理をしてしまって待ちがつらいので、学習済みモデルをファイルにdumpします。

やり方はこの辺に載ってる。

[How can I save a Keras model?](https://keras.io/getting-started/faq/#how-can-i-save-a-keras-model)

`mnist_cnn.py`の末尾に下記を追記して再学習すると、同じディレクトリに`mnist_cnn_model.h5`が保存されていることが確認出来る。

```
model.save('mnist_cnn_model.h5') 
```

## 分類する
分類のコードはこんな感じ。

mnist_cnn_predict.py

```
from __future__ import print_function
import keras
from keras.datasets import mnist
from keras.models import load_model
from keras import backend as K
import numpy as np

# input image dimensions
img_rows, img_cols = 28, 28

model = load_model('mnist_cnn_model.h5')

# the data, split between train and test sets
(x_train, y_train), (x_test, y_test) = mnist.load_data()
if K.image_data_format() == 'channels_first':
    x_train = x_train.reshape(x_train.shape[0], 1, img_rows, img_cols)
    x_test = x_test.reshape(x_test.shape[0], 1, img_rows, img_cols)
    input_shape = (1, img_rows, img_cols)
else:
    x_train = x_train.reshape(x_train.shape[0], img_rows, img_cols, 1)
    x_test = x_test.reshape(x_test.shape[0], img_rows, img_cols, 1)
    input_shape = (img_rows, img_cols, 1)


print('x_train.shape: ', x_train.shape)
x = x_train[:10]  # 先頭10件で予測させる
y = model.predict(x)

print('y_train[:10]:  ', y_train[:10])

# one-hotベクトルで結果が返るので、数値に変換する
y_classes = [np.argmax(v, axis=None, out=None) for v in y]
print('y_classes: ', y_classes)
```

分類する。`y_train[:10]`と`y_classes`の値が一致していることを確認。意図した通りに動いてますね。

```
$ python mnist_cnn_predict.py
x_train.shape:  (60000, 28, 28, 1)
y_train[:10]:   [5 0 4 1 9 2 1 3 1 4]
y_classes:  [5, 0, 4, 1, 9, 2, 1, 3, 1, 4]
```

以前にMNIST問題をtensor flowでやってみたことあるけど、kerasは圧倒的に楽ちんですね。大変良い。