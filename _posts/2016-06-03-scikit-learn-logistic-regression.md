---
layout: blog
title: scikit-learnを使ってIrisデータのロジスティック回帰分類を実装する
category: blog
tags: [python,scikit-learn,machine learning,機械学習,Iris,ロジスティック回帰,Logistic Regression]  
summary: scikit-learnでサクッとロジスティック回帰の実装をやってみようと思ったのですが、
author: aharada
---

scikit-learnでサクッとロジスティック回帰の実装をやってみようと思ったのですが、思いのほか`matplotlib`にハマってしまった。エラー対応の履歴がほとんど。

まずは公式ドキュメントのこれを見てやっていきます。

[http://scikit-learn.org/stable/auto_examples/linear_model/plot_iris_logistic.html](http://scikit-learn.org/stable/auto_examples/linear_model/plot_iris_logistic.html)

とりあえず`pip`でライブラリをインストールしてからはじめます。

```
$ pip install numpy
$ pip install matplotlib
$ pip install scipy
$ pip install scikit-learn
```

## matplotlibがimportに失敗する問題の対処

```
$ python main.py
Traceback (most recent call last):
  File "main.py", line 3, in <module>
    import matplotlib.pyplot as plt
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/pyplot.py", line 29, in <module>
    import matplotlib.colorbar
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/colorbar.py", line 32, in <module>
    import matplotlib.artist as martist
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/artist.py", line 14, in <module>
    from .transforms import (Bbox, IdentityTransform, TransformedBbox,
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/transforms.py", line 39, in <module>
    from matplotlib._path import (affine_transform, count_bboxes_overlapping_bbox,
ImportError: dlopen(/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/_path.so, 2): Symbol not found: _PyUnicodeUCS2_AsASCIIString
  Referenced from: /Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/_path.so
  Expected in: flat namespace
 in /Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/_path.so
```

matplotlibをimportしようとしたらエラーになりました。

なかなか対策を見つけられなかったのですが、こちらのエントリの通りpythonを再インストールすることで動作するようになりました。

[http://qiita.com/como1559/items/feac9bacbbf362817c66](http://qiita.com/como1559/items/feac9bacbbf362817c66)

```
$ pyenv uninstall 2.7.10
$ PYTHON_CONFIGURE_OPTS="--enable-unicode=ucs2" pyenv install 2.7.10
```

これで勝つる！と思いきやまたエラー。

```
$ python main.py
/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/font_manager.py:273: UserWarning: Matplotlib is building the font cache using fc-list. This may take a moment.
  warnings.warn('Matplotlib is building the font cache using fc-list. This may take a moment.')
Traceback (most recent call last):
  File "main.py", line 3, in <module>
    import matplotlib.pyplot as plt
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/pyplot.py", line 114, in <module>
    _backend_mod, new_figure_manager, draw_if_interactive, _show = pylab_setup()
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/backends/__init__.py", line 32, in pylab_setup
    globals(),locals(),[backend_name],0)
  File "/Users/haradaatsushi/.pyenv/versions/2.7.10/lib/python2.7/site-packages/matplotlib/backends/backend_macosx.py", line 24, in <module>
    from matplotlib.backends import _macosx
RuntimeError: Python is not installed as a framework. The Mac OS X backend will not be able to function correctly if Python is not installed as a framework. See the Python documentation for more information on installing Python as a framework on Mac OS X. Please either reinstall Python as a framework, or try one of the other backends. If you are Working with Matplotlib in a virtual enviroment see 'Working with Matplotlib in Virtual environments' in the Matplotlib FAQ
```

どういう原因かは深掘りしないけど、これでなおるみたい。

```
$ mkdir ~/.matplotlib
$ vim ~/.matplotlib/matplotlibrc

backend : TkAgg
```

今度こそ実行出来るようになったハズ。

## ロジスティック回帰の実装
コードがごにょごにょ書いてあるのですが、結局プロットする部分は飛ばしてロジスティック回帰の学習と、予測だけ実装しました。

main.py

```
# -*- coding:utf-8 -*-
import numpy as np
import matplotlib.pyplot as plt
from sklearn import linear_model, datasets

iris = datasets.load_iris()
X = iris.data[:, :2]  # フィーチャーを2つに絞る
Y = iris.target

# 学習させる
logreg = linear_model.LogisticRegression(C=1e5)
logreg.fit(X, Y)
print logreg.predict([6.5, 3])
```

[6.5, 3]というパラメータを渡すと2と予測された模様。

```
$ python main.py
[2]
```

## 少しだけ解説

irisデータっていうのは機械学習ではおなじみなんですが、アヤメのデータから、アヤメの種類を分類するっていうデータです。下記リンクのようなデータが入っていて、`Iris-setosa`,`Iris-versicolor`,`Iris-virginica`という三種類に分類します。

[https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data](https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data)

```
iris = datasets.load_iris()
X = iris.data[:, :2]  # フィーチャーを2つに絞る
Y = iris.target
```

学習部分。モデルを定義して、トレーニングセットを流し込んでいる。`C`はオーバーフィッティングを避ける正規化パラメータなんだけど`1e5`ってなんだろな。

```
logreg = linear_model.LogisticRegression(C=1e5)
logreg.fit(X, Y)
```

パラメータ`[6.5, 3]`で予測し、結果を標準出力に表示する。結果が2となるので、`Iris-virginica`らしいということになります。

```
print logreg.predict([6.5, 3])
```

簡単ですね！これならアリでも簡単にサクッと機械学習出来ちゃいますね。
