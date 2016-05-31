---
layout: blog
title: scikit-learnを使って線形回帰モデルをクロスバリデーションする
category: blog
tags: [python,scikit-learn,machine learning,機械学習,cross validation]  
summary: 最近もっぱらscikit-learnをいじっているのですが、クロスバリデーションってどうやるんだろう
author: aharada
---

最近もっぱらscikit-learnをいじっているのですが、クロスバリデーションってどうやるんだろうと思い調べてました。

非常にシンプルな例ですが、sckit-learnに付属しているテストデータを使って実際にやってみます。ちなみにpython2.7.10を使ってます。scikit-learnはpipでインストールするなりしてくだしあ。

main.py

```
# -*- coding: utf-8 -
from sklearn import cross_validation
from sklearn import datasets
from sklearn import linear_model

# ボストンの家の部屋数と、家の価格データ（詳しくは公式参照）
# http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_boston.html#sklearn.datasets.load_boston
boston = datasets.load_boston()

# train_test_splitを使うと簡単にトレーニングセットとテストセットを分割出来る
X_train, X_test, y_train, y_test = cross_validation.train_test_split(
    boston.data, boston.target, test_size=0.4, random_state=0)

# 学習させる
regr = linear_model.LinearRegression()
regr.fit(X_train, y_train)

# クロスバリデーションを実行
print regr.score(X_test, y_test)
```

```
$ python main.py
0.688178486968
```

68%くらいの精度ということでいいのかな。あんまり高くないですね。
