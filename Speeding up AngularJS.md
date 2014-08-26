#加速！优化Angular应用性能

## 性能的考虑

AngularJS是一个非常优秀且改变攻城狮思维方式的框架，能够帮助我们有调理的组织代码的MV*结构。    
虽然他本身已经built-in了很多性能优化工作，普通富前端应用可能不需要考虑太多，但毕竟框架很庞大，碰到逻辑复杂的场景，难免遇到瓶颈。   
Google一波已经有不少优化方案了，这里再抛出一些，欢迎砸玉~

## yes，首当其冲的就是双向绑定机制

关于性能考究 最直接也最关键的就是：减少angular内部```$$watchers```实例数量，以提高```$digest```循环的性能。也就可以保证了应用足够稳定和对交互的快速响应。

每当一个Model更新（不管是用户直接在视图中操作引起的，还是```controller```里通过注入的```service```操作改变），Angular内部都会执行__2到10遍__（```Dirty Checking```）$digest循环遍历__整个__应用去查找挂在```$scope```下的各Model的更新，期间同时处理Model可能的再次变更。

当我们创建一个数据绑定时，会隐式创建更多的$$watchers和$scope等对象，因为处于$digest监控范围，他们反过来也会使每次$digest执行更久。当应用逻辑不断复杂时，我们更需要留心。

## 几个方面

### 合理利用一次性绑定机制

Angular最近的几次更新中引入了一个很有用的功能：一次性的渲染模板某些变量，并且他们不会收到未来Model变化的影响。这对开发者改善应用性能是个福音，在这之前我们可能是这样设置模板数据：

```
<i>{{ tip }}</i>
```
使用一次性变量渲染语法则可以这样：

```
<i>{{ ::tip }}</i>
```

这样当Angular按常规处理完DOM和该变量后，他会在内部$$watchers的监控列表中删除这些一次性变量。

这非常有用了。我们知道因为便捷背后的Dirty Checking，当Angular管理有2000个双向绑定时(比如各种云盘文件列表页)，应用的反应速度就会可感知的变慢。越少的绑定数量对应用的性能加速越好。这种做法简单快速，可以有效减少$$watcher的负担。

### $scope.$apply() or $scope.$digest()？

使用Angular的过程中，总有一阵子掉进过 ```$scope.$apply()```的坑里。一不小心就会被滥用，比如解决不同插件间的逻辑等。这不是使用API的最好方式，虽然有时被误解，但他确实是相当简洁。

$scope.$apply被设计__用来告诉Angular一个Model在他的生命周期&监控范围外发生了变化__，仅此而已。我们调用他只是告诉Angular去更新一下$scope下的各Model。在正确的时机使用他非常重要，否则会降低应用性能。如果在不合适的时机使用他，还会被抛出一些 errors，比如“堆积过高的调用栈”。

我们通常使用三方插件，他们可能在Angular管理之外改变了DOM，此时就是$scope.$apply方法须要出场的时候了：    
当DOM更新完毕后（需要这层逻辑处于可控状态），调用$scope.$apply方法，他会在内部开始$digest循环，从而达到在Angular体系外更新模型。实现形式可能是这样：
```
$(selector).dialog({
  okHide: function () {
    $(this).val('xxx');
    $scope.$apply();
  }
});
```
执行后会通过```$rootScope.$digest()```使应用开始同步数据变更，这也就是$digest循环是如何在内部被激活的。这个循环会处理调用根$scope下所有的watchers直到监听器全部触发完。简单场景下处理是相当迅速的，随时间推移应用庞大后就可能会变慢了。

代替直接$scope.$apply()的方案是使用$scope.$digest，他会执行同样的$digest循环，但只在当前$scope及子$scope下执行， 相比起来开销就小多了 

这个途径唯一的风险是,如果该作用域和父$scope的Model间还有联动关系，父$scope下不会产生变化，因为$scope.$digest只会以当前$scope节点开始进行深度遍历，[不会向上追溯整个$scope树](http://stackoverflow.com/questions/12333410/why-scope-apply-calls-rootscope-digest-rather-than-this-digest)。如果有这种情况，那只能乖乖的使用$scope.$apply跑完整个循环。

### 可能的话避免使用```ng-repeat```指令

经常不知名的坑就潜伏在我们最常用的那些事物，比如ng-repeat。涉及CURD的ng-repeat当然不能动，但也有不少并无逻辑或轻逻辑的循环结构被我们直接ng-repeat渲染出来。从这个角度看ng-repeat，还是很容易被滥用，直接拖累着$digest循环。     
我们已经知道少量的绑定增加就会带来更大的多的$digest循环开销，从指令方面来说，创造一个可能的静态渲染组件，并在真正需要外都独立在Angular监控体系外是相当有用的。

一个想法是，通过```$interpolate provider``` 来获取对象编译渲染静态模板，并将其插入DOM节点，对像navigation的模块还是很有效的。总之应该保持对隐式创建的watch对象的留心。比如在想好应用结构、coding前关心一下自己可以减少多少个watchers。

## 适当的直接操作DOM

是的，某些情况下很有必要这样做。

比如另一个可能造成$$watcher数量攀升的是Angular核心指令的滥用，比如常见de ng-show/ng-hide。他们不会直接造成什么大损耗，但可能隐藏在ng-repeat的使用后造成数量急剧增加。

对无逻辑或轻逻辑的的list，比如仅仅是切换show/hide，稍微变换下方式会更有意义。比如下面常见的使用方式：
```
<li ng-show=”bool”></li>
//blabla
$scope.bool = false;
$scope.toggleBool = function () {
  $scope.bool = true;
};
```
如果创建的指令以及他的逻辑并不需要Model支持，那么不要将他包装为Angular形式。
```
var $obj = $(selector);
$obj.hide();
$scope.toggleShow = function () {
  $obj.show();
};
```
这种情况下，比起通过$scope设置Model值为false/true再通过隐式$apply变更视图，更好的做法是直接通过.hide()/.show()方法调用来实现同样的逻辑。    
再比如Angular提供的指```令ng-click```等，他们在内部做的事情可不止绑定一个事件监听器，还会变成$digest循环的一部分并增添的应用的负担，而这块可能根本不需要这样做。在指令link回调中，应该更提倡使用原生或jQuery的添加事件监听的方法。

这样做可能很好的帮我们分离Angular逻辑无须他介入的场景。在这个例子里，我们不依赖通过在指令里改变Model来显示/隐藏元素，而是直接像普通DOM那样修改，节省了$$watchers也就加速了应用。

### 节制DOM过滤器的使用

Angular过滤器非常简单易用，变量后插入管道符再加过滤器名称就OK了。然而一旦某些东西变化了，每个过滤器在每次$digest循环里会执行2次。这还是蛮繁重的。    
其中第一次运行是源自$$watchers检测任何变化，第二次运行是看是否存在在上次循环里又变化的需要更新的数据。      

下面是一个过滤器的例子，最慢的使用方式。
```
{{ filter_expression | filter : expression : comparator }}
```
如果能避免使用inline过滤器语法，并预处理我们的数据而不是等到$digest的时候，性能就会优化不少。    
Angular自带的```$filter```provider可以让我们的filter在编译到DOM中之前进行一些逻辑处理，比如在把数据发送到View前进行预处理，也就避免了编译到DOM阶段的开销。
```
$filter('filter')(array, expression, comparator);
```

## 一点小结

上面讨论的一些关注点可以可以帮助我们开发性能更优异的应用，下面是一些建议小结，欢迎提出见解。

* 在进入coding前，考虑一下怎样设计应用框架可以避免因Dirty Check造成的性能损耗。
* 留意ng-repeat,使用场景会存在复杂前端逻辑吗，不要滥用，小心他对$digest循环带来的大量开销。
* Angular不是银弹，不是所有功能都要整成Angular型。在指令里可能有许多case是应该直接操作DOM而不是通过Model。
* 越多的$$watchers意味着更慢应用，多关注细节使用可能会带来很大的差异：）
