---
title: NGINX Location Block Matcher
catalog: true
date: 2019-07-10 13:28:41
subtitle:
header-img:
tags:
- Nginx
categories:
- Nginx
---

# 前言

後端開發過程中，時不時會遇到需要針對path設定cache control或做url rewrite或redirect服務

這時就會需要調整NGINX，在不同的Location Blocks根據需求作設定

NGINX支援以下三種Block Match的機制，用以設定各種Location Block

* Exact Prefix Match
* Non-exact Prefix Match
* Regular-expression Match

NGINX再根據演算法決定URL Path最終會匹配到哪個Block的設定

雖然是最常使用的NGINX configuration的機制

但目前工作上對NGINX調整的需求量很小，故常常會需要重新複習

故本篇作為筆記用，方便之後快速回憶

# Location Block Selection Algorithm

在NGINX中，一個Location Block的定義Syntax為

```nginx

location optional_modifier location_to_match {
  ...
}

```

其中影響演算法最主要的環節就是optional_modifier的使用、modifier的權重以及這些Rules擺放的先後順序

接下來會先解釋modifier的使用，最後在帶入解釋演算法中使用modifier的權重及擺放順序的影響

NGINX 支援底下幾種modifier

* (NONE): non-prefix match
* = : exact-prefix match
* ~ : case-sensitive regular expression match
* ~\* : case-insensitive regular expression match
* ^~ : non-regular prefix match

各種modifier使用例子如下

## Non-prefix match

```nginx
location /prefix1 {
        ...
}
```

可匹配: https://example.com/prefix1/test.jpg
無法匹配: https://example.com/prefix2/test.jpg

## Exact prefix match

```nginx
location = /page1 { 
	...
}
```

可匹配: https://example.com/page1
無法匹配: https://example.com/page1/index.html

## Case-sensitive regular expression match

```nginx
location ~ .(png|jpg/gif)$/{
	...
}
```

可匹配: https://example.com/dir1/dir2/page.png
無法匹配: https://example.com/dir1/dir2/page.PNG

## Case-insensitive regular expression match

```nginx
location ~* .(png|jpg|gif)$ {
	...
}
```

可匹配: https://example.com/dir1/dir2/page.PNG
無法匹配: https://example.com/dir1/dir2/page.html

## Non-regular prefix match

```nginx
location ^~ /path1 {
	...
}
```

可匹配: https://example.com/path1/page.png
無法匹配: https://example.com/path2/page.PNG

因為匹配的

主要用來處理regular expression match中的特殊狀況

## 演算法

底下是整個演算法的流程圖

![]( /images/block_selection_algorithm.png "圖1: Location Block Selection Alogrithm Flow")

* 首先，先match所有prefix match rules中，有exact match(=)的rule，如果能match的話立即apply
* 接下來match剩餘prefix match rules中，有non-exact match( (none),  ^~ )的rules，並依照底下的邏輯運行
	* 當能match (^~)的rule時，立即apply
        * 儲存最長能匹配的( (none) ) rule，等等可能會用到
        * NGINX_HTTP_MODULE 會依照字典順序排序prefix match rules，故跟nginx.conf中定義的順序無關
* 接著match其餘rules中，regular expression match(~, ~\*) rules，根據上到下的順序，如果能match的話立即apply
* 若都未能match，apply能剛剛儲存最長能匹配prefix的rule

