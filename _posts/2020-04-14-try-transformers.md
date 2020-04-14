---
layout: blog
title: huggingfaceのtransformersでNER(named entity recognition)を試してみる
category: blog
tags: [huggingface, Machine Learning, NER, 固有表現抽出]
summary: 最近、BERTで学習済みの日本語言語モデルが使えるようになったらしいhugginfaceのtransformers。自然言語処理界隈では主流になっているらしい。
author: aharada
image: 
---

最近、BERTで学習済みの日本語言語モデルが使えるようになったらしいhugginfaceのtransformers。自然言語処理界隈では主流になっているらしい。普通にBERTを自前で実装すると非常に大変だけど、バックエンドにtensolflowやpytorchに、このtransformersを使用すると、とても簡単に実装出来ると聞いたのでやってみる。

公式のリポジトリ。

[https://github.com/huggingface/transformers](https://github.com/huggingface/transformers)

READMEに記載されている通りにexampleの中のnerを試してみる。

[https://github.com/huggingface/transformers/blob/master/examples/README.md](https://github.com/huggingface/transformers/blob/master/examples/README.md)

環境を構築する。

```
$ git clone https://github.com/huggingface/transformers -b v2.8.0
$ pip install .
$ pip install -r ./examples/requirements.txt
$ pip install torch
```

masterではなくv2.8.0を使用しているのは、masterだと下記エラーが出て動かなかったため。issueに上がってて、おそらく公式のexampleが修正に追いついていないのかな？

```
File "/transformers/examples/ner/utils_ner.py", line 123, in convert_examples_to_features special_tokens_count = tokenizer.num_added_tokens() 
AttributeError: 'BertTokenizer' object has no attribute 'num_added_tokens'
```

run.shに実行権限をつける。

```
$ cd examples/ner
$ chmod u+x run.sh
```

普通のCPUで動いてるMac Bookとかで実行すると数時間かかる。サクッと動かして試してみたいだけなので、run.shを編集して学習させるデータを減らしておく。

run.sh

```
curl -L 'https://sites.google.com/site/germeval2014ner/data/NER-de-train.tsv?attredirects=0&d=1' \
| grep -v "^#" | cut -f 2,3 | tr '\t' ' ' > train.txt.tmp
curl -L 'https://sites.google.com/site/germeval2014ner/data/NER-de-dev.tsv?attredirects=0&d=1' \
| grep -v "^#" | cut -f 2,3 | tr '\t' ' ' > dev.txt.tmp
curl -L 'https://sites.google.com/site/germeval2014ner/data/NER-de-test.tsv?attredirects=0&d=1' \
| grep -v "^#" | cut -f 2,3 | tr '\t' ' ' > test.txt.tmp
# macだとwgetが入っていないのでcurlに書き換えた
curl -OL "https://raw.githubusercontent.com/stefan-it/fine-tuned-berts-seq/master/scripts/preprocess.py"

export MAX_LENGTH=128
export BERT_MODEL=bert-base-multilingual-cased
python3 preprocess.py train.txt.tmp $BERT_MODEL $MAX_LENGTH > train.txt
python3 preprocess.py dev.txt.tmp $BERT_MODEL $MAX_LENGTH > dev.txt
python3 preprocess.py test.txt.tmp $BERT_MODEL $MAX_LENGTH > test.txt
cat train.txt dev.txt test.txt | cut -d " " -f 2 | grep -v "^$"| sort | uniq > labels.txt

# 学習に時間がかかるためデータ数を減らす
sed -i -e '10001,$d' train.txt
sed -i -e '2501,$d' test.txt

export OUTPUT_DIR=germeval-model
export BATCH_SIZE=32
export NUM_EPOCHS=3
export SAVE_STEPS=750
export SEED=1

python3 run_ner.py --data_dir ./ \
--model_type bert \
--labels ./labels.txt \
--model_name_or_path $BERT_MODEL \
--output_dir $OUTPUT_DIR \
--max_seq_length  $MAX_LENGTH \
--num_train_epochs $NUM_EPOCHS \
--per_gpu_train_batch_size $BATCH_SIZE \
--save_steps $SAVE_STEPS \
--seed $SEED \
--do_train \
--do_eval \
--do_predict
```

train.txtの中身をちょっと見てみると、単語ごとにB-PER(Begin-人名)などのラベルが振られているのが分かる。ファイル名にdeが含まれているのでおそらくドイツ語だろう。

```
Schartau B-PER
sagte O
dem O
" O
Tagesspiegel B-ORG
" O
vom O
Freitag O
, O
Fischer B-PER
sei O

...
```

実行すると数分で完了する。

```
$ ./run.sh

...

04/14/2020 15:55:07 - INFO - __main__ -   ***** Running evaluation  *****
04/14/2020 15:55:07 - INFO - __main__ -     Num examples = 122
04/14/2020 15:55:07 - INFO - __main__ -     Batch size = 8
Evaluating: 100%|████████████████████████████████████████| 16/16 [00:15<00:00,  1.00it/s]
04/14/2020 15:55:23 - INFO - __main__ -   ***** Eval results  *****
04/14/2020 15:55:23 - INFO - __main__ -     f1 = 0.339622641509434
04/14/2020 15:55:23 - INFO - __main__ -     loss = 0.1777263912372291
04/14/2020 15:55:23 - INFO - __main__ -     precision = 0.3016759776536313
04/14/2020 15:55:23 - INFO - __main__ -     recall = 0.38848920863309355
```

test_predictions.txtにpredictの結果が入るので、text.txtと比較してみる。

test_predictions.txt 

```
1951 O
bis O
1953 O
wurde O
der O
nördliche O
Teil O
als O
Jugendburg O
des O
Kolpingwerkes B-LOC
gebaut O
. O

Da O
Muck B-PER
das O
Kriegsschreiben O
nicht O
überbracht O
hat O
, O
wird O
er O
als O
Retter O
des O
Landes O
ausgezeichnet O
und O
soll O
zum O
Schatzmeister O
ernannt O
werden O
. O

Mit O
1. O
Jänner O
2007 O
wurde O
Robert B-PER
Schörgenhofer B-PER

...
```

test.txt

```
1951 O
bis O
1953 O
wurde O
der O
nördliche O
Teil O
als O
Jugendburg O
des O
Kolpingwerkes B-OTH
gebaut O
. O

Da O
Muck B-PER
das O
Kriegsschreiben O
nicht O
überbracht O
hat O
, O
wird O
er O
als O
Retter O
des O
Landes O
ausgezeichnet O
und O
soll O
zum O
Schatzmeister O
ernannt O
werden O
. O

Mit O
1. O
Jänner O
2007 O
wurde O
Robert B-PER
Schörgenhofer I-PER

...
```

パッと見でも正確にラベリング出来ていないことが分かる。
普通のCRFで実装したときは、この程度のデータ量でもうまく動いたけど、やっぱり教師データ減らし過ぎたのが良くなかったかな。
