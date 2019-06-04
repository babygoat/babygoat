---
title: '[SHELL] Execution Type'
catalog: true
date: 2019-06-04 10:12:29
subtitle:
header-img:
tags:
- Shell
- Source
categories:
- Linux
---

# 前言

最近開發常需要測試環境變數去overwrite default的設定檔

```bash
~> var1=env1 go run server.go

```

但如果要覆寫的環境變數一多，直接打在command line上就顯得過於冗長

這時就想到可以用env檔的方式(e.g., dev.env)創造出需要的環境

```bash
~> cat dev.env
#!/bin/bash
var1=env1
var2=env2
var3=env3
...

~>source dev.env && go run server.go
```

但觀念不清楚的狀況下常會，source | bash(sh) | . | ./[script]混用

可能造成環境變數設定不檔

本篇為整理結果筆記

# Execution Type

在LINUX/UNIX執行環境下，可分為三種exectuion type

* Fork (default)
* Source
* Exec

最主要的差異性在於執行後的結果，**是否影響當前shell的環境**

底下一一來探討

## Fork

預設的執行模式，顧名思義就是在執行script時會**fork**出一個child process來運行

故當script執行完畢時，即回到parent process(原本的shell)，而不改變環境

底下為常見的fork模式

* 使用shell interpreter

```bash

~> sh ./script

or

~> bash ./script

```

* 直接使用script filename

```bash

~>./script

```

若以script filename執行時，需有"+x" (execution的權限）

## Source

Source執行模式則是直接在目前的Shell下運行，如執行中有改變環境時，

則會**保留更動**

底下為常見的Source模式使用

* 使用source builtin

```bash

~> source ./script

```

* 使用 .(period) builtin

```bash

~> . ./script

```

## Exec

Exec的執行模式跟Source一樣，都是在當前的Shell運行，故也會**保留過程中的環境更動**

但不同的是，執行後會結束當前的Shell

且script需有可執行(+x)的權限

底下為常見的Exec模式使用

* 使用exec builtin

```bash
~> exec ./script

```

# 驗證

為了驗證剛剛的理論，可以使用[pstree](http://man7.org/linux/man-pages/man1/pstree.1.html)來觀察三種不同模式的樹狀圖

透過tmux或開啟兩個terminal，一個執行script，另外一個透過process樹狀圖觀察

底下是準備的script.sh的snippet

```bash
#!/bin/bash
read -p "Please input a number:" number
echo "Your number is:${number}" 

```

為了方便觀測，用read builtin command，等待輸入，可以比較明顯的看出執行時process的變化

## Fork

首先是fork

window 1

```bash
~> bash script.sh
Please input a number:2
Your number is: 2
~> echo "After script:${number}"
After script:
```

window 2

```bash
~> pstree -s bash
...
   \-+= 13902 babygoat bash
     \--= 14357 babygoat bash script.sh

```

明顯可以看出在fork下，bash script.sh的執行是由shell fork出的child process所執行，故回到原本shell執行echo指令時，$number的值仍為空

## Source

window 1

```bash
~> source script.sh
Please input a number: 2
Your number is: 2
~> echo "After script:${number}"
After script: 2

```

window 2

```bash
~> pstree -s bash
...
    \--= 13902 babygoat bash

```

可以看出在source模式下，並沒有fork出child process來運行script，故只能看到bash shell的process，且變數$number被保留在執行script的環境

## Exec

window 1

```bash
# 要先將script.sh的權限增加可執行
~> chmod +x script.sh
~> exec script.sh
Please input number:2
Your number is: 2
# Shell關閉

```

window 2

```bash
~> pstree -s bash
...
    \--= 13902 babygoat /bin/bash /Users/babygoat/script.sh

```

發現執行shell的process context被替換成執行script.sh了，也沒有生出額外的process，執行完畢後process即結束

也驗證了bash manual裡有關exec 的說明
> If command is supplied, it replaces the shell without creating a new process.

# Reference

1. [Bourne Shell Builtins](https://www.gnu.org/software/bash/manual/bash.html#Bourne-Shell-Builtins)
2. [linux里source、sh、bash、./有什么区别](https://www.cnblogs.com/pcat/p/5467188.html)
3. [Bash Shell – source script, sh script, ./script (2)](http://benjr.tw/97083)

