# 前言

前阵子项目中出了点问题，其中就涉及了对**attribute**和**property**异同的理解，正好趁新年伊始再看一下。
隐含二者差异的地方我会在句末用‘_!!!_’标识

1. DOM的attribute和property
1. 二者间的同步（sync）机制
1. jQuery、kissy的attr、prop使用场景
1. 拙计的ie
1. ...

# DOM的attribute和property

字面本身就容易让人混淆，翻译都有“属性”的意思。
## attribute
这个大家应该最熟悉，对DOM属性节点的概念就是它，先复习下下表
<table>
    <tr> <td>nodeType</td><td>节点类型</td> </tr>
    <tr> <td>1</td><td>元素element</td> </tr>
    <tr> <td>2</td><td>属性attribute</td> </tr>
    <tr> <td>3</td><td>文本text</td> </tr>
    <tr> <td>8</td><td>注释comments</td> </tr>
    <tr> <td>9</td><td>文档document</td> </tr>
</table>

attribute是一个DOM节点在文档树上的属性，总是对应一个nodeType为2的属性节点，并有nodeName和nodeValue对应其属性名和属性值。相关官方文档[在这](http://www.w3.org/TR/DOM-Level-3-Core/core.html#ID-637646024)  
对于nodeType为1的element节点，都会有一个**attributes属性**，是包含所有属性节点信息的NamedNodeMap，map里的属性就是这里所说的attribute。  
**attribute是直接影响文档树的**，并与之有着强对应关系，属性值严格等于其在html文档里被写入或通过以下标准方法set／get的值（除了某些特殊属性如href、class、checked、value等，下文会说明）通过标准化方法塞入的属性总会体现在innerHTML里。

    ele.hasAttribute(key) //ie6、7 没有
    ele.getAttribute(key)
    ele.setAttribute(key, value)
    ele.removeAttribute(key) //ie6、7 没有

```javascript
<body class="iambody" bdbd="yeah">
  <div aa="aaa" bb="bbb" class="iamclass"></div>
  <script>
    var div = $('div')[0];
    alert( div.getAttribute('AaA') ); // aaa
    
    div.setAttribute('xxx', 123);  
    alert( document.body.innerHTML );  // <div aa="aaa" bb="bbb" xxx="123" class="iamclass"></div>
    console.log(document.body.attributes)  //array-like Object  
  </script>
</body>
```
注意到key不区分大小写（因为html属性名不区分大小写），value总会转成string值。_!!!_
## property
每个DOM节点在js引擎里都会映射为一个js对象，储存着很属性比如id、href、innerHTML、clientWidth和方法，但只有极少部分属性会被同步到文档树的attribute上，比如id、title等
但他们和自定义属性一起存在于这个对象里，可序列化（serializable）可枚举,同时作为js对象成员标准可通过```delete```删除。

```javascript
document.body.customProp = 123;
var list = [];
for(var key in document.body) {
  list.push(key);
}
alert(list.length); // 144
alert(list.join('\n')); // 一大波属性list
delete document.body.customProp;
```

由于他是更抽象层次的对象属性，所以**不直接影响html文档**，同时因为是严格的js对象属性，自然属性名是区分大小写的。

# 二者间的同步（sync）机制

我想attribute和property最容易让人困惑就是他们间的同步机制了吧，部分属性在二者中都存在,可通过任一方式读或写，修改了一者，另一者会随之同步，注意这里说的“同步”而不是“复制”，因为……的确不是复制，部分属性值在同步过程中会发生改变，下文会介绍。

对自定义属性，二者是无关系的，attr只是读写文档树，prop只是读写js对象。
```javascript
<input id="lol" type="checkbox" customId1="dota2">
console.log( ele.getAttribute('customId1') ); // dota2
console.log( ele.customId1 ); // undefined
elem.customId2 = 'ff7';
console.log( ele.getAttribute('customId2') ) // null
```
而对标准html文档属性，简单的同步情况就是复制了
```javascript
console.log( ele.getAttribute('id') ); // lol
console.log( ele.id ); // lol
ele.id = 'dota';
console.log( ele.getAttribute('id') ); // dota
```
复杂一些的是部分属性在同步时发生的变化,就是上文曾说过的，部分属性同步时会发生改变。  
注意ie不区分二者，表现一致，下面例子是标准浏览器的结果。  
## href
href属性在ie8下和标准浏览器表现一致。
```javascript
<a id="sa" href="q/w.html">nginx.org</a>.<br/>
<a id="aa" href="/">nginx.org</a>.<br/>
<script>
    console.log(document.getElementById('sa').href); // http://itaofe.info/q/w.html
    console.log(document.getElementById('sa').getAttribute('href')); // q/w.html
    console.log(document.getElementById('aa').href); // http://itaofe.info/
    console.log(document.getElementById('aa').getAttribute('href')); // /
</script>
```
## value
property```input.value```会一次性从attribute同步属性值，反之则不会，见栗子🌰
```javascript
<body>
  <input id="input1" type="text" value="originVal1">
  <script>
    var input = document.body.getElementById('input1');
    input.setAttribute('value', 'newVal1');
    alert( input.value ) // 'newVal1',已同步 
    input.value="newVal2";
    alert( input.getAttribute('value') ) // 'newVal1',没有同步
    
    //注意value在attribute同步到property的一次性
    input.setAttribute('value', 'newVal3');
    alert(input.value) // newVal2  没有同步
  </script>
</body>
```
运行结果如注释，property的value更新时对应的attribute保持原样。此时如果通过setAttribute修改value值，只这也解释了在chrome命令控制台的2点现象：

1. js修改property来改变text input的value，页面输入框的值变化了，"Elements"面板html结构里的值却没有变化
2. 再通过setAttribute修改，"Elements"面板的属性值变了，input.value却没变。

可以认为作为表单控件，优先使用property值，没有则使用attribute,所以一般操作value不会通过attribute操作，而是直接操作property。   
value的这种**一次性单向同步**机制,本可用作表单校验用户是否有输入，可惜在ie下二者表现一致，好在各大类库已做了兼容性处理。

## class 
作为js的保留字，如果通过attribute读写class需要使用的key是“className”，但在ie<9下会因为attribute和property的融合情况而失效，所以我们都通过property的“className”读写即可。  
当然各大类库也已经帮我们做了这个事情，但知其然 知其所以然是很重要的。

## checked,readonly,disabled...
在w3c标准里还有一类属性，是存在即生效的，而不关心属性值，基本都是应用在表单元素里。

```javascript
<input id="input2" type="checkbox" checked="blabla">
<script>
  var input  = document.getElementById('input2');
  alert( input.checked ) // true
  alert( input.getAttribute('checked') ) // "blabla"
  input.setAttribute('checked', false);
  alert( input.checked ) // true
  alert( input.getAttribute('checked') ) // "false"  
</script>
```
可见将这些属性通过attribute设置为true或false均不影响被表单的识别，只要存在便生效。注意我这里用的是布尔的true／false，因为上文说了，通过attribute设置到文档树上的属性，都会被转为string型。  
这个栗子更明确的表现了attribute反应的是HTML文档里的实实在在可以看见的属性值，property则是更抽象级、更趋逻辑化。

此时就涉及到各大类库的容错性处理了(当然是传入布尔型的true／false)，虽然jQuery，kissy都有attr、prop方法，但在这类属性上，通过```ele.attr('checked', false)```的语法是可以正确使用的，偏离了w3c本意，却降低了学习门槛－ －  
不过jQuery官方文档也说了，建议通过标准的模式管理这些属性：  
>the .attr() method returns undefined for attributes that have not been set. To retrieve and change DOM properties such as the checked, selected, or disabled state of form elements, use the .prop() method.

# jQuery、kissy的attr、prop使用场景

经过以上分析，对类库attr、prop的使用原则应该也就明了了，这里再借jQuery文档的一段话：
>For example, selectedIndex, tagName, nodeName, nodeType, ownerDocument, defaultChecked, and defaultSelected should be retrieved and set with the .prop() method. Prior to jQuery 1.6, these properties were retrievable with the .attr() method, but this was not within the scope of attr. These do not have corresponding attributes and are only properties.

那些不被表征在文档树，或者简单的说，不是html结构标准属性的“属性”都应该通过prop来管理。

# 拙计的ie

首先关于xxAttribute的4个方法，在ie<8下只有getAttribute和setAttribute存在，
并且他们修改的是DOM的**property**而不是attribute。没有大问题的原因是ie<8下两者被**融合**了，但有时会造成怪异的结果。

其次在ie<9下，property和attribute会被完全同步，并保留值类型不变。
```javascript
document.body.setAttribute('ie', 123);
alert( document.body.ie === 123 );  // true in IE<9,注意是强类型等于===
```
另外一点是在ie<8下，对property和attribute的读写是相同的，并趋向于property，比如对class的读写，从property的角度出发总是没错的。

# P.S.或Summary之类的

网上有些文章将二者的区别用可序列化与否来判断，其实这是不严谨的，从上文知道，通过遍历attributes属性，也可以将attribute们序列化枚举出来。  

下表是一些差异对比
<table>
    <tr> <td>attribute</td><td>property</td> </tr>
    <tr> <td>属性名不区分大小写，属性值总是字符串</td><td>区分大小写，严格的保留原值类型</td> </tr>
    <tr> <td>直接表现在文档树上(通过innerHTML可看到)</td><td>不表现在文档树上，小部分经过“同步”借attribute影响文档树</td> </tr>
    <tr> <td>成员须通过DOM属性attributes枚举</td><td>成员可直接通过遍历js对象形式枚举</td> </tr>
    <tr> <td>最次被js使用</td><td>js逻辑、表单逻辑优先使用该值</td> </tr>
    <tr><td colspan="2">标准DOM属性会被同步，自定义属性不会</td></tr>
</table>  
如果需要使用自定义属性，就使用标准HTML5自定义attribute```data-xxx```吧，相关标准[在这](http://www.w3.org/html/wg/drafts/html/master/dom.html#custom-data-attribute)

实际情况中很多情景可以用properties管理（可能你无意中用attr()时方法内部调的是prop），考虑iebug时由甚，只有2种情况是非attribute不可

1. 自定义属性，因为他不会被同步到property里，data-xxx的标准也是attribute的子类。
2. 无法从property同步的HTML内置属性，而且你清楚的确定你需要attribute，比如初始化的input value。
