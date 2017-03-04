---
layout: blog
title: TensorFlowのseq2seqで英仏語翻訳のチュートリアルを試す①
category: blog
tags: [機械学習,machine learning,TensorFlow,seq2seq,RNN]
summary: 前々回あたりで、tensorflowのseq2seqで対話システムをつくる記事を書いたのだけれど、
author: aharada
---

[前々回](http://tech.mof-mof.co.jp/blog/seq2seq.html)あたりで、tensorflowのseq2seqで対話システムをつくる記事を書いたのだけれど、実はまだあんまり動く動いていなくてあれこれ試行錯誤しているところです。

学習処理はうまく走らせることが出来たし、収束している様子も確認したのですが、肝心の対話を動作させてみるとまるっきり使えないっていう状態でハマってまして、一旦初心に返って公式チュートリアルからやりなおそうと思って次第。

## チュートリアル通りにやってみる

こちらが公式チュートリアル。

[https://www.tensorflow.org/tutorials/seq2seq](https://www.tensorflow.org/tutorials/seq2seq)

このあたりのエントリを参考にさせていただいた。

- [http://d.hatena.ne.jp/shi3z/20160312/1457774055](http://d.hatena.ne.jp/shi3z/20160312/1457774055)
- [https://www.slideshare.net/tak9029/tensorflowai](https://www.slideshare.net/tak9029/tensorflowai)

ローカルでも実行出来るのですが、学習データ量がかなり大きいのでGPUがある端末で動かすのが良いかと思う。ぼくはAWSのGPUインスタンスで走らせることにします。

AWSのGPUインスタンスで`tensorflow`が動かせるようにする設定はこちらのエントリ参照。

[http://tech.mof-mof.co.jp/blog/tensorflow-aws-gpu.html](http://tech.mof-mof.co.jp/blog/tensorflow-aws-gpu.html)

tensorflow/modelsリポジトリからcloneして必要な箇所だけ抜き出してコピーしておく。

```
$ g clone git@github.com:tensorflow/models.git
$ cp -r models/tutorials/rnn/translate .
```

学習処理の実行。自動的に英語・フランス語翻訳データをダウンロードしてくれるのですが、すさまじく時間がかかる。ひたすら待ち続けるしかない。なにせ20GBのデータだそうで(圧縮後は2.3GBっぽい)。

```
$ cd translate
$ mkdir data
$ mkdir train
$ python translate.py --data_dir data --train_dir train
```

こんな感じでコーパス作成処理を延々とやっていてまだ終わりそうもないので、しばらく放置する。

```
Creating vocabulary data/vocab40000.to from data data/giga-fren.release2.fixed.fr
  processing line 100000
  processing line 200000
  processing line 300000
  processing line 400000

  略
```

めっちゃ時間がかかるので朝まで放置してから見てみると`Error running translate.py - 'module' object has no attribute 'GRUCell'`って感じの例外で落ちてた。。orz

原因はtensorflowのバージョン違いだったので、最新のtensorflow1.0にアップデートします。GPUインスタンスを使っているのでtensorflow-gpuですね。

```
$ pip uninstall tensorflow-gpu
$ pip install tensorflow-gpu
```

再び実行。またひたすら待たされるですよ。。

```
$ python translate.py --data_dir data --train_dir train
...
Reading development and training data (limit: 0).
  reading data line 100000
  reading data line 200000
  reading data line 300000
  reading data line 400000
  reading data line 500000
  reading data line 600000
  reading data line 700000
  reading data line 800000
  reading data line 900000
  reading data line 1000000

  略

  reading data line 14100000
  reading data line 14200000
  reading data line 14300000
  reading data line 14400000
  強制終了
```

強制終了ってなんや。。ぐぬぬ。。

実行中に`free`コマンドでメモリ容量みていたらメモリが枯渇していた模様。
`g2.2xlarge`インスタンスはメモリ15GBだったので、60GB使える`g2.8xlarge`インスタンスに変更。

```
$ free
              total        used        free      shared  buff/cache   available
Mem:       15399000    14983344       79952      272584      335704        7760
Swap:      
```

もっかい実行。

```
$ python translate.py --data_dir data --train_dir train

略...

ResourceExhaustedError (see above for traceback): OOM when allocating tensor with shape[64,40,1,1024]
         [[Node: gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/mul_grad/mul_1 = Mul[T=DT_FLOAT, _device="/job:localhost/replica:0/task:0/gpu:0"](embedding_attention_seq2seq/embedding_attention_decoder/attention_decoder/AttnV_0/read, gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/Sum_grad/Tile)]]
         [[Node: GradientDescent_3/update/_2352 = _Recv[client_terminated=false, recv_device="/job:localhost/replica:0/task:0/cpu:0", send_device="/job:localhost/replica:0/task:0/gpu:0", send_device_incarnation=1, tensor_name="edge_157_GradientDescent_3/update", tensor_type=DT_FLOAT, _device="/job:localhost/replica:0/task:0/cpu:0"]()]]
```

またダメかー

今日はとりあえずここまで。続きは別のエントリに書きます。
