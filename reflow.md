# 主要记下reflow吧

repaint(重绘)在元素的外观经过改变，但没改变页面layout布局的情况下发生，如改变visibility、background、color、前景色等；    
当repaint发生时，浏览器会验证DOM树上的所有其它结点的visibility属性。

reflow(回流)则是引起DOM操作相关脚本执行时低性能的一个关键点。页面上任一节点触发reflow，都会导致它的子节点、祖先节点重新渲染。

Nicole在[一篇老文](http://www.stubbornella.org/content/2009/03/27/reflows-repaints-css-performance-making-your-javascript-slow/)里总结了在哪些情况下会导致reflow发生：

1. 窗囗大小改变
2. 文字的font各样式属性改变
3. stylesheet添加/删除
4. 改变内容，如用户在输入框中敲字
5. 激活伪类，如:hover (IE里是一个兄弟结点的伪类被激活)
6. 操作class属性
7. JS操作DOM
8. 读取offsetWidth和offsetHeight等layout属性
9. 设置style属性

降低影响的基本途径有以下几点

1. 尽可能限制reflow的影响范围。以上面的代码为例，要改变p的样式，class不要加在div上，通过父级元素影响子元素不好。最好直接加在p上。
2. 通过设置style属性改变结点样式的话，每设置一次都会导致一次reflow。所以最好通过设置class的方式。
3. 实现元素的动画，它的position属性应当设为fixed或absolute，这样不会影响其它元素的布局。
4. 权衡速度的平滑。比如实现一个动画，以1个像素为单位移动这样最平滑，但reflow就会过于频繁，CPU很快就会被完全占用。如果以3个像素为单位移动就会好很多。
5. 不要用tables布局的另一个原因就是tables中某个元素一旦触发reflow就会导致table里所有的其它元素reflow。在适合用table的场合，可以设置table-layout为auto或fixed，这样可以让table一行一行的渲染，这种做法也是为了限制reflow的影响范围。
6. 很多情况下都会触发reflow，如果css里有expression，每次都会重新计算一遍。
