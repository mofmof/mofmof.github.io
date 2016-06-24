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
  <img style="width:100%" src="https://www.tensorflow.org/versions/r0.9/images/MNIST.png">
</div>

各画像にはラベルも含まれており、その画像がどの数値に対応するかを示しています。例えば上の画像には5, 0, 4, 1というラベルが対応します。

このチュートリアルでは、画像を見てることでモデルを訓練し、どの数字であるかを予測させます。今回のゴールは最高水準のパフォーマンスの精巧なモデルを訓練することではなく(それはまたあとでコードをお見せします)、むしろTensorFlowにまず触れてみることです。まずは非常にシンプルなモデルであるソフトマックス回帰から始めて見ましょう。

このチュートリアルでの実際のコードは非常に短く、たった3行で全ての興味深いことが起こります。しかし、これはどのようにTensorFlowが動作するのか　また機械学習の概要の背景を理解するために非常に重要なものです。このため、コードを通して丁寧に進めて行きます。
