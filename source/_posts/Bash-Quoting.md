---
title: '[Bash] Quoting'
catalog: true
date: 2019-04-24 00:44:28
subtitle:
header-img:
tags:
- Linux
- Bash
categories:
- Linux
---

# 前言
最近常用shell script處理資料，一直以來對於bash的字串總是沒有特別的仔細去確認背後含義，以致於單引號(')，雙引號(")總是混著用，也常因保留符號而搞得自己需要花點時間trial and error，藉由這次機會仔細翻一下Bash的manual[1]，也順便留個心得筆記，也作為再次嘗試經營部落格的開端

# Quoting
在SHELL裡面，為了區別命令(command)、特殊字元(special characters)及一般字串，需要設計區隔兩者的方式，稱為Quoting，底下是manual內文的引用
> 3.1.2
> ...
> Quoting is used to <strong>remove the special meaning</strong> of certain characters or words to the shell.
> Quoting can be used to <strong>disable special treatment for special characters</strong>, <strong>to prevent reserved words from being recognized as such</strong>, and to <strong>prevent parameter expansion</strong>.  

主要可以分成三種方式
* 跳脫字元(Escape Character)
* 單引號(Single Quote)
* 雙引號(Double Quotes)

## 跳脫字元(Escape Character)
在Bash模式，'\'可用來跳脫還原後面緊接的字元成字面上的意義，如雙引號("),在Bash裡面是個特殊字元，當我們想把'"'當成一般字元用的時候可以這麼做

```bash
~> echo "   # "是特殊字元
dquote>
... 
~> echo \"  # 將"還原成字面上
"
```

此外,反斜線(backslash)還可用作接續的作用，如果有的command參數太多，導致易讀性下降可利用backslash來換行，Shell判讀時會自動忽略反斜線，把新的一行當作接續的參數
```bash
~> command arg1 arg2 arg3 arg4 arg5 arg6 # 過長
~> command arg1 arg2 arg3 \              # 下一行會自動接續在上面
> arg4 arg5 arg6   

```

## 單引號(Single Quote)
在Bash模式下，單引號所括住的字串會被自動還原成字面上的意思(literal value)

```
~> echo 'string'
string
```

但若字串中存在單引號，則無法正確還原字元，也無法使用backslash來還原
```
~> echo 'string\'' # '還是特殊字元
quote>
...
~> echo "'"        # '還原回一般字元
'
```

## 雙引號(Double Quote)
在Bash模式下，單引號所括住的字串也會被自動還原成字面上的意思，除了以下字元
'$', '`'(backtick), '\', '!'(history mode enabled)

```
~> echo "string"
string

~> echo "\"   # \被拿來escape後面的雙引號，而進入雙引號interactive模式
dquote>
```

可以雙引號內用反斜線來跳脫

```
~> echo "\\"
\
```

## ANSI-C Quoting & Locale Translation

此外，bash也支援ANSI-C standard的特殊字串[2]，可用$'string'來表示，在雙引號除了特殊字串外可還原回字面上的意思，這次為了取代horizontal tab，試用了這個quoting，省了不少功夫

```
~> echo "john	mary" | sed $'s/\t/,/g'
john,mary
```

也有根據系統環境翻譯功能，可透過$"string"來轉換，但這邊目前還沒有實際上的應用，就留待後續研究

# Reference
1. [Bash manual](https://www.gnu.org/software/bash/manual/bash.html#Quoting
)
2. [ANSI-C Special character](https://www.gnu.org/software/bash/manual/bash.html#ANSI_002dC-Quoting)
3. [how to ask bash that line command script continues next line](https://www.cyberciti.biz/faq/howto-ask-bash-that-line-command-script-continues-next-line/)
