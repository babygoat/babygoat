---
title: Docker log 踩雷紀錄
catalog: true
date: 2019-08-30 11:36:09
subtitle:
header-img:
tags:
- Docker
- Golang
categories: Docker
---

紀錄一下最近建置logging system所踩的坑，以及思路

因為容器化(Containerized)的關係

目前大部分application log都是直接輸出至container裡的STDOUT/STDERR

再由docker engine的logger daemon輸出至console

底下是這次遇到狀況的簡化版


```go
// main.go
package main

import (
	"log"
	"net/http"

	"github.com/gin-gonic/gin"
)

func main() {	
	r := gin.New()
	
	// Define new format for structured log
	r.Use(gin.LoggerWithFormatter(func(param gin.LogFormatterParams) string{
		return fmt.Sprintf("{JSON Format Log}")
	}))

	r.Use(gin.Recovery())

	r.GET("/ping", func(c *gin.Context){
 		c.JSON(http.StatusOK, gin.H{
			"message": "pong",
		})
	})
	r.POST("/doSomething", func(c *gin.Context){	
		log.Println("Do Something")
		c.JSON(http.StatusCreated, gin.H{
			"status": "success",
		})
	})

	log.Println("Start Server!")
	r.Run()
}

```

使用的Dockerfile

```docker
FROM golang:1.12.6-alpine3.10

WORKDIR /go/cache

COPY go.mod go.sum .
RUN go mod download
RUN go mod verify

WORKDIR /src/
COPY main.go .

RUN go install

CMD main
```


為了要統整各個app的log format，首先就必須先定義好格式

所以我先從route (request response) log下手

將原本gin的default格式

```bash
...
[GIN] 2019/08/30 - 14:28:27 | 200 |     211.808µs |             ::1 | GET      /ping
...

```

轉換成json格式

在local terminal端測試沒問題之後

就build出test docker image做最後測試

而這時候出現了奇怪的狀況

在加了log formatter的middleware了之後，/ping的 route log就消失了

怎麼ping都沒有log，似乎有被buffer住的狀況，但STDERR的訊息卻能正常顯示(Start Server)

但這並不符合我對golang I/O的理解，預設的gin logwriter應該是寫到os.Stdout

而os.Stdout是採用unbuffered I/O的方式[1]

且原本在local terminal端測試時，是能夠正常顯示每一條log的

為了避免混淆

決定只關注STDOUT所產生的docker log

```
~> docker logs -f [CONTAINER\_ID] 2\>/dev/null

```

依然毫無所獲

與同事討論後，一開始也毫無頭緒，甚至懷疑是否是base image的os所造成的影響

直到他測試到另一個 /doSomething 的route，裡面帶有其他logger的訊息

這時先前一大串 /ping 的log才傾巢而出

這才找到moby(docker community)上的這條issue[2]

原來docker logs預設的buffer size是16 byte[3]

而在轉換之前能夠正常顯示log的原因

是因為docker預設是使用'\n'當作分隔符(delimiter)的

因為其他logger預設加上了換行，所以能夠正常的輸出(Start Server)

可能對很多人是常識的小細節

但無意間踩到還是做個紀錄

以及反省在Debug時，應多做不同的測試想辦法歸納出具體pattern

而不該只在錯誤處附近打轉，造成自以為了解問題資訊的幻想

Ref:
1. [go-nuts-os-stdout-is-not-buffered](https://grokbase.com/t/gg/golang-nuts/158cgb3rfd/go-nuts-os-stdout-is-not-buffered)
2. [moby/moby#20119](https://github.com/moby/moby/issues/20119)
3. [docker default buffer size](https://github.com/moby/moby/blob/master/daemon/logger/copier.go#L21)
