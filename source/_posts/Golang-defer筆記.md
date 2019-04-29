---
title: '[Golang] defer筆記'
catalog: true
date: 2019-04-26 02:05:45
subtitle:
header-img:
tags:
- Golang
- 學習筆記
categories: Golang
---

# 前言

本篇想記錄一下，從c轉換過來golang後，接觸到比較不一樣的語法：defer

剛開始學c的FILE I/O的時候，如果I/O的任務過於複雜的時候

常常會忘記最後要把FILE pointer 關掉

```
...

FILE* fptr;

fptr = fopen("a.txt", "w+");

// perform writing

return true;

// memory leak!!

```

所以後來就養成習慣，寫完fopen馬上接著寫fclose，確保file pointer的memory有被回收

而defer寫起來的感覺也蠻符合之前的習慣

```golang
func Copy(dstName, srcName string) (written int64, error) {
    src, err := os.Open(srcName)
    if err != nil {
        return
    }
    defer src.Close()
    ...
}
```

# Rules

Defer主要的功能是讓defer後面的function statement，在function結束return後執行，故常用來做一些Clean up或Wrapper的工作，底下是[Go Blog](https://blog.golang.org/defer-panic-and-recover)裡所提出要注意的rules

## Rule 1：Defer function 執行時參數即賦值

>
> A deferred function's arguments are evaluated when the defer statement is evaluated.
>

如果被延遲執行的function帶有參數，參數的值就已被決定

```golang
func example1() {
    i := 0
    defer fmt.Print(i) // will print 0
    i = 1
    return
}
```

## Rule 2: LIFO defer function lists

>
> Deferred function calls are executed in Last In First Out order after the surrounding function returns.
>

若function中有多個defer statements時，延遲執行順序會是後進先出（LIFO)

```golang

func example2() {
    for i := 0; i < 4; i++ {
        defer fmt.Print(i) // wil print 3210
    }
}

```

## Rule 3: Defer function 可以操作原function的回傳值

>
> Deferred functions may read and assign to the returning function's named return values.
>

因為在golang裡面，可以賦予回傳值名稱，故可以用來在defer function裡面操作，如常見的clean up或return wrapper等

```
func a() (i int){
    defer func() {i++} ()
    return 1
}


func example3() {
    print a() // 2
}

```

# 小結

本篇算是golang 學習的開端，接下來幾篇想記錄一下，golang 中的panic v.s. fatal
以及defer常搭配的anonymous function所形成的closures

# Reference
1. [Defer, Panic, and Recover](https://blog.golang.org/defer-panic-and-recover) 
