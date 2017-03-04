---
layout: blog
title: TensorFlowのseq2seqで英仏語翻訳のチュートリアルを試す②
category: blog
tags: [機械学習,machine learning,TensorFlow,seq2seq,RNN]
summary: 前回でTensorFlowのseq2seqのチュートリアルの英語フランス語翻訳を実装してみたのですが、
author: aharada
---

前回でTensorFlowのseq2seqのチュートリアルの英語フランス語翻訳を実装してみたのですが、あちらこちらにハマりポイントがあり、未だにうまくいってません。続きを書いていこうと思います！

学習を実行したら以下のようなエラーが出てしまったのでこれの対処から。

```
$ python translate.py --data_dir data --train_dir train

略...

ResourceExhaustedError (see above for traceback): OOM when allocating tensor with shape[64,40,1,1024]
         [[Node: gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/mul_grad/mul_1 = Mul[T=DT_FLOAT, _device="/job:localhost/replica:0/task:0/gpu:0"](embedding_attention_seq2seq/embedding_attention_decoder/attention_decoder/AttnV_0/read, gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/Sum_grad/Tile)]]
         [[Node: GradientDescent_3/update/_2352 = _Recv[client_terminated=false, recv_device="/job:localhost/replica:0/task:0/cpu:0", send_device="/job:localhost/replica:0/task:0/gpu:0", send_device_incarnation=1, tensor_name="edge_157_GradientDescent_3/update", tensor_type=DT_FLOAT, _device="/job:localhost/replica:0/task:0/cpu:0"]()]]
```

どうやら単にGPUのメモリが足りないということらしい。トレーニングセットのデータ数を減らせば良いのかと思ったので`max_train_data_size`を指定してみるも、やっぱりダメ。

```
$ python translate.py --data_dir data --train_dir train --size 400 --num_layers 1 --batch_size 5 --max_train_data_size 100

略...

ResourceExhaustedError (see above for traceback): OOM when allocating tensor with shape[64,40,1,1024]
         [[Node: gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/Sum_1_grad/Tile = Tile[T=DT_FLOAT, Tmultiples=DT_INT32, _device="/job:localhost/replica:0/task:0/gpu:0"](gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/Sum_1_grad/Reshape, gradients_3/model_with_buckets/embedding_attention_seq2seq_3/embedding_attention_decoder/attention_decoder/Attention_0_32/Sum_1_grad/floordiv)]]
         [[Node: GradientDescent_3/update/_2352 = _Recv[client_terminated=false, recv_device="/job:localhost/replica:0/task:0/cpu:0", send_device="/job:localhost/replica:0/task:0/gpu:0", send_device_incarnation=1, tensor_name="edge_157_GradientDescent_3/update", tensor_type=DT_FLOAT, _device="/job:localhost/replica:0/task:0/cpu:0"]()]]
```

GPUの状態をwatchしてみる。

```
$ watch nvidia-smi

Every 2.0s: nvidia-smi                                           Sat Mar  4 02:20:22 2017

Sat Mar  4 02:20:22 2017
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 367.57                 Driver Version: 367.57                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GRID K520           Off  | 0000:00:03.0     Off |                  N/A |
| N/A   40C    P0    71W / 125W |   3774MiB /  4036MiB |     81%      Default |
+-------------------------------+----------------------+----------------------+
|   1  GRID K520           Off  | 0000:00:04.0     Off |                  N/A |
| N/A   31C    P8    17W / 125W |   3737MiB /  4036MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  GRID K520           Off  | 0000:00:05.0     Off |                  N/A |
| N/A   27C    P8    17W / 125W |   3737MiB /  4036MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   3  GRID K520           Off  | 0000:00:06.0     Off |                  N/A |
| N/A   27C    P8    17W / 125W |   3737MiB /  4036MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|    0      2796    C   python                                        3772MiB |
|    1      2796    C   python                                        3735MiB |
|    2      2796    C   python                                        3735MiB |
|    3      2796    C   python                                        3735MiB |
+-----------------------------------------------------------------------------+
```

ふーむ。Memory-Usageをみるとたしかに使い切っているように見える。

次に幾つかパラメータを指定して流してみる。これならいけるようだ。試しに`num_layers`を3で流したら落ちたので、この辺が影響してるっぽい。

```
$ python translate.py --data_dir data --train_dir train --size 400 --num_layers 1 --batch_size 5

global step 39400 learning rate 0.4173 step-time 0.23 perplexity 1.86
  eval: bucket 0 perplexity 1496.77
  eval: bucket 1 perplexity 1331.72
  eval: bucket 2 perplexity 804.83
  eval: bucket 3 perplexity 4232.93
global step 39600 learning rate 0.4173 step-time 0.23 perplexity 1.86
  eval: bucket 0 perplexity 3453.77
  eval: bucket 1 perplexity 507.74
  eval: bucket 2 perplexity 1483.24
  eval: bucket 3 perplexity 1091.38
```

perplexityが1に近づいてきたので、実際に翻訳して確かめてみる。

```
$ python translate.py --data_dir data --train_dir train --size 400 --num_layers 1 --batch_size 5 --decode

> Who is the president of the United States?
La question se sont une importance aux Etats-Unis et salles ?
```

google翻訳で英語に戻すと、

```
The question are of importance in the United States and rooms?
```

ぜんぜんちげえええええ！

## 次回に続く

どうしたら良いかわからないので、もうちょっと公式チュートリアルをちゃんと読んでみたら、「デフォルトのパラメータだとGPUのパワーがめっちゃ必要だから、そうじゃない環境ではこんな感じでやってみてね」的なことが買いてあった。とりあえず、そこらへんに買いてあったパラメータでまた流しておいて、しばらく放置してみる。

```
$ python translate.py --data_dir data --train_dir train --size=256 --num_layers=2 --steps_per_checkpoint=50

略...

global step 3450 learning rate 0.4614 step-time 0.40 perplexity 50.41
  eval: bucket 0 perplexity 659.05
  eval: bucket 1 perplexity 227.56
  eval: bucket 2 perplexity 171.14
  eval: bucket 3 perplexity 174.53
global step 3500 learning rate 0.4614 step-time 0.38 perplexity 44.19
  eval: bucket 0 perplexity 268.53
  eval: bucket 1 perplexity 148.96
  eval: bucket 2 perplexity 173.45
  eval: bucket 3 perplexity 182.31
global step 3550 learning rate 0.4614 step-time 0.37 perplexity 42.18
  eval: bucket 0 perplexity 288.56
  eval: bucket 1 perplexity 139.56
  eval: bucket 2 perplexity 151.27
  eval: bucket 3 perplexity 154.05
```

チュートリアルの説明をちゃんと読むと、`bucket`の方の`perplexity`は`development set`に対しての数値だよって書いてある。`development set`が何のことを言っているのかわからないけど、トレーニングセットってことだったら、こっちを収束するのをみなきゃいけないのもかも知れない。

次回に続く。
