---
layout: blog
title: mecabで形態素解析したものをマッピングオブジェクトに突っ込む
category: blog
tags: [機械学習,machine learning,mecab,形態素解析]  
summary: 機械学習に絶賛ハマり中の原田です。機械学習でこんなあんなことやことできないかなーって色々考えるのですが、
author: aharada
---

機械学習に絶賛ハマり中の原田です。

機械学習でこんなあんなことやことできないかなーって色々考えるのですが、結構文章を解析してごにょごにょする必要があるケースが多くて、そうすると日本語をパラメータにして機械学習させることになるので、形態素解析はやらねばいかんなと思い入門してみた。

## mecabのインストール
もちろん言語はpythonです。mac osx使っているのでhomebrewでサクっとインストールしちゃいます。

インストールに関してはほぼこちらの通り。

[http://qiita.com/ysk_1031/items/7f0cfb7e9e4c4b9129c9]([http://qiita.com/ysk_1031/items/7f0cfb7e9e4c4b9129c9)

```
$ brew install mecab
$ brew install mecab-ipadic
```

あのロマサガの名言で試してみます。特に問題なくサクっと形態素解析できちゃいましたね。

```
$ mecab
皇帝陛下ですね？私は七英雄の一人　ノエルと申します。妹ロックブーケのカタキです。殺らせていただきます。
な、なにをするきさまらー
皇帝    名詞,一般,*,*,*,*,皇帝,コウテイ,コーテイ
陛下    名詞,一般,*,*,*,*,陛下,ヘイカ,ヘイカ
です    助動詞,*,*,*,特殊・デス,基本形,です,デス,デス
ね      助詞,終助詞,*,*,*,*,ね,ネ,ネ
？      記号,一般,*,*,*,*,？,？,？
私      名詞,代名詞,一般,*,*,*,私,ワタシ,ワタシ
は      助詞,係助詞,*,*,*,*,は,ハ,ワ
七      名詞,数,*,*,*,*,七,ナナ,ナナ
英雄    名詞,一般,*,*,*,*,英雄,エイユウ,エイユー
の      助詞,連体化,*,*,*,*,の,ノ,ノ
一      名詞,数,*,*,*,*,一,イチ,イチ
人      名詞,接尾,助数詞,*,*,*,人,ニン,ニン
　      記号,空白,*,*,*,*,　,　,　
ノエル  名詞,一般,*,*,*,*,ノエル,ノエル,ノエル
と      助詞,格助詞,引用,*,*,*,と,ト,ト
申し    動詞,自立,*,*,五段・サ行,連用形,申す,モウシ,モーシ
ます    助動詞,*,*,*,特殊・マス,基本形,ます,マス,マス
。      記号,句点,*,*,*,*,。,。,。
妹      名詞,一般,*,*,*,*,妹,イモウト,イモート
ロック  名詞,サ変接続,*,*,*,*,ロック,ロック,ロック
ブーケ  名詞,一般,*,*,*,*,ブーケ,ブーケ,ブーケ
の      助詞,連体化,*,*,*,*,の,ノ,ノ
カタキ  名詞,一般,*,*,*,*,*
です    助動詞,*,*,*,特殊・デス,基本形,です,デス,デス
。      記号,句点,*,*,*,*,。,。,。
殺ら    動詞,自立,*,*,五段・ラ行,未然形,殺る,ヤラ,ヤラ
せ      動詞,接尾,*,*,一段,連用形,せる,セ,セ
て      助詞,接続助詞,*,*,*,*,て,テ,テ
いただき        動詞,非自立,*,*,五段・カ行イ音便,連用形,いただく,イタダキ,イタダキ
ます    助動詞,*,*,*,特殊・マス,基本形,ます,マス,マス
。      記号,句点,*,*,*,*,。,。,。
EOS

な      助詞,終助詞,*,*,*,*,な,ナ,ナ
、      記号,読点,*,*,*,*,、,、,、
なに    名詞,代名詞,一般,*,*,*,なに,ナニ,ナニ
を      助詞,格助詞,一般,*,*,*,を,ヲ,ヲ
する    動詞,自立,*,*,サ変・スル,基本形,する,スル,スル
き      助動詞,*,*,*,文語・キ,基本形,き,キ,キ
さま    名詞,一般,*,*,*,*,さま,サマ,サマ
ら      名詞,接尾,一般,*,*,*,ら,ラ,ラ
ー      名詞,一般,*,*,*,*,*
EOS
```

## 言語処理100本ノック第4章ちょっとだけやってみる

これだけじゃあっさりしすぎなので、もう少し進めてみます。言語処理プログラミングのトレーニングの定番？「[言語処理100本ノック](http://www.cl.ecei.tohoku.ac.jp/nlp100/)」をやります。

第4章は夏目漱石の「吾輩は猫である」の本文をを形態素解析してあれこれするトレーニングです。まずは形態素解析された結果を別ファイルに出力します。ファイルは100本ノックに置いてあるのでダウンロードしておく。

```
$ mecab neko.txt -o neko.txt.mecab
```

解析結果はこんな感じになっている(一部)

neko.txt.mecab

```
一	名詞,数,*,*,*,*,一,イチ,イチ
EOS
EOS
　	記号,空白,*,*,*,*,　,　,　
吾輩	名詞,代名詞,一般,*,*,*,吾輩,ワガハイ,ワガハイ
は	助詞,係助詞,*,*,*,*,は,ハ,ワ
猫	名詞,一般,*,*,*,*,猫,ネコ,ネコ
で	助動詞,*,*,*,特殊・ダ,連用形,だ,デ,デ
ある	助動詞,*,*,*,五段・ラ行アル,基本形,ある,アル,アル
。	記号,句点,*,*,*,*,。,。,。
EOS
名前	名詞,一般,*,*,*,*,名前,ナマエ,ナマエ
は	助詞,係助詞,*,*,*,*,は,ハ,ワ
まだ	副詞,助詞類接続,*,*,*,*,まだ,マダ,マダ
無い	形容詞,自立,*,*,形容詞・アウオ段,基本形,無い,ナイ,ナイ
。	記号,句点,*,*,*,*,。,。,。
EOS
EOS
　	記号,空白,*,*,*,*,　,　,　
どこ	名詞,代名詞,一般,*,*,*,どこ,ドコ,ドコ
で	助詞,格助詞,一般,*,*,*,で,デ,デ
生れ	動詞,自立,*,*,一段,連用形,生れる,ウマレ,ウマレ
た	助動詞,*,*,*,特殊・タ,基本形,た,タ,タ
か	助詞,副助詞／並立助詞／終助詞,*,*,*,*,か,カ,カ
とんと	副詞,一般,*,*,*,*,とんと,トント,トント
見当	名詞,サ変接続,*,*,*,*,見当,ケントウ,ケントー
が	助詞,格助詞,一般,*,*,*,が,ガ,ガ
つか	動詞,自立,*,*,五段・カ行イ音便,未然形,つく,ツカ,ツカ
ぬ	助動詞,*,*,*,特殊・ヌ,基本形,ぬ,ヌ,ヌ
。	記号,句点,*,*,*,*,。,。,。
EOS
```

30番の課題です。

> 30. 形態素解析結果の読み込み
> 形態素解析結果（neko.txt.mecab）を読み込むプログラムを実装せよ．ただし，各形態素は表層形（surface），基本形（base），品詞（pos），品詞細分類1（pos1）をキーとするマッピング型に格納し，1文を形態素（マッピング型）のリストとして表現せよ．第4章の残りの問題では，ここで作ったプログラムを活用せよ．

ちょっとどういう状態を作ればいいのかピンとこないんだけど、jsonで言うと下記のようなマッピングオブジェクトを作れということだと仮定して進める。

surface,base,pos,pos1キーを持つマッピングが行ごとの配列に格納され、更にその行を全て配列にまとめて文章全体を表現する構造になっている。

```
[
  [
    {
      surface: '皇帝',
      base: '皇帝',
      pos: '名刺',
      pos1: '一般'
    },
    {
      surface: '陛下',
      base: '陛下',
      pos: '名刺',
      pos1: '一般'
    },
    {
      省略
    }
  ],
  [
    {
      surface: 'な',
      base: 'な',
      pos: '助詞',
      pos1: '終助詞'
    },    
    {
      surface: '、',
      base: '、',
      pos: '記号',
      pos1: '読点'
    },
    {
      省略
    }
  ]
]
```

こんな感じのコードになった。

try-mecab.py

```
# -*- coding: utf-8 -
lines = []
morphemes = []

for line in open('neko.txt.mecab', 'r'):
  list = line.split('\t')
  surface = list[0]

  if surface == "EOS\n":
    morphemes = []
    lines.append(morphemes)

  else:
    values = list[1].split(",")
    dict = {
      "surface": surface,
      "base": values[6],
      "pos": values[0],
      "pos1": values[1]
    }
    morphemes.append(dict)

print("surface: " + lines[1][1]['surface'])
print("base: " + lines[1][1]['base'])
print("pos: " + lines[1][1]['pos'])
print("pos1: " + lines[1][1]['pos1'])
```

実行。

```
$ python try-mecab.py
surface: 吾輩
base: 吾輩
pos: 名詞
pos1: 代名詞
```

本当は配列全体をprintしたかったんだけど、pythonでは日本語の格納されたオブジェクトをprintしてもunicodeのコードで表示されてしまうのね。デバッグしづらいなー。
