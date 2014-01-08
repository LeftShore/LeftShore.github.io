通过brew安装vim及插件无法使用的问题
==============================================

##  名词解释

## 背景
OSX 10.9 终端（bash已换为更牛叉的[zsh](#))由于没有以前使用的apt-get或类似内置命令组，所以安装了现在正火,有超越macports之势的[Homebrew](http://brew.sh/)。
接着就是一顿狂装，考虑到vim已经更新到7.4，历经数年的7.3却是OSX的默认vim版本,于是又重新安装了一个:

    #更新软件版本库到最新
    brew update
    brew install vim

安装完成后可以看到：

    🍺  /usr/local/Cellar/vim/7.4: 1558 files, 25M, built in 31 seconds
    ➜  ~  where vim
    /usr/bin/vim
    /usr/local/bin/vim

嗯，2个vim，然后在.zshrc里配置alias别名```alias vi="/usr/local/bin/vim"```即可正常使用vim了.

为了配合css颜色值直接附带彩色高亮，安装了ap/vim-css-color插件，这时问题出现了

    插件无效

## 分析

最开始是担心插件位置放错或者需要在`.vimrc`写配置,折腾一番后发现不是这个原因（该插件不需要额外配置，`let g:cssColorVimDoNotMessMyUpdatetime = 1`即使不配置也可以正常使用插件）
