---
layout: blog
title: spaCy + GiNZAの固有表現抽出(CRF)で、電話番号とFAX番号を識別出来るか試す
category: blog
tags: [spaCy, GiNZA, NER, 固有表現抽出, CRF]
summary: 前回、spaCyとGiNZAを使って固有表現抽出するところまでやったので、今回は電話番号とFAX番号を固有表現抽出で識別出来るか試してみます。
author: aharada
image: 
---

前回、spaCyとGiNZAを使って固有表現抽出するところまでやったので、今回は電話番号とFAX番号を固有表現抽出で識別出来るか試してみます。

[spaCy + GiNZAを使って固有表現抽出とカスタムモデルの学習をしてみる](/blog/spacy-ner.html)

CRF(Conditional Random Fields)の仕組みを解説したいところだけど解説出来るほどの知識量が足りないので、実際に動かして傾向を把握することにします。

```
$ jupyter notebook
```

前回のコードを引き続き使う。`TRAIN_DATA`に電話番号とFAX番号が混在した教師データを入れる。

```
TRAIN_DATA = [
	('お電話はこちらから: 03-3333-4444、ご注文はFAXからでも行えます。FAX:03-3333-4445 ご注文お待ちしております。', {'entities': [(11, 23, 'TEL'), (44, 56, 'FAX')]}), 
	('お申し込みFAX番号　03-6666-6666', {'entities': [(11, 23, 'FAX')]}),
	('総務課 電話番号：03-8888-7777 FAX番号：03-8888-7778', {'entities': [(9, 21, 'TEL'), (28, 40, 'FAX')]}),
	('電話番号：03-4444-1111（代表）　FAX番号：03-4444-1112', {'entities': [(5, 17, 'TEL'), (28, 40, 'FAX')]}),
	('ＦＡＸ番号が変更になりましたので、お知らせいたします。新ＦＡＸ番号：03-2222-5555', {'entities': [(34, 46, 'FAX')]}),
	('TEL：03-5555-1111（代）　FAX：03-5555-1112', {'entities': [(4, 16, 'TEL'), (24, 36, 'FAX')]})
]

def train_spacy(data,iterations):
    TRAIN_DATA = data
#     nlp = spacy.blank('en')  # create blank Language class
    nlp = spacy.blank('ja')  # create blank Language class
    # create the built-in pipeline components and add them to the pipeline
    # nlp.create_pipe works for built-ins that are registered with spaCy
    if 'ner' not in nlp.pipe_names:
        ner = nlp.create_pipe('ner')
        nlp.add_pipe(ner, last=True)
       

    # add labels
    for _, annotations in TRAIN_DATA:
         for ent in annotations.get('entities'):
            ner.add_label(ent[2])

    # get names of other pipes to disable them during training
    other_pipes = [pipe for pipe in nlp.pipe_names if pipe != 'ner']
    with nlp.disable_pipes(*other_pipes):  # only train NER
        optimizer = nlp.begin_training()
        for itn in range(iterations):
            print("Statring iteration " + str(itn))
            random.shuffle(TRAIN_DATA)
            losses = {}
            for text, annotations in TRAIN_DATA:
                nlp.update(
                    [text],  # batch of texts
                    [annotations],  # batch of annotations
                    drop=0.2,  # dropout - make it harder to memorise data
                    sgd=optimizer,  # callable to update weights
                    losses=losses)
            print(losses)
    return nlp


prdnlp = train_spacy(TRAIN_DATA, 40)

# Save our trained Model
modelfile = input("Enter your Model Name: ")
prdnlp.to_disk(modelfile)

#Test your text
test_text = input("Enter your testing text: ")
doc = prdnlp(test_text)
for ent in doc.ents:
    print(ent.text, ent.start_char, ent.end_char, ent.label_)
```

結果。教師データに入っているテキストを入れてみたら正しく抽出出来た(そりゃそうだ)。

```
Statring iteration 0
{'ner': 92.61980903148651}
Statring iteration 1
{'ner': 32.28841861433466}
Statring iteration 2
{'ner': 15.22787763429585}
Statring iteration 3
{'ner': 54.940556606326595}
Statring iteration 4
{'ner': 38.99507546105323}
Statring iteration 5
{'ner': 19.006579443288047}
Statring iteration 6
{'ner': 73.64191366296832}
Statring iteration 7
{'ner': 75.02919562291186}
Statring iteration 8
{'ner': 48.15495109953948}
Statring iteration 9
{'ner': 56.59243585390277}
Statring iteration 10
{'ner': 37.516817034307905}
Statring iteration 11
{'ner': 18.544477883659965}
Statring iteration 12
{'ner': 17.486992379765265}
Statring iteration 13
{'ner': 31.31742142809455}
Statring iteration 14
{'ner': 25.753152605662695}
Statring iteration 15
{'ner': 27.189502558613892}
Statring iteration 16
{'ner': 22.540683782826534}
Statring iteration 17
{'ner': 16.671177273729015}
Statring iteration 18
{'ner': 10.10902092439708}
Statring iteration 19
{'ner': 5.6178949416471955}
Statring iteration 20
{'ner': 0.8204771258328467}
Statring iteration 21
{'ner': 0.14647625923280155}
Statring iteration 22
{'ner': 0.5714098855746599}
Statring iteration 23
{'ner': 0.0012918024679473444}
Statring iteration 24
{'ner': 0.0007140604429003128}
Statring iteration 25
{'ner': 0.08589177494769792}
Statring iteration 26
{'ner': 4.0084468914959356e-05}
Statring iteration 27
{'ner': 0.0006815655828836105}
Statring iteration 28
{'ner': 1.001842008281531e-05}
Statring iteration 29
{'ner': 1.441425372649282e-06}
Statring iteration 30
{'ner': 3.4397922933582945e-06}
Statring iteration 31
{'ner': 6.984775009784372e-06}
Statring iteration 32
{'ner': 2.755347397455804e-06}
Statring iteration 33
{'ner': 1.8889010991780776e-07}
Statring iteration 34
{'ner': 7.340185455034415e-07}
Statring iteration 35
{'ner': 1.3201836638998097e-06}
Statring iteration 36
{'ner': 1.6599996854295993e-05}
Statring iteration 37
{'ner': 5.170394290007117e-07}
Statring iteration 38
{'ner': 1.5867757308576705e-06}
Statring iteration 39
{'ner': 8.932150472315945e-07}
Enter your Model Name: telfax.mdl
Enter your testing text: お電話はこちらから: 03-3333-4444、ご注文はFAXからでも行えます。FAX:03-3333-4445 ご注文お待ちしております。
03-3333-4444 11 23 TEL
03-3333-4445 44 56 FAX
```

教師データにないテキストを入れてみると、全てFAX番号になってしまった。

```
Enter your testing text: 郵便番号 000-0000 住所 大阪府大阪市住之江区南港南1-11-11 電話番号 06-6666-6666 FAX番号 06-6666-6666
000-0000 5 13 FAX
1-11-11 30 37 FAX
06-6666-6666 43 55 FAX
06-6666-6666 62 74 FAX
```

他のテキストも試すが、全てFAX番号になってしまった。

```
Enter your testing text: 〒111-1111 神奈川県平塚市浅間町1番1号 別館1階 直通電話：03-2222-3333 FAX番号：03-2222-3331
111-1111 1 9 FAX
03-2222-3333 35 47 FAX
03-2222-3331 54 66 FAX
```

電話番号とFAX番号の位置を入れ替えて試したら、やはり全てFAX番号になってしまった。

```
Enter your testing text: ご注文はFAXからでも行えます。FAX:03-3333-4445 ご注文お待ちしております。お電話はこちらから: 03-3333-4444
03-3333-4445 20 32 FAX
03-3333-4444 57 69 FAX
```

データが少なすぎるのでなんとも言えないが、ちょっと過学習しているような感じもある。TELが先、FAXが後っていう文章でなければ正しくラベリング出来なかった。前後のラベルよりも出現位置もしくは、出現頻度が強く反映された結果になってしまった。

## 電話番号とFAX番号を分けて教師データにしてみる

文章内に電話番号とFAX番号を混在させていると、番号の前後のラベルがどちらも近くにいることが多く、付近のラベルが特徴量としてあまり反映されないのではないか？という仮説を立て、電話番号とFAX番号を分けて教師データにしてみることにより、付近のラベル(「お電話」とか)の影響が強くなるか試してみる。

```
import spacy
import random


TRAIN_DATA = [
	('お電話はこちらから: 03-3333-4444', {'entities': [(11, 23, 'TEL')]}), 
	('ご注文はFAXからでも行えます。FAX:03-3333-4445', {'entities': [(20, 32, 'FAX')]}),
	('ご注文お待ちしております。', {'entities': []}),
	('お申し込みFAX番号　03-6666-6666', {'entities': [(11, 23, 'FAX')]}),
	('総務課 電話番号：03-8888-7777', {'entities': [(9, 21, 'TEL')]}),
	('FAX番号：03-8888-7778', {'entities': [(6, 18, 'TEL')]}),
	('電話番号：03-4444-1111（代表）', {'entities': [(5, 17, 'TEL')]}),
	('FAX番号：03-4444-1112', {'entities': [(6, 18, 'FAX')]}),
	('ＦＡＸ番号が変更になりましたので、お知らせいたします。', {'entities': []}),
	('新ＦＡＸ番号：03-2222-5555', {'entities': [(7, 19, 'FAX')]}),
	('TEL：03-5555-1111（代）', {'entities': [(4, 16, 'TEL')]}),
	('FAX：03-5555-1112', {'entities': [(4, 16, 'FAX')]})
]


def train_spacy(data,iterations):
    TRAIN_DATA = data
#     nlp = spacy.blank('en')  # create blank Language class
    nlp = spacy.blank('ja')  # create blank Language class
    # create the built-in pipeline components and add them to the pipeline
    # nlp.create_pipe works for built-ins that are registered with spaCy
    if 'ner' not in nlp.pipe_names:
        ner = nlp.create_pipe('ner')
        nlp.add_pipe(ner, last=True)
       

    # add labels
    for _, annotations in TRAIN_DATA:
         for ent in annotations.get('entities'):
            ner.add_label(ent[2])

    # get names of other pipes to disable them during training
    other_pipes = [pipe for pipe in nlp.pipe_names if pipe != 'ner']
    with nlp.disable_pipes(*other_pipes):  # only train NER
        optimizer = nlp.begin_training()
        for itn in range(iterations):
            print("Statring iteration " + str(itn))
            random.shuffle(TRAIN_DATA)
            losses = {}
            for text, annotations in TRAIN_DATA:
                nlp.update(
                    [text],  # batch of texts
                    [annotations],  # batch of annotations
                    drop=0.2,  # dropout - make it harder to memorise data
                    sgd=optimizer,  # callable to update weights
                    losses=losses)
            print(losses)
    return nlp


prdnlp = train_spacy(TRAIN_DATA, 40)

# Save our trained Model
modelfile = input("Enter your Model Name: ")
prdnlp.to_disk(modelfile)

#Test your text
test_text = input("Enter your testing text: ")
doc = prdnlp(test_text)
for ent in doc.ents:
    print(ent.text, ent.start_char, ent.end_char, ent.label_)
```

元々の電話番号とFAX番号混在のテキスト。うまく抽出出来た。

```
Enter your testing text: お電話はこちらから: 03-3333-4444、ご注文はFAXからでも行えます。FAX:03-3333-4445 ご注文お待ちしております。
03-3333-4444 11 23 TEL
03-3333-4445 44 56 FAX
```

電話番号とFAX番号を逆転してみたテキスト。こちらも正しく抽出出来た。

```
Enter your testing text: ご注文はFAXからでも行えます。FAX:03-3333-4445 ご注文お待ちしております。お電話はこちらから: 03-3333-4444
03-3333-4445 20 32 FAX
03-3333-4444 57 69 TEL
```

教師データないテキスト。こちらも上手く抽出出来た。

```
Enter your testing text: 〒111-1111 神奈川県平塚市浅間町1番1号 別館1階 直通電話：03-2222-3333 FAX番号：03-2222-3331
-1111 4 9 FAX
03-2222-3333 35 47 TEL
03-2222-3331 54 66 FAX
```

## まとめ

今回試したような電話番号とFAX番号のように表記は同じだけど、異なるラベル付けをしたいときにどうしたら良いかを調べていたわけですが、教師データを短くして、値を表すテキストの特徴量が強く反映されるようにすることで、概ねうまくいきそうな気がします。

現在では、固有表現抽出は、Bi-LSTM-CRFという、深層学習であるLSTMとCRFを組み合わせるのが精度が高いと言われているけど、このくらいのシンプルさならまだ普通のCRFでもそこそこ動かせそうということが分かった。