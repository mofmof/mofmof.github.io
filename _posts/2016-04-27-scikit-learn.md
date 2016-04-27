---
layout: blog
title: scikit-learnを使ってテキストから素性ベクトルを取得する
category: blog
tags: [python,scikit-learn,machine learning,機械学習]  
summary: 機械学習をやっていると、実際に何か使えるものをサクっと作りたくなってくるんですが、膨大なデータが必要だったり
author: aharada
---

機械学習をやっていると、実際に何か使えるものをサクっと作りたくなってくるんですが、膨大なデータが必要だったり、実装ボリュームが大きくなりすぎたりするんですが、テキスト解析ならちょうど良いノリで出来そう。

そこで、TwitterのツイートをSVMにかけてネガポジ判定するコードを書いてみたいと思います。今回はひとまず、ツイートをinputとして素性抽出してベクトル化するところまでやってみます。

こちらの記事に大変お世話になった。コードはかなりコピペさせていただいた。

[https://datumstudio.jp/backstage/662](https://datumstudio.jp/backstage/662)

## 素性抽出とは

実装に入る前に「素性抽出」について理解を確認したい。

通常、機械学習にかけるとき、当然学習のためのデータを流し込む必要があるわけなのですが、一般的には機械が理解できる形式として数値に置き換える必要があります。テキストはそのままでは機械が認識することが出来ないので「素性抽出」という作業を経て数値に置き換えられます。

素性抽出は様々なやり方があるのですが、一般的な(？)やり方にBag of Wordsというものがあります。これは各テキストに対して特徴語の出現頻度をベクトル化するやり方です。

例えば、下記のような3つの文章がinputされて、特徴語を「amet」「quam」「elit」とします。

```
Lorem ipsum dolor sit quam amet, consectetur adipiscing elit.
Maecenas a elit viverra risus, at consequat ipsum.
Aliquam condimentum turpis sit quam amet mauris convallis, ut lacinia quam eleifend.
```

これにBoWで素性ベクトルを作ると以下のようになります。

```
[
  [1, 1, 1],  # ametが1回、quamが1回、elitが1回出現する
  [0, 0, 1],  # ametが0回、quamが0回、elitが1回出現する
  [1, 2, 0]   # ametが1回、quamが2回、elitが0回出現する
]
```

素性抽出とBag of Wordsについて参考

[http://stmind.hatenablog.com/entry/2013/11/04/164608](http://stmind.hatenablog.com/entry/2013/11/04/164608)

## 実装
inputデータ。1列目はテキスト、2列目の1はポジティブ、-1はネガティブを表します。実際にはもっと多くのデータがないと精度が出ません。ちなみにこの文章はぼくのフォロワーさんのTweetのコピペです。

data.tsv

```
もう何がなんだかわからない一年戦争後	-1
面白い高校生youtuberが、大学受験とかで配信減らすと、あぁ、みんな受験を経て普通の人になっちゃうのか、と物悲しくなる。学歴にもデメリットがあるのになー。	-1
rubocopは英語帝国主義なのがとても嫌いだったな	-1
三菱なのに燃費がいい炎上	-1
ねこ「ほかのねこのにおいするでくんかくんか」	1
なんちゃって(๑´ڡ`๑)	1
亜人ちゃんは語りたいのほうもいい。	1
ミス東工大が可愛すぎるから東工大	1
ハリッサ？ 唐辛子のやーつ！	1
さて、今夜はカツヲのお刺身！	1
浮名を流すカーチャン( ‘н‘*)	1
実用ではなくて「コミュニケーションのきっかけとしての質問」てちょっと面白いな。(mittomoと教えてgooに対する感想	1
実際問題、工場さえあれば量産できちゃう米帝チートすぎるでしょ	-1
2日シャンプー使わずシャワー長めにしただけで髪の毛バッサバサ	-1
（´-`三´-`）はんにゃらららーばんにゃららら〜！！！(まめさんありがとうございましたの舞	1
リア充になろうと頑張ったオタク何人か見たけどなれた人を知らないんだよなぁ。。変えるなら大学入る前だなw	-1
行くぜ！おおさかぁー！！	1
この季節だからかおっさんも湧いてるな	-1
めっちゃ楽しかったわ〜！いいお酒飲めたわ(ﾉ∀`)	1
またわけがわからんエラーにまみれている。	-1
お、オジタンもご帰還( ‘н‘*)	1
本当世の中すげー人いるんだなって感心してる	1
DVDドライブくんがクソの役にも立たなくて、十秒くらい再生したら停止しやがる	-1
これ買ってよかった	1
かわいい	1
「大企業がオワコン」みたいなのは、別にぼくに限らず主張していることですし、東芝、シャープ、三菱自動車しかり、事例ベースで明らかな話。そもそも働いていて「こりゃ終わってる」と感じるでしょうし。今はまだ牧歌的な時代なんでしょうねぇ。	-1
ゲーセンめちゃめちゃ暑い7月並みか	-1
```

runner.py

```
# -*- coding: utf-8 -

import MeCab
import pandas as pd
import numpy as np
from sklearn import svm
from sklearn.grid_search import GridSearchCV
from sklearn.feature_extraction.text import CountVectorizer

# 参考サイトのコピペ
# 文章をmecabで分かちがきして、名詞・動詞・形容詞の単語一覧を返す
def wakati(text):
    tagger = MeCab.Tagger()
    #text = text.encode("utf-8")
    node = tagger.parseToNode(text)
    word_list = []
    while node:
        pos = node.feature.split(",")[0]
        if pos in ["名詞", "動詞", "形容詞"]:
            lemma = node.feature.split(",")[6].decode("utf-8")
            if lemma == u"*":
                lemma = node.surface.decode("utf-8")
            word_list.append(lemma)
        node = node.next
    return u" ".join(word_list[1:-1])


def parse():
    lines = []
    for line in open('data.tsv', 'r'):
        arr = line.split("\t")
        lines.append(arr)
    return lines

wakatis = []
for line in parse():
  wakatis.append(wakati(line[0]))

# テキスト内の単語の出現頻度を数えて、結果を素性ベクトル化する(Bag of words)
count_vectorizer = CountVectorizer()
feature_vectors = count_vectorizer.fit_transform(wakatis)  # csr_matrix(疎行列)が返る
print 'word数: ' + len(feature_vectors.toarray()[0])

# print feature_vectors
  # (0, 24)       1
  # (0, 42)       1
  # (1, 58)       1
  # (1, 2)        1
  # (1, 38)       1
  # (1, 35)       2
  # (1, 54)       1
  # (1, 47)       1
  # (1, 22)       1
  # (1, 50)       1
  # (1, 44)       1
  # (1, 14)       2
  # ...

# 素性ベクトルの中身を確認する
#  print feature_vectors.toarray()
# [[0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0
#   0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0]
#  [0 0 1 1 0 0 0 0 0 0 1 0 0 0 2 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 2 0
#   0 1 1 0 0 0 0 1 0 0 1 0 1 1 0 0 0 1 0 0 0 1 0]
#  [0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0
#   0 0 0 0 1 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0]
#  [0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0
#   0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0]
# ...

# 素性ベクトルに対応する単語の一覧を取得する
vocabulary = count_vectorizer.get_feature_names()
```

homebrewでmecabをインストール。ちなみにpythonはmacデフォのpythonをそのまま使います。

```
$ brew install mecab
$ brew install mecab-ipadic
```

pandasやscikit-learnなど必要なパッケージをpipでインストールする。

```
$ pip install mecab-python
$ pip install pandas
$ pip install numpy
$ pip install scipy
$ pip install scikit-learn
```

ちなみにmecab-ipadicをインストールしていないと下記エラーになるで気をつけて。

```
File "/usr/local/lib/python2.7/site-packages/MeCab.py", line 307, in __init__
    this = _MeCab.new_Tagger(*args)
RuntimeError
```

実行するコマンド。

```
$ python runner.py
```

feature_vectorsが素性ベクトル化されたfeatureになるので、次はこれをパラメータにしてSVMを走らせたい。
