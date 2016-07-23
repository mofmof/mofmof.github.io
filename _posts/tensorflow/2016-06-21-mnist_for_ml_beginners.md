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

MNISTのラベルに対応するのは、画像に割り振られた数値で0〜9の値です。このチュートリアルの目的は、ラベルを表す"one-hotなベクトル"を得ることです。one-hotなベクトルはほとんどの次元が0で、内一つだけが1であるベクトルです。この場合、数値*n*を表すベクトルは、*n*番目のみが1のベクトルです。例えば3の場合は`[0,0,0,1,0,0,0,0,0,0]`というベクトルになります。なので、`mnist.train.labels`は`[55000, 10]`の浮動小数点数の配列になります。

<div style="width:40%; margin:auto; margin-bottom:10px; margin-top:20px;">
  <img style="width:100%" src="/images/tensorflow/2016-06-21-mnist_for_ml_beginners/mnist-train-ys.png">
</div>

これで実際にモデルを作る準備が出来ました！

## ソフトマックス回帰

MNISTは0〜9の数値の画像であることはわかっています。その画像を見てどの数値に対応する見込みかを判断出来るようにしたいです。例えば、9の画像を処理すると、80%の確率で9であると判断しますが、5%の確率で8であると判断し(数字の上半分が円状になっているため)、残りの数値は正しくないので、少しの確率でそれ以外の数値と判断します。

これは古典的なケースでソフトマックス回帰は自然でシンプルなモデルです。もし確率を別の異なるオブジェクトに割り当てたいなら、ソフトマックスがそれになりますが、最後のステップでソフトマックス層について扱うので、後ほど洗練されたモデルの訓練について学びます。

ソフトマックス回帰には2つのステップがあります。最初に確実に分類された入力のエビデンスを追加し、それからエビデンスを確率に変換します。

特定のクラスに振り分けられた画像のエビデンスを計算するために、ピクセルの濃さの重みを付けて合計します。重みは、エビデンスがそのクラスに入っているのに対してピクセルが濃い場合は負の値になり、そのエビデンスが有利なら正の値になります。

次の図は、それらのクラスについて学習済みのモデルの重みを表しています。赤い部分は負の重みで、青い部分は正の重みです。

<div style="width:40%; margin:auto; margin-bottom:10px; margin-top:20px;">
  <img style="width:100%" src="/images/tensorflow/2016-06-21-mnist_for_ml_beginners/softmax-weights.png">
</div>

その他のバイアスと呼ばれるいくつかのエビデンスを追加します。つまりは、いくつかのものは入力に依存する可能性が高いということです。結果は入力 $$x$$ を与えて分類 $$i$$ というエビデンスになります。

$$\text{evidence}_i = \sum_j W_{i,~ j} x_j + b_i$$

$$W_i$$ は重みで、 $$b_i$$ はクラス $$i$$ と入力画像 $$x$$ のピクセルの合計のインデックスである $$j$$ に対するバイアスです。「ソフトマックス」関数を使って、エビデンスの計算書を予測された確率 $$y$$ に変換します。

$$y = \text{softmax}(\text{evidence})$$

ソフトマックスは「アクティベーション」や「リンク」関数のような役割を担い、線形関数の出力をフォームに入れます。この場合、確率の割当は10ケースになります。変換の、各クラスに対応する確率のエビデンスを計算すると、考えることが出来ます。これはこのように定義されます。

$$\text{softmax}(x) = \text{normalize}(\exp(x))$$

この式を展開するとこのようになります。

$$\text{softmax}(x)_i = \frac{\exp(x_i)}{\sum_j \exp(x_j)}$$

ですが、しばしば最初にソフトマックスを考えることの助けになります。べき乗は入力し正規化します。べき乗する方法は、もう一つのエビデンスの単位は、仮説の乗法に与えられた重みが増加します。一方で、小さい単位のエビデンスの平均は、仮説が示す初期の重みの一部です。仮説はゼロもしくはマイナスの重みになることはありません。ソフトマックスはそれらの重みを正規化すると、それに加算し、有効な可能性の割り当てを形成します。（さらにソフトマックス関数について直感を得るには、Michael Nielsen氏の本の[セクション](http://neuralnetworksanddeeplearning.com/chap3.html#softmax)がおすすめです。双方向の視覚化を完全にします。）

You can picture our softmax regression as looking something like the following, although with a lot more xs. For each output, we compute a weighted sum of the xs, add a bias, and then apply softmax.
