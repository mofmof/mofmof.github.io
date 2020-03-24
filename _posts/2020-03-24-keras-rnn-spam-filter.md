---
layout: blog
title: Kerasを使ってRNN(LSTM)でスパムメッセージの分類をしてみる
category: blog
tags: [Keras, RNN, LSTM, Machine Learning]
summary: 自社のサービスで、テキストを分類したりラベリングしたりということをやりたくなったので、文章タスクが得意らしいRNN(LSTM)で単純な分類問題を解いてみたい。
author: aharada
image: 
---

自社のサービスで、テキストを分類したりラベリングしたりということをやりたくなったので、文章タスクが得意らしいRNN(LSTM)で単純な分類問題を解いてみたい。

スパムメッセージ分類がちょうど良い難易度っぽいので、以下エントリのコードをお借りして動くところまでやってみた。

[https://ohke.hateblo.jp/entry/2019/04/27/154500](https://ohke.hateblo.jp/entry/2019/04/27/154500)

## 環境構築

大体この手のはanaconda使えと書いてあるが、ぼくはなんとなく使わない派(それほど深い理由はない)なので、pypenv+virtualenvを使います。コードの実行は定番のjupyter notebookを使います。

```
$ mkdir keras-rnn-spam-filter
$ cd keras-rnn-spam-filter
```

今回のプロジェクト専用の仮想環境を作る。

```
$ pyenv virtualenv 3.7.3 keras-rnn-spam-filter
$ pyenv local keras-rnn-spam-filter
$ pyenv versions
  system
  3.6.9
  3.7.3
* keras-rnn-spam-filter (set by /Users/atsushiharada/source/keras-rnn-spam-filter/.python-version)
```

あれ notebook がインストールできないやんけ。opensslの問題ぽい。

```
$ pip install notebook                                                                                                                               ✘ 1
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Collecting jupyter
  Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/jupyter/
  Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/jupyter/
  Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/jupyter/
  Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/jupyter/
  Retrying (Retry(total=0, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.")': /simple/jupyter/
  Could not fetch URL https://pypi.org/simple/jupyter/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/jupyter/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
  Could not find a version that satisfies the requirement jupyter (from versions: )
No matching distribution found for jupyter
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
```

ここを参考に再インストールしてみるもダメだった。

[https://qiita.com/drumehiron/items/da79d837ffea3723aafe](https://qiita.com/drumehiron/items/da79d837ffea3723aafe)

```
$ brew unlink openssl
$ brew reinstall openssl
```

reinstall時に表示されるログの通り、パスを通してみたがこれも解決せず。

```
$ echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.zshrc
$ source ~/.zshrc
```

たぶんpythonインストール時にopensslが使用されるっぽいんだけど、それが以前のバージョンと異なっているからエラーになっている？

pythonのインストールからやり直してみる。

```
$ pyenv uninstall 3.7.3
$ pyenv install 3.7.3
```

再び仮想環境を作る。

```
$ pyenv virtualenv 3.7.3 keras-rnn-spam-filter
$ pyenv local keras-rnn-spam-filter
$ pyenv versions
  system
  3.6.9
  3.7.3
  3.7.3/envs/keras-rnn-spam-filter
* keras-rnn-spam-filter (set by /Users/atsushiharada/source/keras-rnn-spam-filter/.python-version)
```

再び notebook をインストール。今回は上手く行ったぞ。

```
$ pip install notebook
```

起動。

```
$ jupyter notebook
```

![Jupyter Notebook](/images/blog/2020-03-24-keras-rnn-spam-filter/jupyter-notebook.png)

## スパム分類器の実装

必要なライブラリをインストールする。

```
$ pip install pandas scikit-learn keras tensorflow
```

こちらからスパム分類のデータセットをダウンロードしてくる。

[https://archive.ics.uci.edu/ml/datasets/sms+spam+collection](https://archive.ics.uci.edu/ml/datasets/sms+spam+collection)

解凍後のSMSSpamCollection にこんな感じのデータが入ってる。

```
ham	Go until jurong point, crazy.. Available only in bugis n great world la e buffet... Cine there got amore wat...
ham	Ok lar... Joking wif u oni...
spam	Free entry in 2 a wkly comp to win FA Cup final tkts 21st May 2005. Text FA to 87121 to receive entry question(std txt rate)T&C's apply 08452810075over18's
ham	U dun say so early hor... U c already then say...
ham	Nah I don't think he goes to usf, he lives around here though
spam	FreeMsg Hey there darling it's been 3 week's now and no word back! I'd like some fun you up for it still? Tb ok! XxX std chgs to send, £1.50 to rcv
ham	Even my brother is not like to speak with me. They treat me like aids patent.
ham	As per your request 'Melle Melle (Oru Minnaminunginte Nurungu Vettam)' has been set as your callertune for all Callers. Press *9 to copy your friends Callertune
spam	WINNER!! As a valued network customer you have been selected to receivea £900 prize reward! To claim call 09061701461. Claim code KL341. Valid 12 hours only.
spam	Had your mobile 11 months or more? U R entitled to Update to the latest colour mobiles with camera for Free! Call The Mobile Update Co FREE on 08002986030

...

```

データセットをプロジェクト配下に移動。

```
$ mv ~/Downloads/smsspamcollection/SMSSpamCollection .
```

参考にしたエントリのコードをそのままコピペして動かしてみる。

```
from sklearn.model_selection import train_test_split

X_train, X_test, Y_train, Y_test = train_test_split(
    dataset_df[['text']], dataset_df[['category']], 
    test_size=0.2, random_state=0
)

print(X_train.shape, X_test.shape, Y_train.shape, Y_test.shape)
```

```
from keras.preprocessing.text import Tokenizer
from keras.preprocessing.sequence import pad_sequences

max_len = 100

tokenizer = Tokenizer()
tokenizer.fit_on_texts(X_train['text'])
x_train = tokenizer.texts_to_sequences(X_train['text'])
x_test = tokenizer.texts_to_sequences(X_test['text'])

for text, vector in zip(X_train['text'].head(3), x_train[0:3]):
    print(text)
    print(vector)
```

```
x_train = pad_sequences(x_train, maxlen=max_len)
x_test = pad_sequences(x_test, maxlen=max_len)

print(x_train[0])
```

```
rom keras.models import Sequential
from keras.layers import LSTM, Dense, Embedding

vocabulary_size = len(tokenizer.word_index) + 1 

model = Sequential()

model.add(Embedding(input_dim=vocabulary_size, output_dim=32))
model.add(LSTM(16, return_sequences=False))
model.add(Dense(1, activation='sigmoid'))

model.compile(loss='binary_crossentropy', optimizer='adam', metrics=['accuracy'])

model.summary()
```

```
y_train = Y_train['category'].values
y_test = Y_test['category'].values

history = model.fit(
    x_train, y_train, batch_size=32, epochs=10,
    validation_data=(x_test, y_test)
)
```

学習出来た。

![学習結果](/images/blog/2020-03-24-keras-rnn-spam-filter/train-result.png)

## スパムっぽい文章をいれて分類させてみる

データセットのスパムメッセージ文面を見ながら作ってみたが、自分で英語のスパムメッセージ作るってなかなか難しいのな。。。

こんなGoogle翻訳を使ってメッセージ作ってみた。我ながらしょうもない文面を書いてしまった自覚があるので、日本語にはしない方がいいぞ。

Are you single? Use this site to find women in their 20s who want to have sex right away! Pay $ 100 Now and Get Free! 08717890890

データセットを見ていると、spamラベルがついているメッセージには怪しげなURLがついてることが多いので、怪しげなURL(当然いい加減なURL)を付け加えた版。

Are you single? Use this site to find women in their 20s who want to have sex right away! Pay $ 100 Now and Get FREE! 08717890890 http://porn.sex/xxx.asp?o=13543

あとは普通のお花見に誘うメッセージ。

Hello. It is nice weather. It's time to see the cherry blossoms, so how about cherry blossom viewing together?

```
texts = [
    "Are you single? Use this site to find women in their 20s who want to have sex right away! Pay $ 100 Now and Get FREE! 08717890890",
    "Are you single? Use this site to find women in their 20s who want to have sex right away! Pay $ 100 Now and Get FREE! 08717890890 http://porn.sex/xxx.asp?o=13543",
    "Hello. It is nice weather. It's time to see the cherry blossoms, so how about cherry blossom viewing together"
]
x = tokenizer.texts_to_sequences(texts)
x = pad_sequences(x, maxlen=max_len)

y_pred = model.predict_classes(x)
print(y_pred)
```

結果

![スパム判定結果](/images/blog/2020-03-24-keras-rnn-spam-filter/predict.png)

URLがついていない方はスパム判定してくれませんでした。学習モデルが悪いというより、ぼくのスパムメッセージ作成スキルが足りないせいっぽい。お花見に誘うメッセージはあやしいところはないので当然非スパムと判定された。

RNN(LSTM)でシンプルな分類問題を解くやり方は分かったので、次何しよっかな。