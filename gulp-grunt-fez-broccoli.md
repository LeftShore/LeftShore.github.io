## 简介


[gulp](http://gulpjs.com/)作为一个新生事物，挑战grunt在构建工具领域的king地位。

网上关于gulp的介绍很多了，这里不再赘述，给大家转一个ppt，年前看的，现在发现对gulp感兴趣的人数越来越多，这里转载一下。

简单的说有几个新颖的观念被gulp带入：

* stream ——可以说，Thinking in streams made much more sense。
* [code over configuration](https://en.wikipedia.org/wiki/Convention_over_configuration) ——g**file不再是一个大部分为json的配置文件，而是一个标准的js程序文件,带来了非常大的灵活性，以及入门难度。

## 区别

1. 上面有说，g**file.js是code，而不是config。
2. 你可以用node和javascript的标准库进行流程控制flow control。
3. 我觉得蛮重要的，一个插件只做一件事情，保证本体的基元化。可能不超过20行代码。
4. 任务可以最大并行程度的执行
5. I/O流程会像你脑中想的那样执行，以流的形式，输入——输出——输入——输出；而不是中间生成很多临时文件然后clean掉他们，再重复过程。

<iframe src="http://slid.es/contra/gulp/embed?style=dark" width="576" height="420" scrolling="no" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<br>

## 题外话
不得不说现在构建工具的世界建设的如火如荼，这不，gulp茁壮成长时，后起之秀Fez诞生了，并号称

> Grunt isn’t a build tool at all. Rather, it simply plays host to build steps, among any number of other non-build tasks.

这顶毡帽与grunt的基于任务、gulp的基于流不同，它基于文件，没错，是不是想起以前的Make脚本了。因此Fez的愿景是

> the intelligence of Make and the flexibility of Grunt

入门了解可以看[Fez官方github页](http://fez.github.io/)

另外还有基于增量编译适合大规模项目的[Broccoli](http://www.solitr.com/blog/2014/02/broccoli-first-release/)
