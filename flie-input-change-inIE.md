# file input的在IE下不能通过程序修改值的问题

## 背景
开发upload插件在使用时，是通过监听file input 的```change```事件获取该input的值然后进行后续上传逻辑。

有一个常见的场景（可能真实用户很少会这样做）就是：

    上传图片A，然后删除图片A，再上传图片A。
    
对应html控件的变化是

    file input由用户选择文件而设值，然后通过$('.file').val('')清空，再重复第一步
    
其他浏览器都OK，但IE不行，不管是IE8还是IE10，调试发现是因为执行```$('.file').val('')```后file input的值没有发生变化，此时用户在通过打开系统文件选择框选择相同文件时，自然不会触发change事件。    
一番google后在[微软开发者官网](http://blogs.msdn.com/b/ie/archive/2008/07/02/ie8-security-part-v-comprehensive-protection.aspx)找到原因:

    the File Path edit box is now read-only. The user must explicitly select a file for upload using the File Browse dialog.
    
出于这个安全策略的原因，IE下无法通过programmatic方式修改file input的值。

## 解决方案

有问题自然得有方案，目前想到比较好的就是利用```replaceWith```的思路，在每次change回调处理完后，深度复制本身（file input）节点，然后替换掉自己插到页面上

还有什么好的方案欢迎提供
