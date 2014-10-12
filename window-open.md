# 关于window.open 使用

记不得多久了，自从高级浏览器以及IE都更新了安全策略后，window.open 开始受这个策略影响会被阻止弹出,使我们心理上有些排斥这个用法。

而实际上浏览器只是阻止非用户交互立即产生的打开窗口这个操作，如果足够实时，比如直接响应click后执行```window.open```,那还是有效的。

直接上2个例子，什么都明白了。

```

//可以打开窗口
$(document).click(function(){
  window.open('http://itaofe.info/')
})

//窗口打开被阻止
$(document).click(function(){
  $.get('xxx').error(function(){
    window.open('http://itaofe.info/')
  })
})

```

可以将上面2段代码分别copy到控制台试验。我一再chrome、火狐和IE8及以上浏览器试验过，均OK

那是不是在用户交互后有了异步交互就不行了呢，实际上，浏览器机制有一个用户交互后到执行```window.open```的时间间隔判断，如果时间大于1000ms，则认为这不是用户预期的状况，就会阻止，反之则直接弹出。

```
//可以打开窗口
$(document).click(function(){
  setTimeout(function(){
    window.open('http://itaofe.info/')
  },1000)
})


//窗口打开被阻止
$(document).click(function(){
  setTimeout(function(){
    window.open('http://itaofe.info/')
  },1001)
})

```

这么说来异步请求倒是个比较独特的案例，因为很多情况一个请求响应周期是小于1000ms的，上面的例子更是用的error事件和一个无效的请求接口。这类似Node里process.nextTick()后，即在下个事件循环起点必须能够执行window.open，否则就认为“超时”，并阻止命令执行。

总之以后可以放心的在用户交互后通过js在新页面打开新窗口了 ：）
