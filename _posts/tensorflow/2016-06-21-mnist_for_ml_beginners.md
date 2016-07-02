---
layout: blog
title: 機械学習初級者向けMNIST
category: tensorflow
tags: [tensorflow,機械学習,machine learning,tensorflow tutorial,翻訳,MNIST,チュートリアル]
summary: もし機械学習が初めてなら、まずはここからはじめることをおすすめします。典型的な問題である、手書き数字の分類(MNIST)について学び、マルチクラスの分類についても紹介します。
author: aharada
---

*※このエントリはTensorFlowの公式ドキュメント「[MNIST For ML Beginners](https://www.tensorflow.org/versions/r0.9/tutorials/mnist/beginners/index.html)」を翻訳したものです*

---

このチュートリアルは機械学習とTensorFlowの両方に新しく触れる読者を対象にしています。もしすでに、MNISTやソフトマックス回帰が何なのか知っているなら、このチュートリアルは[速いペース](https://www.tensorflow.org/versions/r0.9/tutorials/mnist/pros/index.html)で進めてしまっても構いません。チュートリアルを始める前に[TensorFlowをインストール](https://www.tensorflow.org/versions/r0.9/get_started/os_setup.html)してください。

どのようにプログラムするかを学ぶときには伝統的なやり方として、一番最初にやることは「Hello World.」を出力することです。プログラミングにおけるHello Worldようなものとして、機械学習にはMNISTというものがあります。

MNISTはシンプルな画像認識データセットです。これは以下のような手書きの数字画像から構成されます。

<div style="width:40%; margin:auto; margin-bottom:10px; margin-top:20px;">
  <img style="width:100%" src="/images/tensorflow/2016-06-21-mnist_for_ml_beginners/MNIST.png">
</div>

各画像にはラベルも含まれており、その画像がどの数値に対応するかを示しています。例えば上の画像には5, 0, 4, 1というラベルが対応します。

このチュートリアルでは、画像を見てることでモデルを訓練し、どの数字であるかを予測させます。今回のゴールは最高水準のパフォーマンスの精巧なモデルを訓練することではなく(それはまたあとでコードをお見せします)、むしろTensorFlowにまず触れてみることです。まずは非常にシンプルなモデルであるソフトマックス回帰から始めて見ましょう。

このチュートリアルでの実際のコードは非常に短く、たった3行で全ての興味深いことが起こります。しかし、これはどのようにTensorFlowが動作するのか　また機械学習の概要の背景を理解するために非常に重要なものです。このため、コードを通して丁寧に進めて行きます。

## MNISTデータ

MNISTデータはYann [LeCun氏のWEBサイト](http://yann.lecun.com/exdb/mnist/でホスティングされています。便利に進めるために、TensorFlowにはデータを自動的にダウンロード・インストールするるためのpythonコードが含まれています。下記のように[コード](https://github.com/tensorflow/tensorflow/blob/r0.9/tensorflow/examples/tutorials/mnist/input_data.py)をダウンロードしてインポートするか、シンプルにコピーペーストすることが出来ます。

```
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
```

ダウンロードされたデータは3つの部分に分割されており、55,000件の訓練データ(`mnist.train`)と、10,000件のテストデータ('mnist.test')と、5,000件の検証データ('mnist.validation')で構成されています。この分割は非常に重要で、これは機械学習にとって学習していない区分けされたデータは必要不可欠で、実際に一般化を学習出来ていることを確認出来ます！

先に述べたように、全てのMNISTデータは2つの部分で出来ています。手書き数字画像と、それに対応するラベルです。この画像の方を"xs"、ラベルの方を"ys"とします。訓練セットとテストセットの両方(訓練する画像`mnist.train.images`とそのラベル`mnist.train.labels`)をxsとysに含みます。

各画像は28x28ピクセルの画像です。これは大きい次元数の数値配列として解釈出来ます。

<div style="width:50%; margin:auto; margin-bottom:10px; margin-top:20px;">
  <img style="width:100%" src="/images/tensorflow/2016-06-21-mnist_for_ml_beginners/MNIST-Matrix.png">
</div>

この28x28のベクトルが入った配列を平坦にして784の数値にすることが出来る。これは、画像が一定である限り、どのように配列を平坦にするかという問題ではありません。この観点から見ると、MNIST画像は[非常に豊富な構造](http://colah.github.io/posts/2014-10-Visualizing-MNIST/)を持つ784次元のポイントを束ねたベクトル空間です(注意：計算集約的な視覚化)。

データの平坦化は、画像の2次元構造の情報を破棄します。これは悪いことでしょうか？視覚化する最適な方法は、この構造を利用しすることです。これついてはチュートリアルの後のほうで扱いますが、ソフトマックス回帰ではないシンプル方法はここでも使います。

結果`mnist.train.images`はテンソル(N次元配列)と`[55000, 784]`のシェイプです。最初の次元は画像、次の次元は各画像のピクセルです。テンソル内の各エントリは、いずれかの画像のいずれかのピクセルの色の濃さを0〜1の値で表したものです。

<div style="width:40%; margin:auto; margin-bottom:10px; margin-top:20px;">
  <img style="width:100%" src="/images/tensorflow/2016-06-21-mnist_for_ml_beginners/mnist-train-xs.png">
</div>

MNISTのラベルに対応するのは、画像に割り振られた数値で0〜9の値です。 For the purposes of this tutorial, we're going to want our labels as "one-hot vectors". A one-hot vector is a vector which is 0 in most dimensions, and 1 in a single dimension. In this case, the nth digit will be represented as a vector which is 1 in the nth dimensions. For example, 3 would be [0,0,0,1,0,0,0,0,0,0]. Consequently, mnist.train.labels is a [55000, 10] array of floats.
