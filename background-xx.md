# background-xx组合拳的一个小玩法

最近项目里有个半透明的Tag组件如下，长度随字数自增，箭头可左可右。

![screenshot](http://img2.tbcdn.cn/L1/461/1/18e601ff001271b8ac6d44740bd7d5d28030078b)

开始用代码实现，分拆为【圆角矩形+圆角三角】2个元素组合实现，但总是无法完美实现，原因至少有：

1. 半透明特性，若有重合区域会透明度叠加重影。
2. 两元素的高度定位总无法匹配一致（viewport缩放导致）。

多次折腾后摸索出一个方法，使用小图片，组合多个[CSS3的background子属性](https://developer.mozilla.org/en-US/docs/Web/CSS/background)，只用一个元素实现效果：

```
  padding-right: 18px;
  background-image: url(https://img.alicdn.com/tps/TB1t0O9IFXXXXb8XXXXXXXXXXXX.png), url(https://img.alicdn.com/tps/TB1hNC6IFXXXXaEXpXXXXXXXXXX.png);
  background-position: right 0;
  background-size: contain;
  background-repeat: no-repeat, repeat-x;
  background-clip: padding-box, content-box;
```

1. 首先利用CSS3的多重背景图特性，为一个元素插2张图（图片都很小）。
2. 再使用```background-clip```的特性，将2张背景图片附着于元素的不同区域，这个特性类似box-sizing的设置。
	3. 顺带的，这里可以结合```background-origin```理解下，两种属性的值选项范围相同，但概念有差别，所以实现的效果不同。
	1. clip是裁剪，origin是起点。前者会破坏背景图完整度，后者只改变背景图出现的起点区域。具体定义[可戳这里](https://developer.mozilla.org/en-US/docs/Web/CSS/background-clip)。
	1. 
3. 之后结合双重赋值的```background-position```和```background-repeat```属性，为pading和content区域的背景图片进行差异化设置。
	1. 注意这里的repeat，矩形主体部分因为是repeat-x刷出的，所以代码里使用了```background-clip```而不是 ```background-origin``` ,后者在这里的效果被 ```background-repeat```低效。再次体现了CSS调试里常见的 __交叉干扰__ 现象
4. 这样就可以达成一个 __单元素长度自适应Tag__ 。

最终元素布局审查如图

![screenshot](http://img1.tbcdn.cn/L1/461/1/aa76a8d4c0e2460f73a8d98976f56dd4f07893a9)
