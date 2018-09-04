---
layout: blog
title: 手書きのワイヤーフレームからHTMLコードを生成するAI Sketch2Codeを試す
category: blog
tags: [AI, 人工知能]
summary: 最近ちょっと界隈でざわついているSketch2Code。これは手書きのワイヤーフレームをアップロードすると人工知能がHTMLソースコードに変換して出力してくれるというツールです。
author: aharada
image: /images/blog/2018-09-04-sketch2code/screenshot.png
---

![Sketch2Code](/images/blog/2018-09-04-sketch2code/screenshot.png)

最近ちょっと界隈でざわついている[Sketch2Code](https://sketch2code.azurewebsites.net/)。

これは手書きのワイヤーフレームをアップロードすると人工知能がHTMLソースコードに変換して出力してくれるという、昔からあったようななかったような気がする便利ツールです。

この手のツールは実戦投入出来るようなクオリティであることは稀。期待せず試してみる。

## やってみよう

ぼくは元々良く手書きでワイヤーフレームを書いていたタイプなので、まずは過去書いたやつを引っ張り出してきてアップしてみる。

ちなみに今はツールを使っていることが多いけど。やはり手書きは迷いなく作れる点が良い。

![元画像1](/images/blog/2018-09-04-sketch2code/image1-1.jpg)

以下が出力結果。なんだこりゃ全然ダメだ。

![結果画像1](/images/blog/2018-09-04-sketch2code/image1-2.png)

日本語はまだムリっぽい。あと複雑なレイアウトはダメそうなので、シンプルなやつで再チャレンジしてみよう。Sketch2Codeのページにサンプル画像があるので、それを真似た感じでよくあるレイアウトを手書きしてみた。

![元画像2](/images/blog/2018-09-04-sketch2code/image2-1.jpg)

おっ今回はいい感じ。グローバルナビゲーションはなかったことにされているようだ。なにゆえ。

![結果画像2](/images/blog/2018-09-04-sketch2code/image2-2.png)

で、吐き出されたHTMLはこちら

```
<!DOCTYPE html>

<html lang="en">
<head>
    <meta name="viewport" content="width=device-width" />
    <title>HTML Result</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
          integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
</head>
<body>
    <div class="container body-content">




<div class="container">


                    <div class="row justify-content-start" style="padding-top:10px;">

                <div class="col" style="padding-top:10px;">


<img alt="Image html" width="90%" height="90%" style="max-height:500px;max-width:500px;" src="https://sketch2code.azurewebsites.net/Content/img/fake_img.svg" />


                </div>
                <div class="col" style="padding-top:10px;">

                <div class="row justify-content-end" style="padding-top:10px;">
<label>Service</label>



                </div>
                <div class="row justify-content-end" style="padding-top:10px;">
<p class="text-black-50">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit
    <br />
    sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
    <br />
    Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris
</p>
                </div>
                </div>
                </div>
                <div class="row justify-content-start" style="padding-top:10px;">

                <div class="col" style="padding-top:10px;">

                <div class="row justify-content-start" style="padding-top:10px;">
<label>Service ?</label>



                </div>
                <div class="row justify-content-start" style="padding-top:10px;">
<p class="text-black-50">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit
    <br />
    sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
    <br />
    Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris
</p>
                </div>
                </div>
                <div class="col" style="padding-top:10px;">


<img alt="Image html" width="90%" height="90%" style="max-height:500px;max-width:500px;" src="https://sketch2code.azurewebsites.net/Content/img/fake_img.svg" />


                </div>
                </div>
                <div class="row justify-content-start" style="padding-top:10px;">

                <div class="col" style="padding-top:10px;">


<img alt="Image html" width="90%" height="90%" style="max-height:500px;max-width:500px;" src="https://sketch2code.azurewebsites.net/Content/img/fake_img.svg" />


                </div>
                <div class="col" style="padding-top:10px;">

                <div class="row justify-content-end" style="padding-top:10px;">
<label>Service ?</label>



                </div>
                <div class="row justify-content-end" style="padding-top:10px;">
<p class="text-black-50">
    Lorem ipsum dolor sit amet, consectetur adipiscing elit
    <br />
    sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.
    <br />
    Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris
</p>
                </div>
                </div>
                </div>


</div>



    </div>
</body>
</html>
```

開くとこんな感じ。

![html](/images/blog/2018-09-04-sketch2code/html.png)

## 結論

- 良かった点
  - 絶対配置ではなくbootstrapベースでいい感じにレイアウトしてくるっぽい
  - ちゃんと手直しすれば使えないこともない
- 微妙な点
  - HTMLのインデントがバラバラなので、整形が必要。めんどい(ツールはあると思うけど)。
  - HTMLの構造は絶望的ではないけど、メンテナブルにするにはそれないに手直しが必要

まだちょっと実用で使えるとは言えないかなー
