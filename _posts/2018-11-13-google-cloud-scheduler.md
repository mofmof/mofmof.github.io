---
layout: blog
title: ついにGCPでジョブスケジューリングが出来るようになったぞ！Google Cloud Schedulerを試してみる
category: blog
tags: [cron, crontab, job, Cloud Scheduler]
summary: TODO
author: aharada
image: /images/blog/2018-11-13-google-cloud-scheduler/new.png
---

FirebaseのCloudFunctionは自前でサーバサイドを持たなくてもちょっとした処理ならhttps経由で実行したり出来るので便利。実装のしづらさはあるけど、よく使っている。

そこでちょっと困ることがあって、CloudFunctionsはそれ単独では定期実行出来ないという問題がある。Google App EngineのPub/Subを使うとか、やり方はいろいろあるんだけど非常にめんどくさい。

だがついに簡単にジョブスケジューリング出来るGoogle Cloud Schedulerが誕生したのである。

[Google Cloudがマネージドcronサービスを提供、自分でcronするより楽？](https://jp.techcrunch.com/2018/11/07/2018-11-06-google-launches-cloud-scheduler-a-managed-cron-service/)

今回はFirebase経由で作ったCloudFunctionsを叩きたいので、GCPから見えるか念のためチェック。

[Google Cloud Platform](https://console.cloud.google.com/functions/list)

普通に見える。

![Functions](/images/blog/2018-11-13-google-cloud-scheduler/functions.png)

次はcloud schedulerからcloud functionが叩けるかどうか確認しておきたい。

![Cloud Scheduler初回表示](/images/blog/2018-11-13-google-cloud-scheduler/new.png)

> Google Cloud Scheduler は、フルマネージドの cron ジョブ スケジューリング サービスです。これを使用して、App Engine でジョブをトリガーしたり、Pub/Sub メッセージを送信したり、定期的なスケジュールで任意の HTTP/S エンドポイントをヒットさせたりすることができます。

httpsエンドポイントをヒットさせることができるという文言があるので、CloudFunctionが叩けることが分かる。

## やってみる

ジョブを作成する。crontabと同じ記法でジョブの周期を設定出来る。

![Cloud Schedulerジョブ作成画面](/images/blog/2018-11-13-google-cloud-scheduler/setting.png)

今すぐ実行をして、Firebase側のCloudFunctionのログが流れていることが確認出来た。

![Cloud Schedulerジョブ一覧](/images/blog/2018-11-13-google-cloud-scheduler/done.png)

これは楽ちんだな。非常に良い。
