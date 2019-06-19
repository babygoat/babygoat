---
title: '[Golang] mac上切換多個go版本'
catalog: true
date: 2019-06-19 00:24:49
subtitle:
header-img:
tags:
- Golang
- mac
categories: Golang
---

# 前言

隨著go module在1.11的原生支援，就有想嘗試玩看看的念頭，但苦於工作上的開發需求，

然而目前開發環境仍暫時停留在1.10版，就有了念頭想找看看有沒有方便的version control tool

google了之後發現網路上大概有幾種方式

1. [gvm](https://github.com/moovweb/gvm)

2. 多個GOROOT，利用go[VERSION]的方式呼叫

3. 透過brew switch管理，參考Evan大的這篇[文章](https://www.evanlin.com/go-version-control-with-brew/)

仔細研究後決定使用原生的brew來做版本管理，但因為這篇文章時間點有點久遠

Homebrew的指令支援也有很大幅度的不同，本篇作為研究過程的筆記

# 研究過程

## GVM

GVM是一款在LINUX/UNIX系統上的golang version control tool

但在OSX上的使用還先需要安裝[Mercurial](https://www.mercurial-scm.org/downloads)及[Xcode Command Line Tools](https://apps.apple.com/tw/app/xcode/id497799835)

參考官方的[README](https://github.com/moovweb/gvm)

```bash
~> xcode-select all
...
~> brew update
...
~> brew install mercurial
```

因為這些prerequisite，決定再找找有沒有什麼目前mac有的資源就能方便切換的方式

## 多個GOROOT

接著就看到go-nuts的討論串[這篇](https://groups.google.com/d/msg/golang-nuts/uAcPk39Tb0I/yJ2hZfyMAQAJ)

golang 作者Rob Pike提供了修改GOROOT的方式

觀念上是同時並存多個go binary及libray source

利用GOROOT來決定compile時，要去哪邊找import的standard pacakges

但因為官方的installer偵測到default路徑上(/usr/loca/bin/go)有安裝時

會自動先刪除掉舊版的go

也沒辦法主動選擇其他安裝路徑

需要自己手動將舊版的搬移到其他路徑

正當覺得麻煩，想放棄這個方法時

就看到[官方教學](https://golang.org/doc/install#extra_versions)上就有提供方式

以我使用為例

```bash
~> go version
go version go1.10.8 darwin/amd64

~> go get golang.org/dl/go1.12.6
~> go1.12.6 download
```

接著就可以使用go1.12.6來使用1.12.6的環境

```bash
~> go1.12.6 version
go version go1.12.6 linux/amd64
```

然而這個方式對常常使用makefile來簡化命令的我來說，撰寫命令上不太方便

不能直接統一用同一個go binary來操作build, test, install

決定再想想別的方式

## brew switch

然後看到golang社群的host Evan大的[文章](https://www.evanlin.com/go-version-control-with-brew/)

就發現其實Homebrew除了套件管理外，也能控制套件版本

如原本不是透過Homebrew管理go的，可參考官方[uninstall教學](https://golang.org/doc/install#uninstall)先移除舊版本

再透過Homebrew管理

```bash

~> brew info go
go: stable 1.12.6 (bottled), HEAD
Open source programming language to build simple/reliable/efficient software
https://golang.org
...

```

得知現在(2019.06.19)目前homebrew上的go formula版本為1.12.6

接著安裝

```bash

~> brew install go

```

確認一下go env及go version

```bash

~> go env
...
GOROOT="/usr/local/Cellar/go/1.12.6/libexec"
GOTMPDIR=""
GOTOOLDIR="/usr/local/Cellar/go/1.12.6/libexec/pkg/tool/darwin_amd64"
...

~> go version
go version go1.12.6 darwin/amd64

~> ls -al /usr/local/bin/go
lrwxr-xr-x  1 babygoat  admin  16 Jun 19 20:01 /usr/local/bin/go -> ../Cellar/go/1.12.6/bin/go
```

接下來就卡住了

照文章說明應該可以brew upgrade go到某個版本，但嘗試了後會出現相關錯誤

```bash

~> brew upgarde go -v=1.10.8
...
Error: needless argument: -v=1.10.8

```

推測應該是因為brew版本不同的參數也不一樣

我使用的是Homebrew 2.1.6的版本

```bash
~> brew -v
Homebrew 2.1.6-2-g45863ce
Homebrew/homebrew-core (git revision ebe41; last commit 2019-06-16)
Homebrew/homebrew-cask (git revision 6d550; last commit 2019-06-16)
```

那到底應該怎麽安裝go 1.10呢

查看了formula上的[go](https://formulae.brew.sh/formula/go)

有其他go@1.9, go@1.10, go@1.11的版本

決定先把原本開發版本的go@1.10先裝回來

```bash
~> brew info go@1.10
go@1.10: stable 1.10.8 (bottled) [keg-only]
Go programming environment (1.10)
https://golang.org
Not installed
From: https://github.com/Homebrew/homebrew-core/blob/master/Formula/go@1.10.rb
==> Caveats
go@1.10 is keg-only, which means it was not symlinked into /usr/local,
because this is an alternate version of another formula.
...
~> brew install go@1.10
```

但因為go@1.10是[key-only](https://docs.brew.sh/FAQ#what-does-keg-only-mean)，所以不會自動link到/usr/local/bin/go

我們的目標是能像文章中所說透過底下指令無痛快速切換

```bash

~> brew switch go [version]

```

感覺上還差了臨門一腳

觀察了一下/usr/local/Cellar/go/1.12.6及/usr/local/Cellar/go@1.10/1.10.8

發現資料夾的階層架構是相同的且以版本命名

故推測brew switch應該是以/usr/local/Cellar/[FORMULA]底下的資料夾版本作為切換依據

所以只要將/usr/loca/Cellar/go@1.10下的資料夾挪到/usr/local/Cellar/go就解決了

```bash

~> cp -R /usr/local/Cellar/go@1.10/1.10.8 /usr/local/Cellar/go/

```

測試一下

```bash
~> brew info go  #可以發現這時候local端可以偵測到兩個版本
...
/usr/local/Cellar/go/1.10.8 (8,190 files, 337.0MB) *
  Poured from bottle on 2019-06-16 at 21:37:06
/usr/local/Cellar/go/1.12.6 (9,812 files, 452.7MB)
  Poured from bottle on 2019-06-16 at 18:38:23
...

~> brew switch go 1.10.8
Cleaning /usr/local/Cellar/go/1.10.8
Cleaning /usr/local/Cellar/go/1.12.6
3 links created for /usr/local/Cellar/go/1.10.8
~> go version
go version go1.10.8 darwin/amd64
~> go env
GOARCH="amd64"
...
GOHOSTOS="darwin"
...
GOAPTH="/Users/babygoat/go"
...
GOROOT="/usr/local/Cellar/go/1.10.8/libexec/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/Cellar/go/1.10.8/libexec/go/pkg/tool/darwin_amd64"

~> brew switch go 1.12.6
Cleaning /usr/local/Cellar/go/1.10.8
Cleaning /usr/local/Cellar/go/1.12.6
3 links created for /usr/local/Cellar/go/1.12.6
~> go version
go version go1.12.6 darwin/amd64
~> go env
GOARCH="amd64"
...
GOHOSTOS="darwin"
...
GOAPTH="/Users/babygoat/go"
...
GOROOT="/usr/local/Cellar/go/1.12.6/libexec/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/Cellar/go/1.12.6/libexec/go/pkg/tool/darwin_amd64"

```

打完收工！

可以發現到GOROOT也會自動切換

從go1.10的[release note](https://golang.org/doc/go1.10#goroot)

可以發現到go1.10以後如果沒有設定GOROOT，go tool會主動去猜測GOROOT的可能路徑

我想這也就是為什麼GOROOT會自動變更的緣故

故如果有安裝go@1.9之前的版本，有自行設定GOROOT的話

要使用這個方法時一定要記得先拿掉自己的設定啊！
