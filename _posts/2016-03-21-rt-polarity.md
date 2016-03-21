---
layout: blog
title: mecabで形態素解析したものをマッピングオブジェクトに突っ込む
category: blog
tags: [機械学習,machine learning,mecab,形態素解析]  
summary: 機械学習に絶賛ハマり中の原田です。機械学習でこんなあんなことやことできないかなーって色々考えるのですが、
author: aharada
---
http://www.cl.ecei.tohoku.ac.jp/nlp100/

前回に続けて文章を機械学習のインプットにしたいので、文章解析処理を学習するため「[言語処理100本ノック](http://www.cl.ecei.tohoku.ac.jp/nlp100/)」をやります。今回は一気に飛ばして第8章の機械学習にいってみます。

前回の「[mecabで形態素解析したものをマッピングオブジェクトに突っ込む](http://tech.mof-mof.co.jp/blog/mecab-install.html)」はmecabで形態素解析を試してました。

## 70. データの入手・整形
引き続きpythonで書いていきます。

> 文に関する極性分析の正解データを用い，以下の要領で正解データ（sentiment.txt）を作成せよ．

> rt-polarity.posの各行の先頭に"+1 "という文字列を追加する（極性ラベル"+1"とスペースに続けて肯定的な文の内容が続く）
> rt-polarity.negの各行の先頭に"-1 "という文字列を追加する（極性ラベル"-1"とスペースに続けて否定的な文の内容が続く）
>上述1と2の内容を結合（concatenate）し，行をランダムに並び替える
> sentiment.txtを作成したら，正例（肯定的な文）の数と負例（否定的な文）の数を確認せよ．

これは比較的簡単ですね。こんな感じで実装した。

make_sentiment_txt.py

```
# -*- coding: utf-8 -
import random

def read_polarity():
    lines = []

    for line in open('rt-polarity.pos', 'r'):
      lines.append("+1 " + line)

    for line in open('rt-polarity.neg', 'r'):
      lines.append("-1 " + line)

    random.shuffle(lines)
    return lines

def write_sentiment(lines):
    f = open('sentiment.txt', 'w')
    for line in lines:
        f.write(line)
    f.close()

write_sentiment(read_polarity())
```

こんな感じのファイルが出来上がります。

sentiment.txt

```
+1 if a big musical number like 'praise the lord , he's the god of second chances' doesn't put you off , this will be an enjoyable choice for younger kids .
+1  . . . a rich and intelligent film that uses its pulpy core conceit to probe questions of attraction and interdependence and how the heart accomodates practical needs . it is an unstinting look at a collaboration between damaged people that may or may not qual
-1 the essential problem in orange county is that , having created an unusually vivid set of characters worthy of its strong cast , the film flounders when it comes to giving them something to do .
+1 it's a count for our times .
+1 laced with liberal doses of dark humor , gorgeous exterior photography , and a stable-full of solid performances , no such thing is a fascinating little tale .
-1 an alternately raucous and sappy ethnic sitcom . . . you'd be wise to send your regrets .
-1 the movie is a negligible work of manipulation , an exploitation piece doing its usual worst to guilt-trip parents .
-1 the film's bathos often overwhelms what could have been a more multifaceted look at this interesting time and place .
+1 led by griffin's smartly nuanced performance and enthusiasm , the cast has a lot of fun with the material .
-1 lisa rinzler's cinematography may be lovely , but love liza's tale itself virtually collapses into an inhalant blackout , maintaining consciousness just long enough to achieve callow pretension .
-1 the art direction is often exquisite , and the anthropomorphic animal characters are beautifully realized through clever makeup design , leaving one to hope that the eventual dvd release will offer subtitles and the original italian-language soundtrack .
+1 smith profiles five extraordinary american homes , and because the owners seem fully aware of the uses and abuses of fame , it's a pleasure to enjoy their eccentricities .
+1 that rare film whose real-life basis is , in fact , so interesting that no embellishment is needed .
-1 pretentious editing ruins a potentially terrific flick .
-1 the entire movie is so formulaic and forgettable that it's hardly over before it begins to fade from memory .

... 省略
```

## 71. ストップワード

> 英語のストップワードのリスト（ストップリスト）を適当に作成せよ．さらに，引数に与えられた単語（文字列）がストップリストに含まれている場合は真，それ以外は偽を返す関数を実装せよ．さらに，その関数に対するテストを記述せよ．

ストップワードとは、is, be, a, the などのように検索精度を向上させるために検索対象から除外せざるを得ない語のことを指します。日本語でいうと、「は」「これ」「です」「ます」とかそういうの。

ちょっと検索してみたら何かしらの団体とか組織が公式にストップワードのリストを提供しているみたい。この辺とか。

[https://code.google.com/archive/p/stop-words/source](https://code.google.com/archive/p/stop-words/source)

適当に作成しろと書いてあるので、どっかのサイトに掲載されていたストップワードの一覧をコピペさせてもらった。ライブラリとかAPIで提供されてそうな気もするけどまあいいか。

stop_word.py

```
# -*- coding: utf-8 -

class StopWord:
    def __init__(self):
        self.list = [
            "a", "did", "in", "only", "then", "where",
            "all", "do", "into", "onto", "there", "whether",
            "almost", "does", "is", "または", "therefore", "which",
            "also", "either", "it", "our", "these", "while",
            "although", "for", "its", "ours", "they", "who",
            "an", "from", "just", "s", "this", "whose",
            "had", "ll", "shall", "those", "why",
            "any", "has", "me", "she", "though", "will",
            "are", "have", "might", "should", "through", "with",
            "as", "having", "Mr", "since", "thus", "would",
            "at", "he", "Mrs", "so", "to", "yet",
            "be", "her", "Ms", "some", "too", "you",
            "because", "here", "my", "still", "until", "your",
            "been", "hers", "no", "such", "ve", "yours",
            "both", "him", "non", "t", "very",
            "but", "his", "nor", "than", "was",
            "by", "how", "not", "that", "we",
            "can", "however", "of", "the", "were",
            "could", "i", "on", "their", "what",
            "d", "if", "one", "them", "when"
        ]

    def exists(self, str):
        if str in self.list:
            return True
        else:
            return False
```

試してみる。

runner.py

```
# -*- coding: utf-8 -
from stop_word import StopWord

stop_word = StopWord()
print stop_word.exists("is")
print stop_word.exists("hoge")
```

結果

```
$ python runner.py
True
False
```

## 72. 素性抽出

> 72. 素性抽出
> 極性分析に有用そうな素性を各自で設計し，学習データから素性を抽出せよ．素性としては，レビューからストップワードを除去し，各単語をステミング処理したものが最低限のベースラインとなるであろう．

ここからが正念場っぽい。素性抽出することで機械学習にインプットさせるべき単語を見つけるのかな。パラメータ化の一歩手前くらいなのかな。

ちょっと問題を見た感じサクっと出来そうにはないので、一旦この辺で区切る。
