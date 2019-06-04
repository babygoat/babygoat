---
title: '[Golang] log.Panic v.s. log.Fatal'
catalog: true
date: 2019-05-04 13:02:30
subtitle:
header-img:
tags:
- Golang
- 學習筆記
categories: Golang
---

# 前言

本篇想記錄一下，剛開始學習golang寫API server的時候想要寫error log時，所遇到的問題

底下是拜了google大神之後, 大部分寫CRUD的起手式

```golang
package main

import (
        "database/sql"
        "log"        

      _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "user:wrongpwd@tcp(host:3306)/dbname")
    if err != nil {
       log.Fatal(err) // or log.Panic(err)
    }   
    defer db.Close()

    // sql operations
}

```

panic 以及 fatal 這兩個log function便引起我的好奇心

字面意思上看起來都是很嚴重的錯誤，執行後也都立即結束程式

那到底什麼時候該用哪個才是比較符合go的convention呢？

所以本篇想要試著整理出底下幾個問題的答案

* 兩者的差異性？
* 使用的時機點？
* Go建議的做法？

# 差異性

為了進一步兩者的差別，我先從godoc的原始碼著手

```golang

// Fatal is equivalent to Print() followed by a call to os.Exit(1).
func Fatal(v ...interface{}) {
    std.Output(2, fmt.Sprint(v...))
    os.Exit(1)
}

...

// Panic is equivalent to Print() followed by a call to panic().
func Panic(v ...interface{}) {
    s := fmt.Sprint(v...)
    std.Output(2, s)
    panic(s)
}

```

原來背後是os.Exit(1) v.s panic(s)

那這兩者在執行上又有什麼差異呢？

## Panic & Recover

在查閱godoc有關於[panic](https://golang.org/src/builtin/builtin.go#L233)的說明註解之後

它其實是搭配另外一個builtin function [recover](https://golang.org/src/builtin/builtin.go#L244)一起使用的

類似C++/Java/Python，try catch block的模式

具體而言，當panic發生時，go runtime會執行幾件事

* Unwind program stack until the top-level funciton or the panic is recovered
* Execute deferred functions along the stack(LIFO)
* If not recovered, print panic goroutine stacks and exit program with status 2 (syscall)

白話來說就是

1. 當panic發生時，發生panic的function會停在呼叫panic的地方
2. 從panic的function開始依序往caller端(此goroutine)執行每個function中已紀錄的deferred functions, 因為是stack所以會是後進先出的順序，直到呼叫頂端(main或goroutine的起始function)
3. 如果過程中都沒有遇到recover，則印出error的訊息（傳進panic的參數）及印出發生panic的stack trace
4. 回傳錯誤碼2

這整個執行流程就叫做**panicking**

而recover就是用來調控panicking的流程，類似catch，當panic發生後的錯誤處理，讓goroutine能夠繼續執行

用法如下

```golang
func PanicF() {
    defer func() {
        if r := recover(); r != nil {
           // Dosomething
        }    
    }()

    // panic blocks
    ...
}
```

特別留意recover必須放在**defer的function**裡才能夠啟到回復的作用

## os.Exit(1)

而相對於panic結束前需要的繁複手續

os.Exit(1)就顯得比較直接乾脆了結

```golang
// Exit causes the current program to exit with the given status code.
// Conventionally, code zero indicates success, non-zero an error.
// The program terminates immediately; deferred functions are not run.
func Exit(code int) {
    if code == 0 {
        // Give race detector a chance to fail the program.
        // Racy programs do not have the right to finish successfully.
        runtime_beforeExit()
    }
    syscall.Exit(code)
}
```

基本上當錯誤發生時，就直接呼叫os level的system call結束process並回傳傳入的錯誤碼1

# 使用時機

以下是個人的觀點及整理收集的資訊

這個問題可以被想成os.Exit()及panic()的在例外或錯誤處理時的適用時機

## panic()

* **An unrecoverable error that the program should stop execution**

	今天如果error是程式無法自行復原（e.g., 帳密打錯），程式需要馬上終止執行，這時候就可以用panic()（log.Panic())來報錯，因為它會執行這個發生panic goroutine的deferred functions(clean up)，能做到程序的正常結束(graceful shutdown)

* **A programmer error**

	另外一種Use case可以從go的[原始碼](https://github.com/golang/go/blob/50bd1c4d4eb4fac8ddeb5f063c099daccfb71b26/src/crypto/rand/util.go#L108)看到，錯誤地方式呼叫function（e.g., 穿入參數不符合規範），這也同時符合panic結束的error status 是2的[設計](http://www.tldp.org/LDP/abs/html/exitcodes.html)

* **Deep nested errors**

	還有一種是[良葛格](https://openhome.cc/Gossip/Go/DeferPanicRecover.html)及[wiki](https://github.com/golang/go/wiki/PanicAndRecover)裡有提到，如果錯誤發生在比較深的call stacks裡 (e.g., package internal private function)，這時候就可以考慮用panic(err)直接往最外層（Exposed function)拋，再利用recover來轉譯出error，但注意在go的convention裡，panic state 不應該跨package，package之間還是要盡量要以error來溝通

## os.Exit()

查看golang的[原始碼](https://github.com/golang/go/search?p=1&q=os.Exit&unscoped_q=os.Exit)

主要可以歸類成兩種類型

* **top level function (e.g., main)**
* **test function**

直覺上可以理解為不需要做額外的clean up就直接離開，或者測試有錯就馬上停止

雖然log.Fatal()最後呼叫的是os.Exit(1)，損失了回傳error code的彈性

但我想在測試的環境中還是蠻適合使用有效率立即停止的error out

# Go的建議做法

其實在網路上並沒有針對log.Panic()及log.Fatal()的比較

godoc跟wiki也沒有找到特別針對這部份做說明

但這邊還是記錄一下幾個比較相關的資訊，留著存參

1. [Why does Go not have exceptions?](https://golang.org/doc/faq#exceptions)
2. [Why does Go not have assertions?](https://golang.org/doc/faq#assertions)
3. [when-to-use-os-exit-and-panic](https://stackoverflow.com/a/28473273)

基本上golang是鼓勵好好思考如何處裡錯誤狀況

而不是遇到就直接丟例外，讓程式直接crash

FAQ裡面也有提到golang沒有exceptions或assertions的機制也是為了這個緣故

不過針對os.Exit跟panic

stackoverflow的這個回答蠻精闢的

> So basically panic is for you, a bad exit code is for your user.

可歸咎開發者的包就用panic

要給程式使用者的錯誤資訊就用os.Exit

# 總結

底下是自己的結論

* log.Panic適用
  * 無法處理的錯誤，須立即停止程式
  * 提醒function使用的方式錯誤（e.g., 不合法的input提醒)
  * 太過深層的private function錯誤

* log.Fatal適用
  * 測試，有問題馬上修
  * 不需要clean up的main function錯誤

# Reference
1. [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover)
2. [PanicAndRecover](https://github.com/golang/go/wiki/PanicAndRecover)
3. [when to use os exit and panic](https://stackoverflow.com/questions/28472922/when-to-use-os-exit-and-panic)
4. [panic and recover](https://golangbot.com/panic-and-recover/)
5. [go defer and os exit](https://tachingchen.com/tw/blog/go-defer-and-os-exit/)
6. [Exitcodes](http://www.tldp.org/LDP/abs/html/exitcodes.html)
7. [DeferPanicRecover](https://openhome.cc/Gossip/Go/DeferPanicRecover.html)
