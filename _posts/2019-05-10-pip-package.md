---
layout: blog
title: pipでGitHub経由でインストール出来る自作パッケージを作る
category: blog
tags: [機械学習,machine learning,pip,python]
summary: 毎回手作業で作るのめんどいから、とりあえずスクリプトでサクッと取ってこれる状態にしたい。そこでpip installしたらデータをロードする関数をimport出来るようなパッケージを作りたい。
author: aharada
# image:
---

機械学習や自然言語処理でごにょごにょ試してみるときには、大体の場合データが必要で、どこからか取得してくるか作る必要がある。日本語のデータはあまり多くはなくて、とりあえず試しに動かす程度でいいやってときは自分で適当なデータ作ったりするんだけど、毎回手作業で作るのめんどいから、とりあえずスクリプトでサクッと取ってこれる状態にしたい。そこでpip installしたらデータをロードする関数をimport出来るようなパッケージを作りたい。

ひとまず今回は自作のpip用パッケージを試してみる。自分しか使わないのでPyPIには登録せず、GitHubのURLを指定してpip installするパッケージにする。

参考

[Pythonで作成したライブラリを、PyPIに公開/アップロードする - Qiita](https://qiita.com/icoxfog417/items/edba14600323df6bf5e0)

公式かな？パッケージのテンプレがあるのでこれをベースに進める。

[GitHub - pypa/sampleproject: A sample project that exists for PyPUG's "Tutorial on Packaging and Distributing Projects"](https://github.com/pypa/sampleproject)

## 作ってみる

`say()`と実行したら`hoo`と標準出力されるパッケージを作る。

まずは上記のテンプレをcloneするなりダウンロードするなりして手元に保存。

不要そうなファイルやディレクトリを適当に削除。使わなそうだったので`data`ディレクトリを削除してみた。ルート直下の`sample`ディレクトリを`sample-dt`ディレクトリに更新する。

gitでコミットして`sample-dt`というパッケージ名でリポジトリを作成(パッケージ名は好きな名前で)。

```
$ git add .
$ git commit
$ git create sample-dt
$ git push
```

pipでインストールしてみる。`data`ディレクトリを消しちゃったけど`setup.py`を修正してなかったためエラーぽい。

```
$ pip install git+https://github.com/<your_github_id>/sample-dt

error: can't copy 'data/data_file': doesn't exist or not a regular file
```

`setup.py`の`data_files`部分をコメントアウトして再びgitにcommitしてpushする。

再びpip installすると成功。

`say()`を実行してみる。

```
$ python

>>> from sample-dt.say import hoo
  File "<stdin>", line 1
    from sample-dt.say import hoo
           ^
SyntaxError: invalid syntax
```

importする際にハイフンがダメっぽい。ディレクトリ名を変更する。変更するディレクトリは、ルート自身のディレクトリ名ではなく、ルート直下の`sample-dt`ディレクトリ。これを`sampledt`に変更。

```
$ python
>>> from sampledt.say import hoo
>>> hoo()
hoo
```

出来たー！