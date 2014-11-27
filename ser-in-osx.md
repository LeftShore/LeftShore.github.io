# sed 在Mac下日常使用的差异

## 简介

[sed](https://www.gnu.org/software/sed/manual/sed.html) : stream editor，流编辑器，用coding的模式来编辑，对工程师来说， __一些频繁的批量修改、替换代码文件非常高效__ ，也相当hacker。过程基本就是正则模式匹配+修改，所以对熟悉正则表达式还是蛮有裨益的。

## 基本用法

此处略。。。          
（看阮老师的[sed 简明教程](http://coolshell.cn/articles/9104.html) 就可以很快上手了。）

## Mac差异

日常使用中触发过因OSX和原生linux 的sed命令缺省参数设置不同而使用失败的坑，团队里大都使用的Mac，记录分享下。

### linux下可选参数在Mac下被强制要求

执行命令 ```  sed -i "s/aaa/bbb/g" `grep -rl bbb xx/yy` ```后提示

    sed command a expects xxx followed by text  (这里xxx可能是任何字母)

操作失败，linux下是OK的。因为两个系统下对参数“i”的要求不一样。

参数“i”的用途是直接在文件中进行替换。为防止误操作，可提供一个后缀名，使sed在替换前对文件进行备份,如```  sed -i ".bak" "s/aaa/bbb/g" `grep -rl bbb xx/yy` ```。mac下是强制要求备份的，但linux下是可选的。

所以如果不需要备份，传入空字符串```  sed -i "" "s/aaa/bbb/g" `grep -rl bbb xx/yy` ``` 替代linux下用法即可

### 万年编码问题

说到编辑器，就离不开编码问题，sed作为programmer式的编辑工具也是。

工作时碰到报错

    sed: RE error: illegal byte sequence

这是因为在识别含有多字节编码字符时遇到了解析冲突问题，解决方式是在sed前执行下方命令，或将其加入~/.bash_profile 或 ~/.zshrc

    export LC_CTYPE=C 
    export LANG=C

改变语言编码环境，使Mac下sed正确处理单字节和多字节字符。再次执行sed命令，OK了。

不过弊端也有，在我们UTF8环境的Mac终端里，这么做会让字符中每个字节都被识别为单独的字符（终端里输入汉字会显示为16进制unicode编码）忽略UTF8的编码规则。    
虽然功能上都是OK的，不过放弃了本地化的locale，shell只能识别单字节字符，汉字的显示会变成。。。
![LC_CTYPE=C LANG=C](http://gtms01.alicdn.com/tps/i1/TB1RVnxGFXXXXc0aXXXsG0IQXXX-377-88.png)

有篇详细介绍的帖子，http://stackoverflow.com/questions/19242275/re-error-illegal-byte-sequence-on-mac-os-x

<hr>

如果还有什么小坑，欢迎分享
