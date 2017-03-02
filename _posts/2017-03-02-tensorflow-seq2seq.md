---
layout: blog
title: TensorFlowのseq2seqで英仏語翻訳のチュートリアルを試す
category: blog
tags: [機械学習,machine learning,TensorFlow,seq2seq,RNN]
summary: TODO
author: aharada
---

※この記事はまだ記入途中です。

前々回あたりで、tensorflowのseq2seqで対話システムをつくる記事を書いたのだけれど、実はまだあんまり動く動いていなくてあれこれ試行錯誤しているところです。

学習処理はうまく走らせることが出来たし、収束している様子も確認したのですが、肝心の対話を動作させてみるとまるっきり使えないっていう状態でハマってまして、一旦初心に返って公式チュートリアルからやりなおそうと思って次第。

## チュートリアル通りにやってみる

こちらが公式チュートリアル。

[https://www.tensorflow.org/tutorials/seq2seq](https://www.tensorflow.org/tutorials/seq2seq)

このあたりのエントリを参考にさせていただいた。

- [http://d.hatena.ne.jp/shi3z/20160312/1457774055](http://d.hatena.ne.jp/shi3z/20160312/1457774055)
- [https://www.slideshare.net/tak9029/tensorflowai](https://www.slideshare.net/tak9029/tensorflowai)

ローカルでも実行出来るのですが、学習データ量がかなり大きいのでGPUがある端末で動かすのが良いかと思う。ぼくはAWSのGPUインスタンスで走らせることにします。

tensorflow/modelsリポジトリからcloneして必要な箇所だけ抜き出してコピーしておく。

```
$ g clone git@github.com:tensorflow/models.git
$ cp -r models/tutorials/rnn/translate .
```

学習処理の実行。自動的に英語・フランス語翻訳データをダウンロードしてくれるのですが、すさまじく時間がかかる。ひたすら待ち続けるしかない。なにせ20GBのデータだそうで(圧縮後は2.3GBっぽい)。

```
$ python translate.py --data_dir data
```

こんな感じでコーパス作成処理を延々とやっていてまだ終わりそうもないので、一旦このへんで放置するかな。

```
Creating vocabulary data/vocab40000.to from data data/giga-fren.release2.fixed.fr
  processing line 100000
  processing line 200000
  processing line 300000
  processing line 400000
```

TODO: 続きを書く
