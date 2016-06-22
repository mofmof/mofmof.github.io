---
layout: blog
title: MNIST For ML Beginners
category: tensorflow
tags: [tensorflow,機械学習,machine learning,tensorflow tutorial,翻訳,MNIST,チュートリアル]
summary: If you're new to machine learning, we recommend starting here. You'll learn about a classic problem, handwritten digit classification (MNIST), and get a gentle introduction to multiclass classification.
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
