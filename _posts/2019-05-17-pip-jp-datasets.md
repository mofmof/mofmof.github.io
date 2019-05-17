---
layout: blog
title: pipで機械学習や自然言語処理用のデータをサクッと持ってきたいのでjp-datasetsというパッケージを作った
category: blog
tags: [機械学習,machine learning,pip,python,教師データ]
summary: TODO
author: aharada
# image:
---

データをサクッと取ってきて機械学習とか自然言語処理に使えるパッケージが欲しかったので作ってみました。

前回の続き。

[pipでGitHub経由でインストール出来る自作パッケージを作る](/blog/pip-package.md)

## 開発モードでローカルのソースコードをpip installする

前回は、`pip install git+https://github.com/<your_github_id>/sample-dt`のようにGitHubのリポジトリを指定してインストールしていましたが、これだと呼び出し側からテストしようとすると毎回`git push`して`pip install`し直さなければならなくてつらそうだったので、ローカルのソースコードをインストールする方法を調べた。

参考

 [https://python-packaging-user-guide-ja.readthedocs.io/ja/latest/installing.html](https://python-packaging-user-guide-ja.readthedocs.io/ja/latest/installing.html) 

このようにinstallすれば開発モードでローカルのソースコードからインストール出来る。

```
$ pip install -e /path/to/jp-datasets
```

前回`say.hoo()`を実行すると`hoo`と標準出力されるサンプルを作ったので正しく呼び出せるかを確認しておく。ソースコードを編集した場合は、一度pythonを終了して再度起動すれば更新が反映される。

```
$ python
>>> from jpdatasets.say import hoo
>>> hoo()
hoo
```

## ネガポジデータを取得して返す

適当にTwitterを検索して収集したデータに適当にネガポジラベリングしたデータ。とりあえず動かしてみる程度には使えるだろう。

jpdatasets/data/polarity.csv

```
text,"polarity(0: neutral, 1:negative, 2:positive)"
日本でこのスタイル 流行ってほしいな,0
選ばれたのは綾鷹でした,0
綾鷹杯このメンバーででます(^^)よろしくお願いします(^^♪,2
綾鷹しかなかった,1
イエモンのライブチケット買った！！最近めちゃハマってる♬,2
"あれ？ちび昨日アークしてたけど、イエモン垢のか？ってなった。
まいっかw",0
イエモンしこたま歌うぞ,2
今日は新宿のYOSHIKIさんに会うことができました！,2
「HIDEやTAIJIの素晴らしさは、XJAPANが頑張れば頑張るほど伝わる」世代を超えて2人を敬う人が増えてる証拠がここにあるんですね。,2
でもこういう爽やかで大人しそうな奴に限ってモラハラ男であることを私は知っている,1
モラハラ夫の名言『お前のせいで俺がイライラする』※それなら別れてくれたらいいのに。,1
理系院卒が優良物件というけれど実際のところ環境の影響で大人しくなってるだけでモラハラ系拗らせ男子とか無視できない程度に多い,1
ああでもしなきゃ結婚できないくらいモラハラが日本にはわんさかいた。今も、たくさん。,1
坂本おめでとう！生まれてきてくれてありがとう,2
すっごく遅れたけどひなたお誕生日おめでとう！！！⸜(*ˊᵕˋ* )⸝,2
伊吹亜門さん、おめでとうございます,2
「カワセミハッグ」,0
しんどいくらい悲しいw,1
待って星型(？)のピュレグミあったわ,2
グミ・チョコレート・パイン,0
レモングミ,0
肉球グミはおいしかったです!,2
グミも止まらん～～！,2
本物に会えて喜ぶモモコさんがかわいすぎる！！！,2
オジャマちゃんに差し入れしたいけど何がいいの？,0
巨大深海魚グミ作ってみた！メガマウスザメ・ダイオウグソクムシ・オウムガイ,2
ピュレグミとまらん,2
よりみちなうなうー　グミかってこー,0
マックシェイクS 無料だから飲んじゃった,0
今日めちゃくちゃメイクうまくいった,2
こないだ、結構美味しいと教えてもらい、そういえばたまに見かけるけど食べた事なかったな…って思って購入。即食べ。,2
タイトル：テトリス,0
全消ししてテトリスうまおになった,2
無事離婚できました！,2
10連休トラブル集、殺意・離婚危機や「BBQ最悪」の声も,1
離婚して子供と会えなくなった経験があります,1
子育てってさー私には限界があるわ,1
今まで付き合ってきた人には、その人の好きなタイプに合わせる事ばかり意識して少しでも良い女にと頑張ってたけどそれが失敗だったと実感。,1
裁判で動かぬ証拠を突きつけ慰謝料請求した途端元に戻りたい償いたい私達が居ないと生きてけないって…再構築の連絡,1
```

当初の設計では、スプレッドシートから取得しようと思ってた。そしたらリアルタイムにデータを編集出来るからいいなと思ったけど、キー情報が必要で手軽に動かせなくなるからパッケージにデータを内包する形にした。大きめのデータを扱うときにはムリがあるのでいつか考えよう。

この辺のページを見るとルート直下のdataディレクトリに入れておけばいいらしい、と思ったんだけど、これは/usr/配下とかPCのどっかに設定ファイルを展開したい場合に使うみたい。シェルのコマンドとかを作る場合とかに使いそう。

- [Python: 自作パッケージにデータファイルを含める - CUBE SUGAR CONTAINER](https://blog.amedama.jp/entry/2015/12/26/012332)
- [setup.pyのdata_files引数を使ってデータファイルを/usr以下に配布する - Qiita](https://qiita.com/ysk24ok/items/55ac1f17eb7f1b71f757)

実装途中、なかなかファイルのパスを見つけられなくて苦労した。pipパッケージ内でデータファイルを読み込みたい場合は、単にルートからファイルパスを指定するだけではダメみたい。

```
raceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/harada/source_code/jp-datasets/jpdatasets/polarity.py", line 6, in load_data
    skiprows=1
  File "/Users/harada/.pyenv/versions/3.6.4/lib/python3.6/site-packages/numpy/lib/npyio.py", line 955, in loadtxt
    fh = np.lib._datasource.open(fname, 'rt', encoding=encoding)
  File "/Users/harada/.pyenv/versions/3.6.4/lib/python3.6/site-packages/numpy/lib/_datasource.py", line 266, in open
    return ds.open(path, mode, encoding=encoding, newline=newline)
  File "/Users/harada/.pyenv/versions/3.6.4/lib/python3.6/site-packages/numpy/lib/_datasource.py", line 624, in open
    raise IOError("%s not found." % path)
OSError: data/polarity.csv not found.
```

このようにパスを取ってくる必要がある。

```
d = os.path.dirname(sys.modules['jpdatasets'].__file__)
file_path = os.path.join(d, 'data/polarity.csv')
```

numpyの`loadtxt`や`genfromtxt`でハマりまくった。日本語を含むデータを読み込もうとすると2文字で切れてしまったり、nanになってしまったり、空文字になってしまったりしてどうにもならない。諦めて`csv.reader`で1行ずつ読みこんで、最終的にこんなコードになった。

ちなみに`load_data`の引数と戻り値のインタフェースは[kerasのdatasets](https://github.com/keras-team/keras/tree/master/keras/datasets)を参考にしている。

```
import os, sys
import numpy as np
import csv

def load_data(seed=0):
    d = os.path.dirname(sys.modules['jpdatasets'].__file__)
    file_path = os.path.join(d, 'data/polarity.csv')

    with open(file_path) as f:
        r = csv.reader(f, delimiter=",", doublequote=True, lineterminator="\r\n", quotechar='"', skipinitialspace=True)
        next(r)  # skip header
        data = [row for row in r]

    np.random.seed(seed=seed)
    np.random.shuffle(data)
    
    train, test = np.split(data, [int(len(data) * 0.7)])

    x_train = train[:, 0]
    y_train = train[:, 1].astype(int)
    x_test = test[:, 0]
    y_test = test[:, 1].astype(int)

    return (x_train, y_train), (x_test, y_test)
```

やっと出来たああああ！

```
((array(['イエモンのライブチケット買った！！最近めちゃハマってる♬', 'マックシェイクS 無料だから飲んじゃった',
       '今日めちゃくちゃメイクうまくいった', '無事離婚できました！', '10連休トラブル集、殺意・離婚危機や「BBQ最悪」の声も',
       '巨大深海魚グミ作ってみた！メガマウスザメ・ダイオウグソクムシ・オウムガイ',
       'モラハラ夫の名言『お前のせいで俺がイライラする』※それなら別れてくれたらいいのに。', 'グミも止まらん～～！',
       '理系院卒が優良物件というけれど実際のところ環境の影響で大人しくなってるだけでモラハラ系拗らせ男子とか無視できない程度に多い',
       'よりみちなうなうー\u3000グミかってこー', '待って星型(？)のピュレグミあったわ',
       '伊吹亜門さん、おめでとうございます', '綾鷹杯このメンバーででます(^^)よろしくお願いします(^^♪',
       '裁判で動かぬ証拠を突きつけ慰謝料請求した途端元に戻りたい償いたい私達が居ないと生きてけないって…再構築の連絡', 'レモングミ',
       '子育てってさー私には限界があるわ', '「カワセミハッグ」', '離婚して子供と会えなくなった経験があります',
       '「HIDEやTAIJIの素晴らしさは、XJAPANが頑張れば頑張るほど伝わる」世代を超えて2人を敬う人が増えてる証拠がここにあるんですね。',
       '坂本おめでとう！生まれてきてくれてありがとう', 'あれ？ちび昨日アークしてたけど、イエモン垢のか？ってなった。\nまいっかw',
       'しんどいくらい悲しいw', 'すっごく遅れたけどひなたお誕生日おめでとう！！！⸜(*ˊᵕˋ* )⸝',
       '全消ししてテトリスうまおになった', '今日は新宿のYOSHIKIさんに会うことができました！', 'タイトル：テトリス',
       '選ばれたのは綾鷹でした'], dtype='<U69'), array([2, 0, 2, 2, 1, 2, 1, 2, 1, 0, 2, 2, 2, 1, 0, 1, 0, 1, 2, 2, 0, 1,
       2, 2, 2, 0, 0])), (array(['ピュレグミとまらん', 'ああでもしなきゃ結婚できないくらいモラハラが日本にはわんさかいた。今も、たくさん。',
       'こないだ、結構美味しいと教えてもらい、そういえばたまに見かけるけど食べた事なかったな…って思って購入。即食べ。',
       'オジャマちゃんに差し入れしたいけど何がいいの？', 'イエモンしこたま歌うぞ',
       '本物に会えて喜ぶモモコさんがかわいすぎる！！！', '肉球グミはおいしかったです!', 'グミ・チョコレート・パイン',
       'でもこういう爽やかで大人しそうな奴に限ってモラハラ男であることを私は知っている',
       '今まで付き合ってきた人には、その人の好きなタイプに合わせる事ばかり意識して少しでも良い女にと頑張ってたけどそれが失敗だったと実感。',
       '綾鷹しかなかった', '日本でこのスタイル 流行ってほしいな'], dtype='<U69'), array([2, 1, 2, 0, 2, 2, 2, 0, 1, 1, 1, 0])))
>>>
```

ソースコードはこちらから全て見ることが出来ます。

[https://github.com/harada4atsushi/jp-datasets](https://github.com/harada4atsushi/jp-datasets)