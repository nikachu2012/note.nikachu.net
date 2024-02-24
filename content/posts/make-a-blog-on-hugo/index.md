+++
title = 'ブログ的なやつをHugoで作ってCloudflare Pagesで公開する'
author = 'nikachu2012'
description = "ブログ的なウェブサイトを完全無料で、Hugoを用いて作成し、Cloudflare Pagesで公開する方法を紹介しています。"
date = 2024-02-20T13:21:51+09:00
tags = ["howto", "hugo", "blog"]
series = []
alias = []
draft = true
+++

こんにちは。音楽の独唱テストで死んだニカチュウです

## 各種インストール
### Hugoのインストール

[Installation | Hugo](https://gohugo.io/installation/)を参考にし、Hugoをインストールする。  

簡潔に言えば、Mac(Homebrew)なら、
```shell
brew install hugo
```

Windowsなら、
```shell
winget install Hugo.Hugo.Extended
```

Linux(apt)なら、
```shell
sudo apt install hugo
```

とかでインストールできると思う。

### Gitのインストール
入っているPCもあると思うが、Gitをインストールする。GitHub Desktopでも良い。
GitHub Desktopについては、[GitHub使う]({{% relref "/posts/github_tukau" %}})に記事がある。


## 今回の環境
今回はMacBook Pro(M1)、Hugo v0.122.0で進める。

```
$ hugo version  
hugo v0.122.0-b9a03bd59d5f71a529acb3e33f995e0ef332b3aa+extended darwin/arm64 BuildDate=2024-01-26T15:54:24Z VendorInfo=brew
```

## テーマ選定
ブログのテーマを決める。私はデザイン能力が`null`なので、他の人が公開しているテーマを利用する。

    hugo theme

とでも検索すれば色々出てくると思う。

また、このブログは、[Paper](https://github.com/nanxiaobei/hugo-paper)というテーマをそのまま利用している。  
今回はPaperを用いた方法を解説する。

## ブログを作る
適当なフォルダに移動し、以下のコマンドを実行する。
```shell
hugo new site サイト名
```

実行すると、Hugoが設定済みの、サイト名の部分に指定した名前のフォルダが作成されるので、移動する。
```shell
cd サイト名
```

これでHugoの部分は完成した。

## テーマのインストール
テーマはほとんどがgitで公開されているため、フォルダでgitを使えるようにする。

サイト名のフォルダに移動した状態[^1]で、以下のコマンドを実行すると、フォルダでgitが使えるようになる。
```shell
git init
```

次にgitのsubmoduleという機能を使って、テーマを自分のサイトにコピーする。
```shell
git submodule add https://github.com/nanxiaobei/hugo-paper themes/paper
```

hugo.tomlの最後に、`theme = "paper"`を




[^1]: `cd サイト名`を行なった状態

