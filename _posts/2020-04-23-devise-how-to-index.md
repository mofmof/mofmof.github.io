---
layout: blog
title: "[Rails] devise の実践的な使い方レシピ集"
category: blog
tags: [Rails, devise, gem]
summary: deviseを実務で使うためのTips集を集めました。
author: akito-n
#image: none
---


このページでは、GitHubのdeviseリポジトリにあるWikiから[How-Tos](https://github.com/heartcombo/devise/wiki/How-Tos)、カスタマイズする時に役立つ方法を実際のコードを交えながら紹介していくページになります。

[devise](https://github.com/heartcombo/devise)のGitHub

これまでのDeviseに関する記事はこちら

<a href="/blog/devise-introduction">最小構成、導入からログアウト処理まで</a>

<a href="/blog/devise-opt-01">Database Authenticatable, Registerableについてのメソッドレシピ集</a>

<a href="/blog/devise-opt-02">Validatable Recoverable Rememberableについてのメソッドレシピ集</a>

<a href="/blog/devise-opt-03">Confirmable Lockable Timeoutable Trackableについてのメソッドレシピ集</a>

# ワークフローのカスタマイズ

1.[ユーザーのパスワードを自動生成（登録を簡単にする)]()
