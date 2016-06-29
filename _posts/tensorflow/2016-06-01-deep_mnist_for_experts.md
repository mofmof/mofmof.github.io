---
layout: blog
title: 熟練者のためのディープMNIST
category: tensorflow
tags: [tensorflow,機械学習,machine learning,tensorflow tutorial,翻訳,Deep MNIST,チュートリアル]
summary: もしあなたが既に他のディープラーニングのパッケージや、MNISTに馴染みがあるのなら、このチュートリアルはTensorFlowの簡潔な手引になるでしょう。
author: aharada
---

*※このエントリはTensorFlowの公式ドキュメント「[Deep MNIST for Experts](https://www.tensorflow.org/versions/r0.9/tutorials/mnist/pros/index.html)」を翻訳したものです*

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


#### グラフの計算

Pythonで効率的な数値計算をするためには、一般的にはコストが高い行列の掛け算を行える、別言語で実装された非常に処理効率の良いコードを使うNumPyのようなライブラリを使用します。残念なことに、いまだ全ての操作において、処理をPythonに戻す際に多くのオーバーヘッドが発生してしまいます。GPU上で処理を走らせたい時や分散処理をしたい場合に特に悪く、データ転送するために高い処理コストがかかってしまうことが起こり得ます。

TensorFlowはPythonの外側でこの重い処理を行いますが、このオーバーヘッドを避ける方法を取ります。Pythonから独立してコストの高い処理を走らせる代わりに、TensorFlowでは 完全にPythonの外側で実行される双方向操作のグラフとして記述出来ます。このアプローチはTheanoやTorchを使うとの同様です。

Pythonコードの役割は、外側でグラフ計算を構築するために、実行すべきグラフ計算の部品を記述することです。[グラフ計算](https://www.tensorflow.org/versions/r0.9/get_started/basic_usage.html#the-computation-graph)の[基本的な使用方法](https://www.tensorflow.org/versions/r0.9/get_started/basic_usage.html)のセクションの詳細を参照してください。

## ソフトマックス回帰モデルの構築

このセクションでは、単一の線形レイヤーのソフトマックス回帰モデルの構築について扱います。次のセクションでは、これを拡張した、多層畳み込みネットワークでのソフトマックス回帰について扱います。

### プレースホルダ

入力画像と出力するクラスのためのノードの作成することにより、グラフ計算の構築を始めます。

```
x = tf.placeholder(tf.float32, shape=[None, 784])
y_ = tf.placeholder(tf.float32, shape=[None, 10])
```

この時点では`x`と`y_`には特定の値は入っていません。これらは`プレースホルダ`であり、TensolFlowが処理を実行するタイミングで入力される値です。


入力画像`x`は浮動小数点数を持つ二次元のテンソルからなります。これには`[None, 784]`という抽象的な型を割り当てます。`784`というのはMNIST画像の次元で、`None`はバッチサイズと対応し、任意の値が入ることを示します。ターゲットの出力クラスの`y_`は二次元のテンソルから成り立ち、MNIST画像がどの数字であるかを示す、one-hot(1が1つで残りが全て0である)な10次元のベクトルです。

`shape`は任意の引数ですが、これを指定するとテンソルのシェイプに一致していないことが原因のバグを自動的にキャッチすることが出来ます。

### 変数

まずは重みである`W`とバイアスである`b`を定義します。追加入力のように扱うことが想像できますが、TensorFlowでは`Variable`というよりよい方法があります。`Variable`は TensorFlowのグラフ計算内にある値です。これは計算により利用・修正することが出来ます。機械学習アプリケーションでは、一般的にはモデルのパラメータを`Variable`が持っています。

```
W = tf.Variable(tf.zeros([784,10]))
b = tf.Variable(tf.zeros([10]))
```

`tf.Variable`を呼び出して各パラメータの初期値を渡します。このケースでは、`W`と`b`ともにゼロで埋めたテンソルで初期化しています。`W`は784x10次元の行列（つまり784のフィーチャーと10の出力があるので）で`b`は10次元のベクトル（10の分類があるので）です。

`Variable`sはセッションの中で使うことができ、必ず初期化しなければなりません。 このステップでは既に指定された初期値（この場合はテンソルはゼロ埋められている）をとり、それぞれの変数に割り当てます。これはそれぞれの変数について一度に行うことが出来ます。

```
sess.run(tf.initialize_all_variables())
```

### 予測とコスト関数

これで回帰モデルを実装出来るようになりました。これはなんと一行で実装出来ます！ベクトル化された入力画像の`x`と重みの行列の`W`を掛け、バイアス`b`を足して、各クラスごとのsoftmax確率を計算します。

```
y = tf.nn.softmax(tf.matmul(x,W) + b)
```

訓練中にに最小化されたコスト関数は、簡単に指定することが出来ます。コスト関数はターゲットとモデルの予測との間の交差エントロピーになります。

```
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))
```

`tf.reduce_sum`は全てのクラスにまたがって合計し、`tf.reduce_mean`はそれらの平均をとることに注意してください。

## モデルの訓練
<!-- ここまで文章精査 -->

これまでに訓練するコスト関数を定義しましたので、TensorFlowを使った訓練の仕方は簡単です。TensorFlowは全体のグラフ計算を知っているため、各変数に関する勾配のコストを見つけるために自動的に識別することが出来ます。TensorFlowは様々な[最適化されたアルゴリズムを標準で組み込んでいます](https://www.tensorflow.org/versions/r0.9/api_docs/python/train.html#optimizers)。この例では、交差エントロピーを下降させるために0.5のステップの急な勾配の最急降下法を使用しています。

```
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```

TensorFlowが実際にこの一行をを追加することは、グラフ計算に新しい操作を追加することです。これらの操作には勾配降下を計算し、パラメータの更新ステップを計算し、パラメータに更新ステップを適用する操作が含まれています。

実行時の戻り値`train_step`は勾配降下の更新をパラメータに適用します。モデルは、何度も`train_step`を繰り返し実行することにより完成されます。

```
for i in range(1000):
  batch = mnist.train.next_batch(50)
  train_step.run(feed_dict={x: batch[0], y_: batch[1]})
```

各訓練の反復で、50件のトレーニングセットを読み込みます。`feed_dict`を使い、プレースホルダのテンソルである`x`と`y_`を訓練セットの内容で置き換え、`train_step`の操作を実行します。`feed_dict`を使うことでグラフ計算の中のテンソルを置き換えることが出来ることに注意してください(これはプレースホルダだけに限定されません)。

### モデルの評価

モデルはどれくらいうまく振舞っていますか？

まずは、どこで正しいラベルが予測されたか見つけましょう。`tf.argmax`は非常にに便利な関数で精度の高いエントリーのインデックスを与えてくれます。 例えば、`tf.argmax(y,1)`はモデルが最も可能性が高いと考えているラベルで、`tf.argmax(y_,1)`は正しいラベルです。`tf.equal`は予測が正しいかどうかをチェックするために使うことが出来ます。

```
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
```

結果は真偽値のリストを返します。どの部分が正しいかを決定するために、浮動小数点数にキャストして平均値を取ります。例えば、`[True, False, True, True]`は`[1,0,1,1]`になり`0.75`となります。

```
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

最後に、テストデータにおける正確さを評価すると92%になります。

```
print(accuracy.eval(feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
```

## 多層畳み込みネットワークの構築

MNISTで91%の正確さは、はずかしいくらいに悪い精度です。このセクションでは、非常にシンプルなモデルから洗練されたモデルへと改善させます。小さな畳み込みニューラルネットワークで99.2%の正確さを実現させます。これは最高水準ではありませんが、立派な精度です。

### 重みの初期化

このモデルを作るためには、たくさんの重みとバイアスを作らなければなりません。一般的には対称性を壊し0勾配を避けるために、少しのノイズを含んだを値で重みを初期化する必要があります。

ReLUニューロンを使い始めて以来、小さい正の値でバイアスを初期化することもまた「死んだニューロン」を避けるための良いプラクティスです。繰り返しモデルを構築する代わりに、使いやすい2つの関数を実装してみましょう。

```
def weight_variable(shape):
  initial = tf.truncated_normal(shape, stddev=0.1)
  return tf.Variable(initial)

def bias_variable(shape):
  initial = tf.constant(0.1, shape=shape)
  return tf.Variable(initial)
```

### 畳み込みとプーリング

TensorFlowはたくさんの柔軟な畳み込みとプーリング処理も提供しています。どうやって境界を扱うのでしょうか？歩幅のサイズは？この例では、常に普通なバージョンを選択します。畳み込みは、1の歩幅を使用し、出力が入力とおなじサイズになるようにゼロパディングします。プーリングは2x2ブロックの単純なold max poolingです。綺麗なコードを保つために、そのような抽象的な処理も関数にして実装しましょう。

```
def conv2d(x, W):
  return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
  return tf.nn.max_pool(x, ksize=[1, 2, 2, 1],
                        strides=[1, 2, 2, 1], padding='SAME')
```

### 最初の畳み込みレイヤー

今これで最初のレイヤーを実装出来るようになりました。これは畳み込みとmax poolingから構成されます。この畳み込みは各5x5のパッチの32の特徴で計算されます。この重みテンソルは`[5, 5, 1, 32]`のシェイプを持ちます。最初の2つの次元はパッチサイズで、次は入力チャネルの数で、最後は出力チャネルの数です。また、各出力チャネルの要素にはバイアスのベクトルを持つことになります。

```
W_conv1 = weight_variable([5, 5, 1, 32])
b_conv1 = bias_variable([32])
```

この層に適用するためには、まず最初に`x`を4次元のテンソルに変形します。2、3次元は画像の横幅・縦幅に対応し、最後の次元はカラーチャネルの数に対応します。

```
x_image = tf.reshape(x, [-1,28,28,1])
```

`x_image`と重みテンソルを畳み込み、バイアスを加算したものに、ReLU関数を適用し、最後にmax_poolを実行します。

```
h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
h_pool1 = max_pool_2x2(h_conv1)
```

### 2つ目の畳み込み層

ディープネットワークを構築するためには、このタイプの層を積み重ねます。2つ目の層は5x5のパッチの64の特徴を持つことになります。

```
W_conv2 = weight_variable([5, 5, 32, 64])
b_conv2 = bias_variable([64])

h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
h_pool2 = max_pool_2x2(h_conv2)
```

### 密に接続された層

画像サイズを7x7に減少させ、画像全体から処理することを出来るようにするために1024のニューロンを持つ、完全に接続された層を追加します。プーリング層から取り出したテンソルをベクトルのバッチに形を変えて、重みの行列を掛け、バイアスを加算し、ReLUを適用します。

```
W_fc1 = weight_variable([7 * 7 * 64, 1024])
b_fc1 = bias_variable([1024])

h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)
```

#### ドロップアウト

オーバーフィッティングを避けるために、読み出し層の前にドロップアウトを適用します。 ニューロンの出力はドロップアウトの間、保持される見込みのプレースホルダを作成します。訓練中にドロップアウトをオンにし、テスト中にそれをオフにすることが出来るようになります。TensoFlowの`tf.nn.dropout`はそれらのマスキングに加えて、ニューロン出力のスケーリングを自動的に処理し、ドロップアウトは追加のスケーリングなしに動作します(*1)。

```
keep_prob = tf.placeholder(tf.float32)
h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)
```

### 読み出し層

最後に、上記の単層ソフトマックス回帰のような、ソフトマックス層を追加します。

```
W_fc2 = weight_variable([1024, 10])
b_fc2 = bias_variable([10])

y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)
```

### モデルの訓練と評価

このモデルはどれだけの良く振る舞っているでしょうか？モデルの訓練と評価をするためには、上記のような単層ソフトマックスネットワークとほとんど同じコードを使います。異なる点は次の通りです。急な最急降下法のオプティマイザから、さらに洗練されたADAMオプティマイザに置き換え、ドロップアウト率をコントロールする`keep_blob`という追加パラメータを`feed_dict`に含め、更に、訓練プロセスの繰り返し100回ごとにログ出力するコードも追加します。

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

最終的なテストでは、このコードの実行結果は正確さ99.2%となります。

これでTensorFlowを使って洗練されたディープラーニングのモデルを素早く簡単に構築・学習・評価する方法を学びました。

*1: この小さな畳み込みニューラルネットワークでは、実際にはドロップアウトがあってもなくても近い性能になります。ドロップアウトはオーバーフィッティングを避けるためにたびたび有効ですが、規模の大きなニューラルネットワークを学習させる際にはより便利です。
