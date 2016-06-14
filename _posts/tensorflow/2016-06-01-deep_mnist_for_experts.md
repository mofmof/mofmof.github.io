---
layout: blog
title: 熟練者のためのディープMNIST
category: tensorflow
tags: tensorflow,機械学習,machine learning,tensorflow tutorial,翻訳,Deep MNIST,チュートリアル
summary: もしあなたが既に他のディープラーニングのパッケージや、MNISTに馴染みがあるのなら、このチュートリアルはTensorFlowの簡潔な手引になるでしょう。
author: aharada
---

TensorFlowは大規模な数値計算を行うための強力なライブラリです。TensorFlowはディープニューラルネットワークネットワークの実装・訓練を行うのに優れています。このチュートリアルでは、ディープ畳み込みMNIST分類の基礎的な構成要素を学んでいきます。

このイントロダクションでは、ニューラルネットワークとMNISTデータセットについて良く知っているものと仮定しています。もしそれらのバックグラウンドを持っていないのなら、[初心者向けのイントロダクション](https://www.tensorflow.org/versions/master/tutorials/mnist/beginners/index.html)を確認してください。始める前に必ず[TensorFlowをインストール](https://www.tensorflow.org/versions/master/get_started/os_setup.html)してください。


## セットアップ

モデルを作る前に、まずMNISTデータセットをロードし、TensorFlowのセッションをスタートします。

### MNISTデータのロード

TensorFlowにはMNISTデータセットを自動的にダウンロードしてインポートしてくれる[スクリプト](https://github.com/tensorflow/tensorflow/blob/r0.9/tensorflow/examples/tutorials/mnist/input_data.py)が含まれています。このスクリプトは「MNIST_data」ディレクトリを作成し、データファイルを保存します。

```
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
```

この`mnist`はNumPyの配列のような、訓練・検証・テストセットを持つ軽量のクラスです。また`mnist`はデータのミニバッチによって反復する関数を提供しており、以下で使うことになります。

### TensorFlowインタラクティブセッションのスタート

Tensorflowでの計算は、効率の良いC++バックエンドに依存しています。このバックエンドへのコネクションのことをセッションこと呼んでいます。TensorFlowプログラムの一般的な使い方は、最初にグラフを作成し、セッションでそれを起動します。

代わりにTensorFlowではよりコードを構造的に実装することを柔軟にする便利な`InteractiveSession`クラスを使います。これを使うことで、グラフを実行するのと同時に、[グラフを構築](https://www.tensorflow.org/versions/r0.9/get_started/basic_usage.html#the-computation-graph)する操作をはさみ込むことが可能になります。

これはIPythonのように双方向にやりとりする状況で作業する際に時に特に便利です。もし`InteractiveSession`を使わない場合、セッションを開始し、[グラフを起動](https://www.tensorflow.org/versions/r0.9/get_started/basic_usage.html#launching-the-graph-in-a-session)する前に、全てのグラフを構築しなければならなりません。

```
import tensorflow as tf
sess = tf.InteractiveSession()
```

<!-- ここまで文章精査 -->

#### グラフの処理

Pythonで効率的な数値計算をするためには、一般的にはNumPyのように、コストが高い行列の掛け算を行えるライブラリなど、別言語で実装された非常に処理効率の良いコードを使います。残念なことに、いまだ全ての操作において、処理をPythonに戻す際に多くのオーバーヘッドが発生してしまいます。このオーバーヘッドは、GPU上で処理を走らせたい時や分散処理をしたい場合に特に悪く、データ転送するために高い処理コストがかかってしまうことが起こり得る。

また、TensorFlowはPythonの外側でこの重い処理を行いますが、ステップを踏んで、このオーバーヘッドを避けていきます。 Pythonから独立してコストの高い処理を走らせる代わりに、TensorFlowでは 完全にPythonの外側で実行される双方向操作のグラフ描写を許可します。このアプローチはTheanoやTorchを使うとの同様です。

Pythonコードの役割は、外側でグラフ処理を構築するために、グラフ処理するための部品の実行を命令することです。それでは、グラフ処理の基本的になやり方の詳細を見ていきましょう。

## ソフトマックス回帰モデルの構築

このセクションでは、単一の線形レイヤーのソフトマックス回帰モデルの構築について扱います。次のセクションでは、これを拡張した、マルチレイヤーの畳み込みネットワークを使ったSoftmax回帰を扱います。

### プレースホルダ

入力画像と出力するクラスのためのノードの作成することにより、グラフ処理の構築を始めます。

```
x = tf.placeholder(tf.float32, shape=[None, 784])
y_ = tf.placeholder(tf.float32, shape=[None, 10])
```

この時点では`x`と`y_`には特定の値は入っていません。むしろ、これらは`プレースホルダ`であり、TensolFlowが処理を実行するタイミングで入力されます。

入力画像`x`は二次元の浮動小数点数を持つテンソルから成り立つ。これには`[None, 784]`という抽象的な型を代入します。`784`というのはMNIST画像の数で、`None`は一括処理するサイズにともなって、いくつかの値が入ることを示します。ターゲットの出力クラスの`y_`は2次元のテンソルから成り立ち、MNIST画像がどの数字であるかを示す、one-hot(1が1つで残りが全て0である)な10次元のベクトルです。

`shape`は任意の引数ですが、テンソルのシェイプに一致していないことが原因のバグを自動的にキャッチされます。

### 変数

まずは重みである`W`とバイアスである`b`を定義します。追加入力のように扱うことが想像できますが、TensorFlowでは`Variable`というよりよい方法があります。`Variable`は TensorFlowのグラフ処理で扱われる値です。これは処理において利用・修正することが出来ます。機械学習アプリケーションでは、たいてい1つはモデルの変数パラメータを持っているものです。

```
W = tf.Variable(tf.zeros([784,10]))
b = tf.Variable(tf.zeros([10]))
```

`tf.Variable`を呼び出して各パラメータを初期化を渡します。このケースでは、`W`と`b`ともにゼロで埋めた値で初期化しています。`W`は784x10次元の行列（つまり784個の入力と10の出力がある）で`b`は10次元のベクトル（10の分類）です。

`Variable`sはセッションの中で使うことができ、必ず初期化しなければなりません。 このステップでは既に指定された初期値（この場合はテンソルはゼロ埋めされている）をとり、それぞれの変数に割り当てます。これはそれぞれの変数について一度に行うことが出来ます。

```
sess.run(tf.initialize_all_variables())
```

### 予測とコストファンクション

回帰モデルを実装していきます。これはなんと一行で実装出来ます！ベクトル化された入力画像と重みの行列の`w`を掛け、バイアス`b`を足して、各クラスごとのsoftmax確率を計算します。

```
y = tf.nn.softmax(tf.matmul(x,W) + b)
```

このコストファンクションでトレーニングすることで簡単に最小化することが出来ます。コストファンクションはターゲットとモデルの予測との間の交差エントロピーになります。

```
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))
```

`tf.reduce_sum`は全てのクラスにまたがって合計し、`tf.reduce_mean`はそれらの平均をとります。

## モデルを訓練する

これまでに訓練するコストファンクションを定義しました。これはTensorFlowを使った単純な訓練の仕方の例です。TensorFlowは全体のグラフ処理を知っているため、勾配のコストを自動的に識別することが出来る。TensorFlowは色々の最適化されたアルゴリズムを標準で組み込んでいます。この例は、勾配の大きい最急降下法で、降りるステップの長さが0.5の交差エントロピーです。

```
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```

TensorFlowでは実際にグラフ処理のために一行を追加します。これらの操作には1回の勾配計算が含まれており、パラメータを処理し更新します。

戻り値の`train_step`は最急降下法によって更新されるパラメータです。モデルは、何度も`train_step`を繰り返し実行することにより完成されます。

```
for i in range(1000):
  batch = mnist.train.next_batch(50)
  train_step.run(feed_dict={x: batch[0], y_: batch[1]})
```

トレーニングの反復一回で、50件のトレーニングセットを読み込みます。`train_step`の操作では`feed_dict`を使うことでプレースホルダである`x`と`y_`を訓練セットの内容で置き換えます。`feed_dict`を使うことでグラフ処理の中のテンソルを置き換えることが出来ます。これはプレースホルダだけに限定されません。

### モデルの評価

モデルはうまく振舞っていますか？

最初に、正しいラベルを予測されたことを理解します。 `tf.argmax`は非常にに便利な関数で精度の高いエントリーのインデックスを与えてくれます。 例えば、`tf.argmax(y,1)`はモデルが最も可能性が高いと考えているラベルで、`tf.argmax(y_,1)`は`true`のラベルです。`tf.equal`は予測が正しい方どうかをチェックするために使うことが出来ます。

```
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
```

この結果はbooleanのlistを返します。それぞれが正しいことを決定し、浮動小数点数を指定して実行すると、平均値が取得される。例えば、`[True, False, True, True]`は`[1,0,1,1]`になり`0.75`となる。

```
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

最終的に、テストデータにおける正確さは92%となります。

```
print(accuracy.eval(feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
```

## 複数レイヤー畳み込みネットワークの構築

MNISTで91%の正確さは、はずかしいくらいに悪い精度です。このセクションでは、非常にシンプルなモデルから洗練されたモデルへと改善させます。小さな畳み込みニューラルネットワークで99.2%の正確さを実現させます。 -- 最先端の技術ではありませんが、満足できます。

### 重みの初期化

このモデルを作るためには、たくさんの重みとバイアスを作らなければなりません。一つは、重みはノイズを含んだ非対称の小さい値であり、かつ勾配が0になるのを避ける値である必要があります。ReLUニューロンを使い始めて以来、小さい正の値でバイアスを初期化することもまた、"死んだニューロン"を避けるための良いプラクティスです。その代わりに繰り返しモデルを構築する手軽な2つの関数を実装してみましょう。

```
def weight_variable(shape):
  initial = tf.truncated_normal(shape, stddev=0.1)
  return tf.Variable(initial)

def bias_variable(shape):
  initial = tf.constant(0.1, shape=shape)
  return tf.Variable(initial)
```

### 畳み込みとプーリング

TensorFloはたくさんの柔軟な畳み込みとプーリングも提供しています。どうやって境界を扱うのか？ひとまたぎのサイズは何でしょうか？この例では、常に平凡なバージョンを選択します。畳み込みは、1と0で埋められたひとまたぎを使います。このアウトプットはインプットと同じサイズです。プーリングは2x2ブロックの単純なold max poolingです。綺麗なコードを保つために、そのような抽象的な処理も関数にして実装しましょう。

```
def conv2d(x, W):
  return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
  return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                        strides=[1, 2, 2, 1], padding='SAME')
```

### 最初の畳み込みレイヤー

最初のレイヤーを実装していきます。これはmax poolingの畳み込みから構成されます。この畳み込みは各5x5のパッチの32フィーチャーで処理されます。この重みテンソルは`[5, 5, 1, 32]`のシェイプを持ちます。最初の2つの次元はパッチサイズで、次は入力チャネルの数で、最後はアウトプットチャネルの数です。また、各出力チャネルの要素にはバイアスのベクトルを持っています。

```
W_conv1 = weight_variable([5, 5, 1, 32])
b_conv1 = bias_variable([32])
```

レイヤーに適用するには、まず最初に`x`を4dのテンソルにreshapeします。2、3次元は画像の横幅・縦幅に相当し、最後の次元はカラーチャンネルの数に相当します。

```
x_image = tf.reshape(x, [-1,28,28,1])
```

`x_image`と重みテンソルを畳み込み、バイアスを加算したものに、ReLU関数を適用し、最後にmax_poolを実行します。

```
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
h_pool1 = max_pool_2x2(h_conv1)
```

### 2つ目の畳み込みレイヤー

ディープネットワークを構築するためには、このタイプのレイヤーを積み重ねます。2つのレイヤーは5x5のパッチの64個のフィーチャーを持つことになります。

```
W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])

h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
h_pool2 = max_pool_2x2(h_conv2)
```

### 密集して接続されたレイヤー

画像サイズを7x7に減少させ、全体の画像から処理することを許可された1024のニューロンを持つ、完全に接続されたレイヤーを追加します。プーリングレイヤーから取り出したテンソルを作り変えて、一括ベクトルの中へ入れ、重みの行列を掛け、バイアスを加算し、ReLUを適用します。

```
W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])

h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
```

#### ドロップアウト

オーバーフィッティングを避けるために、読み出しレイヤーの前にドロップアウトを適用します。 ニューロンの出力はドロップアウトの間、保持される見込みの`placeholder`を作成します。トレーニング中にドロップアウトをオンにし、テスト中にそれをオフにすることが出来ます。TensoFlowの`tf.nn.dropout`は追加マスキングの中の出力ニューロンのスケーリングを自動的にハンドリングし、追加のスケーリングなしでドロップアウトを動かせます(*1)。

```
keep_prob = tf.placeholder(tf.float32)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
```

### 読み出しレイヤー

最後に、上記のソフトマックス回帰のような、ソフトマックスレイヤーを追加します。

```
W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])

y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)
```

### モデルのトレーニングと評価

このモデルはどれだけの良く振る舞ってくれるでしょうか？トレーニングと評価をするためには、上記のような1レイヤーのソフトマックスネットワークとほとんど同じコードを使います。The differences are that: 違うところは、急な最急降下法のオプティマイザから、さらに洗練されたADAMオプティマイザに置き換えているところです。ドロップアウト率をコントロールする`feed_dict`の中の`keep_blob`という追加パラメータを含みます。更に、100回ごとの繰り返し　トレーニングプロセスの繰り返し100回ごとにログ出力するコードも追加します。

```
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y_conv), reduction_indices=[1]))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
correct_prediction = tf.equal(tf.argmax(y_conv,1), tf.argmax(y_,1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
sess.run(tf.initialize_all_variables())
for i in range(20000):
  batch = mnist.train.next_batch(50)
  if i%100 == 0:
    train_accuracy = accuracy.eval(feed_dict={
        x:batch[0], y_: batch[1], keep_prob: 1.0})
    print("step %d, training accuracy %g"%(i, train_accuracy))
  train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})

print("test accuracy %g"%accuracy.eval(feed_dict={
    x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))
```

最後のテストでは、このコードの実行結果は正確さ99.2%となります。

これでTensorFlowを使って洗練されたディープラーニングのモデルを素早く簡単に構築・学習・評価する方法を学びました。

*1: この小さな畳み込みニューラルネットワークでは、実際にはドロップアウトがあってもなくても近い性能になります。ドロップアウトはオーバーフィッティングを避けるためにたびたび有効ですが、規模の大きなニューラルネットワークを学習させる際にはより便利です。
