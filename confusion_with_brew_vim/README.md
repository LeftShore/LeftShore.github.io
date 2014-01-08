通过brew安装vim及插件无法使用的问题
==============================================

## 背景
OSX 10.9 终端（bash已换为更牛叉的[zsh + oh my zsh](https://github.com/robbyrussell/oh-my-zsh))由于没有以前使用的apt-get或类似内置命令组，所以安装了现在正火,有超越macports之势的[Homebrew](http://brew.sh/)。
接着就是一顿狂装，考虑到vim已经更新到7.4，历经数年的7.3却是OSX的默认vim版本,于是又重新安装了一个:

    #更新软件版本库到最新
    brew update
    brew install vim

安装完成后可以看到：

    🍺  /usr/local/Cellar/vim/7.4: 1558 files, 25M, built in 31 seconds
    ➜  ~  where vim
    /usr/bin/vim
    /usr/local/bin/vim

嗯，2个vim，然后在.zshrc里配置alias别名`alias vi="/usr/local/bin/vim"`即可正常使用vim了.

为了配合css颜色值直接附带彩色高亮，安装了ap/vim-css-color插件，这时问题出现了

    插件无效

## 分析

最开始是担心插件位置放错或者需要在`.vimrc`写配置,折腾一番后发现不是这个原因（该插件不需要额外配置，`let g:cssColorVimDoNotMessMyUpdatetime = 1`即使不配置也可以正常使用插件）

不是上述问题，就去看下原装vim7.3如何，打开一个css文件，顿时五颜六色亮瞎眼。  
![vim-css-color效果](http://ap.github.io/vim-css-color/screenshot.png)    
o了，那就确认可能是版本问题,我通过brew安装的vim版本是vim7.4.052,官网此时说明是_Vim 7.4.135 is the current versions_,可能是新版本修复了bug，于是通过brew更新vim但是却被提示已是最新版本。

    ➜  www  brew upgrade vim
    Error: vim-7.4.052 already installed

看起来是brew的软件清单里vim拉取的不是官网vim源信息，同时找遍官网也没找到对应小版本的vim下载入口= =!只有很概括的vim7.4下载入口。

另外一点，既然决定终端下软件都通过brew维护，也就不太想自己单独下载安装vim,那就只能自己改brew插件，不过brew是用ruby写的，这里可以编辑`/usr/local/Library/Formula/vim.rb`或执行

    brew edit vim

看到在配置头部有如下3行代码

    # This package tracks debian-unstable: http://packages.debian.org/unstable/vim
    url 'http://ftp.debian.org/debian/pool/main/v/vim/vim_7.4.052.orig.tar.gz'
    sha1 '216ab69faf7e73e4b86da7f00e4ad3b3cca1fdb8

难怪，vim这么大众的软件brew竟然是从debian的镜像下载软件，debian那边因为没有及时更新vim版本信息，导致我这里升级失败。和前端bug一样，定位到问题原因bug就解决一大半了。

## 解决
去vim官网查找那2行信息并替换vim.rb，替换后如下：

    url 'ftp://ftp.vim.org/pub/vim/unix/vim-7.4.tar.bz2'
    sha1 '601abf7cc2b5ab186f40d8790e542f86afca86b7

然后 brew upgrade vim，正常更新完毕后打开css文件，赏心悦目的颜色出来了。

#P.S.

原本使用的插件是Bundle里搜索的css-color-preview,一切ok正常显示，但用他后vi打开文件时都会卡顿一会儿，很不爽，经任行推荐ap的[vim-css-color](https://github.com/ap/vim-css-color/),效果不错，只是出现了上面的风波。

虽然最后解决的问题没什么，但解决过程的确蛮有意思。一步步分析定位，也学到一些经验，记录一下，希望对观者能有所帮助。


