---
layout: blog
title: gasで朝会がちょっとスムージーになったよ
category: blog
tags: [Slack, Bot, gas]
summary: 朝会のしゃべる順番をgasでSlack通知するようにしたよ。
author: sugie
image: /images/blog/2019-09-25-slack-bot-for-asakai/smooooothy.jpeg

---

mofmofでは朝会をやっており、やること・こまっていること・俺トピを出勤した順に喋ってました。

## 課題
- 出勤順が分かりづらいので、`次誰？`問題が発生する
- 毎週火曜のリモートデーの時、適当に決める必要があった

## 解決策

メンバー全員をランダムで順番決めて、朝会時に自動でSlack通知する

## 前提

SlackとGoogle Appsは導入済みの前提とします

## 準備
- Google App Scriptを用意する
- 通知したいSlackチャンネルの`incoming webhook URL`を取得する
- メンバー一覧(文字列)を取得する    
    Slackの`/who`で取得できます。generalチャンネルとかで実行すれば良さそうですね。


## 実装
- ランダム並べ替え    
メンバーをランダムで並べ替える処理を書きます。 
弊社の規模感と管理コストを考慮し、スプレッドシートは使わずにメンバー名は配列リテラルで記載します。

- 並べ替えた結果をSlack通知する処理を書きます。 
取得済の`incoming webhook URL`に並べ替えた結果をpostします。 
gas上では`UrlFetchApp`というライブラリが使えるようになっているので簡単ですね。

- 定期実行にしたいのでスケジュール登録をします。 
ただし時間単位で登録する場合1hの幅があるため、`毎日何時何分に実行`するという方法には向いてないです。 
なので、スケジュール登録をする処理をスケジュール登録するという方法にします。
    - `N時N分にスケジュール登録する処理` をfunctionで定義する
    - 上記をgasでスケジュール登録する(※実際にSlack投稿する1h以上前に設定しておく)
    - ※ `N時N分にスケジュール登録する処理` の中に使用済スケジュールを削除する処理も入れておくと良いと思います。


## 完成
`setTrigger` をgas上でスケジュール登録します
<script src="https://gist.github.com/sugiii8/359f5ec5e541585eaa1f0a7710ac6a44.js"></script>

## 結果

![Slack通知結果](/images/blog/2019-09-25-slack-bot-for-asakai/slack-notify.png)
