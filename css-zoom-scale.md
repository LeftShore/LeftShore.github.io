# CSS中zoom的趣用

## zoom!? ie6/7 haslayout 各种hack

一提到css zoom，上面可能是大多数前端脑海中浮出的keyword. 在遥远的IE6、7年代

```
zoom: 1;
```
帮助前端啃下了很多老IE的梗,被当做万金油在出现IEbug时祭出，不少情况屡试不爽，以至于忽视了他原本的意思。

zoom最早是IE浏览器的专有属性，后来各浏览器陆续支持，Firefox仍不支持。    
它可以设置或检索对象的缩放比例。    
此外就是一下IE hack的功能，触发ie的hasLayout属性，清除浮动、清除margin的重叠等。

即他缩放元素的原本功能很多时候是被我们忽略的，围绕这点就可以产生些有意思的想法了。

## 首先看下浏览器支持情况

<table>
<thead>
<tr>
<th class="chrome"><span>Chrome</span></th>
<th class="safari"><span>Safari</span></th>
<th class="firefox"><span>Firefox</span></th>
<th class="opera"><span>Opera</span></th>
<th class="ie"><span>IE</span></th>
<th class="android"><span>Android</span></th>
<th class="iOS"><span>iOS</span></th>
</tr>
</thead>
<tbody>
<tr>
<td class="yep" data-browser-name="Chrome">Any</td>
<td class="yep" data-browser-name="Safari">4.0+</td>
<td class="nope" data-browser-name="Firefox">None</td>
<td class="nope" data-browser-name="Opera">None</td>
<td class="yep" data-browser-name="IE">5.5+</td>
<td class="yep-nope" data-browser-name="Android">TBD</td>
<td class="yep-nope" data-browser-name="iOS">TBD</td>
</tr>
</tbody>
</table>

其中自safari4开始的版本中，有一个新值 ```zoom:reset``` ，被设置了他的元素无法被用户通过Ctrl/cmd + 等途径进行缩放。

## 一点新想法

zoom既然可以使某个元素产生缩放效果，那么可以通过这点，对一些独立功能组件进行处理，在 __不同尺寸__ 的container中以不同的缩放比例工作。

因为组件内部各部件都是同时等比缩放，所以组件本身在UI上仍是一个可工作的整体。

举个栗子，一个独立的裁剪功能组件，在页面里使用时是这样
![在完整页面里使用的组件](http://gtms04.alicdn.com/tps/i4/TB1Y_cqHXXXXXbbXVXXzkv4TXXX-2040-1154.png)

而在某些对外提供服务的独立插件里可能也用到了，但插件本身被作为弹层呼起，需求上希望能紧凑一些，难道要重写一套CSS？    
利用zoom比如 ``` zoom: 0.7; ``` 即可简单实现复用。
![被其他插件引用时](http://gtms01.alicdn.com/tps/i1/TB1l6AAHXXXXXXTXpXXpyHO6VXX-1604-1196.png)

### firefox

但火狐仍旧不支持，好在提到缩放，CSS3的tranform scale自然拿手，针对火狐 

```
-moz-transform: scale(0.7);
```

即可实现同样不过，但单是这样不够，因为CSS3 transform的缩放是以元素中心为原点进行辐射状缩放的，所以在定位上和其他浏览器会有偏差。

好在transform-origin属性正是设置这个维度的，
```
-moz-transform-origin: center top;
```

这样搭配后，即可以很低成本跨浏览器实现，且不需要利用有性能隐患的滤镜。

### 缺点

没有银弹，这也只算一种奇技淫巧，通用性上可能还不够好，一些奇特的页面元素搭配时可能会出现布局乱掉，所以不建议大面积在生产环境使用~
