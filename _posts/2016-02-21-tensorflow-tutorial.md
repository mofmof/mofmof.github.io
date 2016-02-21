---
layout: blog
title: TensorFlowで手書きの数字を認識するチュートリアルを試した
category: blog
tags: [機械学習,machine learning,TenforFlow]  
summary: デブサミ2016でDataRobotのセッションをみて感激したのですが、そのときにTensorFlowのデモもちょっとだけやっていて、試したくなりました。
image: /images/blog/2016-02-21-tensorflow-tutorial/tensorflow.png
author: aharada
---

![](../images/blog/2016-02-21-tensorflow-tutorial/tensorflow.png)

デブサミ2016でDataRobotのセッションをみて感激したのですが、そのときにTensorFlowのデモもちょっとだけやっていて、試したくなりました。ぼくはRubyistなのでPythonをあまり書いたことがないのですがチャンレンジしてみます。

## TensorFlowをインストール
このあたりを参考にしました。
[https://www.tensorflow.org/versions/master/get_started/os_setup.html#pip_install]([https://www.tensorflow.org/versions/master/get_started/os_setup.html#pip_install)
[http://www.pxt.jp/ja/diary/article/299_tensorflow_getting_started/](http://www.pxt.jp/ja/diary/article/299_tensorflow_getting_started/)

pipが入っている前提で進めます。
ぼくの環境ではこれだけでインストール出来たみたい。

```
$ sudo easy_install --upgrade six
$ sudo pip install --upgrade https://storage.googleapis.com/tensorflow/mac/tensorflow-0.6.0-py2-none-any.whl
```

## チュートリアル
MNISTの手書き数字画像から数字を認識するチュートリアル。
[https://www.tensorflow.org/versions/master/tutorials/mnist/beginners/index.html](https://www.tensorflow.org/versions/master/tutorials/mnist/beginners/index.html)

TensorFlowのチュートリアルに関係するソースを落としてきます。

```
$ git clone --recurse-submodules https://github.com/tensorflow/tensorflow
```

とりあえずチュートリアルに書いてあるコードを全て単にコピペしてみるとこうなる。

mnist_tutorial.py

```
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)

import tensorflow as tf

x = tf.placeholder(tf.float32, [None, 784])

W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
y = tf.nn.softmax(tf.matmul(x, W) + b)

y_ = tf.placeholder(tf.float32, [None, 10])
cross_entropy = -tf.reduce_sum(y_*tf.log(y))

train_step = tf.train.GradientDescentOptimizer(0.01).minimize(cross_entropy)

init = tf.initialize_all_variables()

sess = tf.Session()
sess.run(init)

for i in range(1000):
  batch_xs, batch_ys = mnist.train.next_batch(100)
  sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})

correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
print(sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
```

tensorflowリポジトリに入っているtensorflow/examples/tutorials/mnist/inpyt_data.pyが必要なので、同一ディレクトリにコピペしておく。

実行してみるとチュートリアルに書いてあるとおり約91%の正答率であることが確認できた。

```
$ tensorflow-try  python mnist_tutorial.py
Extracting MNIST_data/train-images-idx3-ubyte.gz
Extracting MNIST_data/train-labels-idx1-ubyte.gz
Extracting MNIST_data/t10k-images-idx3-ubyte.gz
Extracting MNIST_data/t10k-labels-idx1-ubyte.gz
I tensorflow/core/common_runtime/local_device.cc:40] Local device intra op parallelism threads: 4
I tensorflow/core/common_runtime/direct_session.cc:58] Direct session inter op parallelism threads: 4
0.9137
```

しかし何をやっているのか全くわからないとアレなので、ちょっと一つ一つ確認してみる。

## 実装
チュートリアルの説明を読むとsoftmax regressionで予測するらしい。アルゴリズムも書いてある。logistic regressionでもあるのかな。

ちなみにMNISTとは1〜9の手書きの数字をデータセット化したもの。WEB上に公開されているデータで機械学習の入門で使われる定番らしい。

```
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
```

`tensorflow.examples.tutorials.mnist`パッケージの`input_data`をインポートして、`read_data_sets`関数を実行。input_data.pyをざっくり読むとどうやら、MNISTからデータをダウンロードしてきてDataSetクラスにして返してくれるのかな？TensorFlowで使える形式に変換しているのかも。

```
import tensorflow as tf
```

tensorflowをimport。


```
x = tf.placeholder(tf.float32, [None, 784])
```

学習セット用xのプレースホルダー(入れ物)を準備。手書き画像が28x28ピクセルなので、784ピクセル分のデータが入る。

```
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
```

Wは重み、bはバイアスの変数定義らしい。softmax regressionがイマイチ理解しきれていないので、ちょっと詳しくはわからない。784は上と同様28x28ピクセルで、10は結果値yが0-9の10種類存在するので10となる。

```
y = tf.nn.softmax(tf.matmul(x, W) + b)
```

softmaxのモデル式定義かな？

## トレーニングの実装
学習させる部分の実装を見ていきます。

```
y_ = tf.placeholder(tf.float32, [None, 10])
```

結果値のyのプレースホルダをつくる。

```
cross_entropy = -tf.reduce_sum(y_*tf.log(y))
```

コスト関数の定義。

```
train_step = tf.train.GradientDescentOptimizer(0.01).minimize(cross_entropy)
init = tf.initialize_all_variables()
sess = tf.Session()
sess.run(init)
```

最急降下法の定義して、TensorFlowのsessionを初期化する。

```
for i in range(1000):
  batch_xs, batch_ys = mnist.train.next_batch(100)
  sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
```

1000回繰り返し最急降下法の更新をさせる。

これで学習済みの状態になるっぽい。

## 評価させる

```
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

正答率を算出する処理を実装。

```
print(sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
```

MNISTのデータをインプットさせて予測を実行する。これにより約91%の正答率であることが出力される。

## まとめ
この程度だとTensorFlowを使わずとも簡単に出来るのであまりすごさがわからないチュートリアルだった。あとsoftmax regressionを知らないと何やってるかがイマイチわからない。

とはいえpythonは書いたことがあまりないけどTensorFlowの使い方自体はなんとなく見えた。引き続きやれることを増やしていきたい。
