---
title: Hexo 安裝使用心得
date: 2017-09-13 17:56:11
categories: Hexo
tags:
description:

---

# 前言
一直以來技術學習以及解決問題的方式，都是如下面這般絞盡腦汁想關鍵字餵Google

<!--more-->

![]( /images/multiple-pages.png "圖1:快看不到分頁標題XD")

問題解決了，東西懂了個三、四成可以應付需求，過程中掛在心中的不解處，也就雲淡風輕的隨之而過去了，雖然常常覺得遺憾，但過個兩三天就淡忘，直到下次再次踩到痛處；剛好受到柏宇[jyt0532](http://www.jyt0532.com/)、泡哥[moonblack](http://nk910216.github.io/)的啟發，NodeJS, React以及AWS的學習也有筆記學習驗證的需求，於是就有了這篇文章當作開端。

# BLOG部署
Github Repo有提供不限流量網頁空間，也可搭配git管理[文章](https://pages.github.com/)，又剛好可以多熟悉GIT CLI，就列為首選。常見的建置靜態的Tool很多如[jeykll](https://jekyllrb.com/)、[hexo](https://hexo.io/zh-tw/index.html)、[hugo](https://gohugo.io/)，其實相關比較文還不少，有興趣的可以參考一下Reference。

我的選擇還是以Template中較接近我想要的為準，剛好被[Next](https://github.com/iissnan/hexo-theme-next)的簡潔風燒到，而且後來發現hexo framework是以Node.js為底，剛好是現在學習可以觀摩Code的對像，也順便支持一下台灣人的作者 [tommy351](https://twitter.com/tommy351)，~~還有選擇的當下時間已經很晚了~~，就開始著手打造Blog！

## 設定Github Pages
可以參考官方的教學:

* [Github Pages](https://pages.github.com/)

基本上就是開一個以 **username.github.io** 為名的Repo，接下來將這個Repo clone一份在Local，等等會用這個資料夾( **username.github.io** )來init hexo資料夾

## 安裝Hexo
網路上的安裝教學其實也很多，default配了一個模版，不過不符合我需求，而要自己import喜歡的Template進去，並成功generate出預期的頁面也是需要花了一番功夫研究。


### 預備環境
安裝前，我們需要先準備好兩個東西: **Node.js**, **Git**

* [Node.js安裝教學](http://ithelp.ithome.com.tw/articles/10157347)
* [Git安裝教學](https://git-scm.com/book/zh-tw/v2/%E9%96%8B%E5%A7%8B-Git-%E5%AE%89%E8%A3%9D%E6%95%99%E5%AD%B8)

附上安裝的教學連結，細節這邊就不再贅述。

### 安裝Hexo-cli
接著透過npm 安裝hexo

```bash
npm install hexo-cli -g #global
```

測試看看 hexo 指令就知道有沒有安裝成功了
```bash
$ hexo -v
```
### 安裝 Hexo
接著進到剛剛 **username.github.io** 資料夾裡面，安裝套件
```bash
$ hexo init
```

這邊可能會遇到安裝上的第一個問題:
```bash
$ hexo init
FATAL ~/test not empty, please run `hexo init` on an empty
folder and then copy your files into it
FATAL Something\'s wrong. Maybe you can find the solution here: http://hexo.io/do
cs/troubleshooting.html
Error: target not empty
```
如果剛剛有先clone 一份資料夾在本機端的話，非空的資料夾會造成hexo init 失敗
可參閱[這篇討論](https://github.com/hexojs/hexo/issues/1749)

基本上解法有三種，我比較懶是採用第一種偷懶的方法
1. 先將 **username.github.io** 資料夾內的 .git/ 資料夾暫時移到別的地方，完成init之後再移回來
2. 手動更新[hexo-cli](https://github.com/hexojs/hexo-cli)到github上最新的點，目前npm上的是去年的舊版(1.0.3)，至少要拉到 6ac7fcb7 之後的點
3. 先hexo init，再關聯github repo
```bash
$ hexo init username.github.io
$ cd username.github.io
$ git init #Initialize git repo
$ git remote add origin git://github.com/username/username.github.io.git #關連Github
```

完成hexo init 之後，應該會看到幾個檔案
```bash
$ tree -L 1
|-- _config.yml       #Site configuration file (主要)
|-- db.json           #cache
|-- node_modules      #nodejs modules
|-- package.json      #nodejs package dependency
|-- scaffolds
|-- source            #文章相關素材(_posts/)
-- themes             #放置主題 module
```
### Run Server
把hexo server run起來
```bash
$ hexo server # default on port:4000, use -p for your own port
INFO  Start processing
INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

打開[瀏覽器](http://localhost:4000/)看看吧~應該可以看到預設部落格頁面！

![]( /images/default-blog.png "圖2: 預設樣式")

### Deploy 到 Github Pages
在實際deploy到Github 前，還需要一個package來協助我們上傳generate出來的靜態網頁素材: [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)
```bash
$ npm install hexo-deployer-git --save
```

接著我們要在 \_config.yml 設定我們要deploy到遠端的資訊

```bash
deploy:
  type: git
  repo: git@github.com:username/username.github.io
  branch: master
```

最後我們就可以把他deploy到github 上啦！

```bash
$ hexo clean      #Clean up public files & cache
$ hexo deploy
```

到瀏覽器輸入 http://username.github.io ，這時你應該可以看到剛剛的預設頁面，恭喜你成功一半了！

## 安裝 NexT
Hexo 提供的主題 Template有很多種，可以到他們的[主題頁](https://hexo.io/themes/)尋寶，這邊我的選擇是極簡風的[NexT](https://github.com/iissnan/hexo-theme-next)，不過照著官方的說明文件以及建議方式，實際deploy到github上時會遇到deploy頁面仍是預設主題的問題，不確定程序說明上是否有省略，故底下參考[liguanghe大](https://liguanghe.github.io/2017/05/22/blogRebuilt/)提供的方式完成。

### 下載Next主題
官方的Github上有提Release及開發中版本可供選擇:
* [Latest Release Version](https://github.com/iissnan/hexo-theme-next/releases/tag/v5.1.2) (**Stable, recommended**)
* [Tagged Release Version](https://github.com/iissnan/hexo-theme-next/tags) (**Specify the tag you want**)
* [Latest Master Version](https://github.com/iissnan/hexo-theme-next/archive/master.zip) (**Unstable, but include latest feature**)

這邊使用官方推薦的Latest Release Version, 從你的 **username.github.io** 資料夾裡開始
```bash
$ mkdir themes/next
$ curl -L https://api.github.com/repos/iissnan/hexo-theme-next/tarball/v5.1.2 | tar -zxv -C themes/next --strip-components=1 #解壓縮時忽略root資料夾
```

或直接

```bash
$ mkdir themes/next
$  git clone https://github.com/iissnan/hexo-theme-next themes/next
$ cd themes/next
$ git checkout v5.1.2 #checkout latest release tag
```

### 修改設定
1. 修改 \_config.yml 內，指定使用我們的主題:next
```bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

2. 將主題設定擋新增一份放到 source/ 資料夾，之後有關主題的偏好設定都直接更新在這邊，參考[這篇](https://github.com/iissnan/hexo-theme-next/issues/328)
```bash
$ mkdir source/_data
$ cp themes/next/_config.yml source/_data/next.yml
```
3. 刪除預設主題 themes/Landscape/

4. 刪除 themes/next 裡的 .git/ 資料夾

完成以上修改後，在Local端run起來看看，應該可以看到如下的預設主題Scheme: Muse

![]( /images/next-blog.png "圖3:NexT主題頁")

最後，我們就可以使用hexo-deployer-git上傳到github，就完成啦！
```bash
$ hexo clean      #記得一定要清除前一份public資料夾，否則推上去的還會是default的靜態頁面
$ hexo deploy
```

# 後記
真的是很久沒有打落落長的文章了，做起來很快的事情，用文字說明起來卻要咀嚼再三，果然表達能力還是需要靠寫作多加強。最後設定修改的3、4，我覺得是workaround，應該是不符合Hexo 可以自由再不同主題間切換的用意，上面主要是想繞過deploy時一直使用預設的landscape主題，而失敗的問題，但這部份目前猜測應該是跟hexo-deployer-git的實作有關。

目前的方式，NexT整個package變成掛到hexo下，之後如果要update主題版本，變成要手動砍掉重練，再重新clone一份新的，也是有些費時。這部份的maintain解法應該可以透過git submodule來處理: Fork一份 NexT source到自己的repo，再把git submodule add的方式掛到主資料夾下。可以參考Reference 4、5，這邊就先當作Future Work，短時間應該還沒有認真玩Hexo的需求，目前先當作筆記使用，先好好把Nodejs, React, AWS玩到可以面試再說吧(倒)

# Reference
1. [Jekyll vs Hugo vs. Hexo](https://stackshare.io/stackups/hexo-vs-hugo-vs-jekyll)
2. [Jekyll vs Hexo](https://www.slant.co/versus/1006/1020/~jekyll_vs_hexo)
3. [Jekyll vs Hugo](https://opensource.com/article/17/5/hugo-vs-jekyll)
4. [远程仓库无法备份theme/next主题的问题](https://github.com/iissnan/hexo-theme-next/issues/932)
5. [Submodule 解决 Hexo 主题库更新](https://leomu.gq/2017/07/31/Submodule-%E8%A7%A3%E5%86%B3-Hexo-%E4%B8%BB%E9%A2%98%E5%BA%93%E6%9B%B4%E6%96%B0/)
