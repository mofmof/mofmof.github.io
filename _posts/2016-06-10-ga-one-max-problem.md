---
layout: blog
title: 遺伝的アルゴリズムに入門するときに参考になったスライドとOneMax問題の実装
category: blog
tags: [python,machine learning,機械学習,遺伝的アルゴリズム,Deap,genetic algorithm,OneMax問題]  
summary: 弊社の新規事業でとある自動生成機能を実装したいのですが、どういうアルゴリズムにすべきか検討中で
author: aharada
---

弊社の新規事業でとある自動生成機能を実装したいのですが、どういうアルゴリズムにすべきか検討中で、選択肢として遺伝的アルゴリズムが上がっていたので入門してみることにしました。

遺伝的アルゴリズムはニコニコ動画で見たことがあるのでその存在は知っている程度だったのですが、そのレベルからキャッチアップするために非常に参考になったスライドと実装を載せていきます。

<iframe width="312" height="176" src="http://ext.nicovideo.jp/thumb/sm16597051" scrolling="no" style="border:solid 1px #CCC;" frameborder="0"><a href="http://www.nicovideo.jp/watch/sm16597051">【ニコニコ動画】遺伝的アルゴリズムと物理エンジンで二足歩行を学習させた</a></iframe>

## 概念と全貌の理解

まずはアルゴリズムの概念と全貌を理解したいのでインプットしていきます。

こちらはサックリ全貌が理解出来ます。わかりやすいです。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/iAD0wpYQnZbB2" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/sandinist/ss-1617965" title="できる！遺伝的アルゴリズム" target="_blank">できる！遺伝的アルゴリズム</a> </strong> from <strong><a href="//www.slideshare.net/sandinist" target="_blank">Maehana Tsuyoshi</a></strong> </div>

こちらも全貌を説明しつつ、選択や交叉、変異のパターンについて少し詳しく解説しています。

<iframe src="//www.slideshare.net/slideshow/embed_code/key/u93ud6zmXMFYvY" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/kzokm/genetic-algorithm-41617242" title="遺伝的アルゴリズム（Genetic Algorithm）を始めよう！" target="_blank">遺伝的アルゴリズム（Genetic Algorithm）を始めよう！</a> </strong> from <strong><a href="//www.slideshare.net/kzokm" target="_blank">Kazuhide Okamura</a></strong> </div>

生物界の進化の過程を模倣したアルゴリズムなので直感的に理解しやすいと思います。細かいところはともかくたぶん一回読めはなんとなくわかるはず。

## OneMax問題
Pythonの遺伝的アルゴリズムのライブラリ[Deap](https://github.com/deap/deap)を使って実装していきます。

OneMax問題という非常に簡単な問題を遺伝的アルゴリズムで解いていくわけなのですが、OneMax問題というのは、0,1でランダムに初期化されたビット配列を全て1にするという非常にシンプルな問題です。

例えば初期化時の個体が、

```
[1,0,1,1,0,0,0,1,0]
```

だったものを遺伝的アルゴリズムで進化させることで

```
[1,1,1,1,1,1,1,1,1]
```

こういう個体に近づけていくということです。

## 早速実装してみよう

こちらのエントリの通り、下記の要領で遺伝的アルゴリズムを走らせていきます。

[http://qiita.com/simanezumi1989/items/10cfe1e8a23cd9d4c7b1](http://qiita.com/simanezumi1989/items/10cfe1e8a23cd9d4c7b1)


1. 初期設定
2. 評価
3. 選択
4. 交叉
5. 突然変異
6. 2〜5を繰り返す

実装はこちらのエントリをすさまじく参考にさせていただきました。ちょっと理解を深めるためにコメントを追記した程度。

[http://qiita.com/neka-nat@github/items/0cb8955bd85027d58c8e](http://qiita.com/neka-nat@github/items/0cb8955bd85027d58c8e)

main.py

```
# -*- coding: utf-8 -*-
import random

from deap import base
from deap import creator
from deap import tools

# 適合度クラスを作成
creator.create("FitnessMax", base.Fitness, weights=(1.0,))
creator.create("Individual", list, fitness=creator.FitnessMax)

toolbox = base.Toolbox()
# Attributeを生成をする関数を定義(0,1のランダムを選ぶ)
toolbox.register("attr_bool", random.randint, 0, 1)
# 個体を生成する関数を定義(Individualクラスでattr_boolの値を100個持つ)
toolbox.register("individual", tools.initRepeat, creator.Individual, toolbox.attr_bool, 100)
# 集団を生成する関数を定義(個体を持つlist)
toolbox.register("population", tools.initRepeat, list, toolbox.individual)

# 評価関数
def evalOneMax(individual):
    return sum(individual),

# 評価関数を登録
toolbox.register("evaluate", evalOneMax)
# 交叉関数を定義(二点交叉)
toolbox.register("mate", tools.cxTwoPoint)
# 変異関数を定義(ビット反転、変異隔離が5%ということ?)
toolbox.register("mutate", tools.mutFlipBit, indpb=0.05)
# 選択関数を定義(トーナメント選択、tournsizeはトーナメントの数？)
toolbox.register("select", tools.selTournament, tournsize=3)


if __name__ == '__main__':
    # 初期集団を生成する
    pop = toolbox.population(n=300)
    CXPB, MUTPB, NGEN = 0.5, 0.2, 40 # 交差確率、突然変異確率、進化計算のループ回数

    print("進化開始")

    # 初期集団の個体を評価する
    fitnesses = list(map(toolbox.evaluate, pop))
    for ind, fit in zip(pop, fitnesses):  # zipは複数変数の同時ループ
        # 適合性をセットする
        ind.fitness.values = fit

    print("  %i の個体を評価" % len(pop))

     # 進化計算開始
    for g in range(NGEN):
        print("-- %i 世代 --" % g)

        ##############
        # 選択
        ##############
         # 次世代の個体群を選択
        offspring = toolbox.select(pop, len(pop))
        # 個体群のクローンを生成
        offspring = list(map(toolbox.clone, offspring))

        # 選択した個体群に交差と突然変異を適応する

        ##############
        # 交叉
        ##############
        # 偶数番目と奇数番目の個体を取り出して交差
        for child1, child2 in zip(offspring[::2], offspring[1::2]):
            if random.random() < CXPB:
                toolbox.mate(child1, child2)
                # 交叉された個体の適合度を削除する
                del child1.fitness.values
                del child2.fitness.values

        ##############
        # 変異
        ##############
        for mutant in offspring:
            if random.random() < MUTPB:
                toolbox.mutate(mutant)
                del mutant.fitness.values

        # 適合度が計算されていない個体を集めて適合度を計算
        invalid_ind = [ind for ind in offspring if not ind.fitness.valid]
        fitnesses = map(toolbox.evaluate, invalid_ind)
        for ind, fit in zip(invalid_ind, fitnesses):
            ind.fitness.values = fit

        print("  %i の個体を評価" % len(invalid_ind))

        # 次世代群をoffspringにする
        pop[:] = offspring

        # すべての個体の適合度を配列にする
        fits = [ind.fitness.values[0] for ind in pop]

        length = len(pop)
        mean = sum(fits) / length
        sum2 = sum(x*x for x in fits)
        std = abs(sum2 / length - mean**2)**0.5

        print("  Min %s" % min(fits))
        print("  Max %s" % max(fits))
        print("  Avg %s" % mean)
        print("  Std %s" % std)

    print("-- 進化終了 --")

    best_ind = tools.selBest(pop, 1)[0]
    print("最も優れていた個体: %s, %s" % (best_ind, best_ind.fitness.values))
```

deapをインストール

```
$ pip install deap
```

実行します。

```
$ python main.py
進化開始
  300 の個体を評価
-- 0 世代 --
  173 の個体を評価
  Min 42.0
  Max 64.0
  Avg 53.6466666667
  Std 4.09900258871
-- 1 世代 --
  174 の個体を評価
  Min 43.0
  Max 70.0
  Avg 57.33
  Std 3.62046497935
-- 2 世代 --
  170 の個体を評価
  Min 51.0
  Max 72.0
  Avg 60.0666666667
  Std 3.641733409
-- 3 世代 --
  201 の個体を評価
  Min 51.0
  Max 72.0
  Avg 63.3066666667
  Std 3.63766714011

略

-- 37 世代 --
  178 の個体を評価
  Min 89.0
  Max 100.0
  Avg 98.9333333333
  Std 2.24251842554
-- 38 世代 --
  161 の個体を評価
  Min 89.0
  Max 100.0
  Avg 98.8933333333
  Std 2.3766830294
-- 39 世代 --
  186 の個体を評価
  Min 90.0
  Max 100.0
  Avg 98.9933333333
  Std 2.19392089395
-- 進化終了 --
最も優れていた個体: [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1], (100.0,)
```

ちゃんと収束して全てのbitが1の個体が生まれました！！
