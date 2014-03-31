## == 和 === 
js中的 == 和 ===应该说是最容易让人在初期产生迷惑的，正好看到几个国外网站对此进行了总结，之一是在[github](http://dorey.github.io/JavaScript-Equality-Table/)上,实现上用的是table,排版上看起来实在是乱    
**Craig Gidney**看不下去了，用canvas实现了一遍，原理也很简单。考虑到翻墙的因素，这里进行搬运并整理。

## 实现

```

<canvas id="drawCanvas" width="500" height="500" />

```

```

var cmp = function(v1, v2) { return v1 == v2; };
var vals = [
    ["false", function() { return false; }], 
    ["0", function() { return 0; }],
    ['""', function() { return ""; }],
    ["[[]]", function() { return [[]]; }], 
    ["[]", function() { return []; }], 
    ['"0"', function() { return "0"; }], 
    ["[0]", function() { return [0]; }], 
    ["[1]", function() { return [1]; }],
    ['"1"', function() { return "1"; }],
    ["1",function() { return  1; }],
    ["true", function() { return true; }],
    ["-1", function() { return -1; }],
    ['"-1"', function() { return "-1"; }],
    ["null", function() { return null; }],
    ["undefined", function() { return undefined; }],
    ["Infinity", function() { return Infinity; }],
    ["-Infinity", function() { return -Infinity; }],
    ['"false"', function() { return "false"; }],
    ['"true"', function() { return "true"; }],
    ["{}", function() { return {}; }], 
    ["NaN", function() { return NaN; }]
];

var canvas = document.getElementById("drawCanvas");
var ctx = canvas.getContext("2d");
var n = vals.length;
var r = 20; // diameter of grid squares
var p = 60; // padding space for labels

// color grid cells
for (var i = 0; i < n; i++) {
    var v1 = vals[i][1]();
    for (var j = 0; j < n; j++) {
        var v2 = vals[j][1]();
        var eq = cmp(v1, v2);
        ctx.fillStyle = eq ? "orange" : "white";
        ctx.fillRect(p+i*r,p+j*r,r,r);
    }
}

// draw labels
ctx.fillStyle = "black";
var f = 12;
ctx.font = f + "px Helvetica";
for (var i = 0; i < n; i++) {
    var s = vals[i][0];
    var w = ctx.measureText(s).width;
    ctx.save();
    ctx.translate(p+i*r+r/2-f*0.4,p-w-2);
    ctx.rotate(3.14159/2);
    ctx.fillText(s, 0, 0);
    ctx.restore();
}
for (var i = 0; i < n; i++) {
    var s = vals[i][0];
    var w = ctx.measureText(s).width;
    ctx.fillText(s, p-w-2, p+i*r+r/2+f*0.4);
}

// draw grid lines
ctx.beginPath();
ctx.strokeStyle = "black";
for (var i = 0; i <= n; i++) {
    ctx.moveTo(p+r*i, p);
    ctx.lineTo(p+r*i, p+r*n);
    ctx.moveTo(p, p+r*i);
    ctx.lineTo(p+r*n, p+r*i);
}
ctx.stroke();
```

得到的结果如下图

<canvas id="drawCanvas1" width="500" height="500" />
<script type="text/javascript">
var cmp = function(v1, v2) { return v1 == v2; };
var vals = [
    ["false", function() { return false; }], 
    ["0", function() { return 0; }],
    ['""', function() { return ""; }],
    ["[[]]", function() { return [[]]; }], 
    ["[]", function() { return []; }], 
    ['"0"', function() { return "0"; }], 
    ["[0]", function() { return [0]; }], 
    ["[1]", function() { return [1]; }],
    ['"1"', function() { return "1"; }],
    ["1",function() { return  1; }],
    ["true", function() { return true; }],
    ["-1", function() { return -1; }],
    ['"-1"', function() { return "-1"; }],
    ["null", function() { return null; }],
    ["undefined", function() { return undefined; }],
    ["Infinity", function() { return Infinity; }],
    ["-Infinity", function() { return -Infinity; }],
    ['"false"', function() { return "false"; }],
    ['"true"', function() { return "true"; }],
    ["{}", function() { return {}; }], 
    ["NaN", function() { return NaN; }]
];
var canvas = document.getElementById("drawCanvas1");
var ctx = canvas.getContext("2d");
var n = vals.length;
var r = 20;
var p = 60;
for (var i = 0; i < n; i++) {
    var v1 = vals[i][1]();
    for (var j = 0; j < n; j++) {
        var v2 = vals[j][1]();
        var eq = cmp(v1, v2);
        ctx.fillStyle = eq ? "orange" : "white";
        ctx.fillRect(p+i*r,p+j*r,r,r);
    }
}
ctx.fillStyle = "black";
var f = 12;
ctx.font = f + "px Helvetica";
for (var i = 0; i < n; i++) {
    var s = vals[i][0];
    var w = ctx.measureText(s).width;
    ctx.save();
    ctx.translate(p+i*r+r/2-f*0.4,p-w-2);
    ctx.rotate(3.14159/2);
    ctx.fillText(s, 0, 0);
    ctx.restore();
}
for (var i = 0; i < n; i++) {
    var s = vals[i][0];
    var w = ctx.measureText(s).width;
    ctx.fillText(s, p-w-2, p+i*r+r/2+f*0.4);
}
ctx.beginPath();
ctx.strokeStyle = "black";
for (var i = 0; i <= n; i++) {
    ctx.moveTo(p+r*i, p);
    ctx.lineTo(p+r*i, p+r*n);
    ctx.moveTo(p, p+r*i);
    ctx.lineTo(p+r*n, p+r*i);
}
ctx.stroke();
</script>

## < 的结果整理

其实 == 和 === 只要理解了原理和少量特例，还是容易理解的。但 x<y (或>)运算符则不同，他会先根据x和y的类型进行转换，转成可对比的值后再进行比较。

实现方式和上文类似，这里不再贴出，可以去[jsfiddle](http://jsfiddle.net/Mickey0/Ms3ej/1/)把玩，下面直接展示```x<y```在大部分情况的结果：

<canvas id="drawCanvas2" width="500" height="500" />
<script type="text/javascript">
var cmp = function(v1, v2) { return v1 < v2; };
var vals = [
    ["-Infinity", function() { return -Infinity; }],
    ["-1", function() { return -1; }],
    ['"-1"', function() { return "-1"; }],
    ['""', function() { return ""; }],
    ["[[]]", function() { return [[]]; }], 
    ["[]", function() { return []; }], 
    ["false", function() { return false; }], 
    ["0", function() { return 0; }],
    ["null", function() { return null; }],
    ['"0"', function() { return "0"; }], 
    ["[0]", function() { return [0]; }], 
    ["[1]", function() { return [1]; }],
    ['"1"', function() { return "1"; }],
    ["1",function() { return  1; }],
    ["true", function() { return true; }],
    ["Infinity", function() { return Infinity; }],
    ["{}", function() { return {}; }], 
    ['"false"', function() { return "false"; }],
    ['"true"', function() { return "true"; }],
    ["undefined", function() { return undefined; }],
    ["NaN", function() { return NaN; }]
];
var canvas = document.getElementById("drawCanvas2");
var ctx = canvas.getContext("2d");
var n = vals.length;
var r = 20;var p = 60;
for (var i = 0; i < n; i++) {
    var v1 = vals[i][1]();
    for (var j = 0; j < n; j++) {
        var v2 = vals[j][1]();
        var eq = cmp(v1, v2);
        ctx.fillStyle = eq ? "orange" : "white";
        ctx.fillRect(p+i*r,p+j*r,r,r);
    }
}
ctx.fillStyle = "black";
var f = 12;
ctx.font = f + "px Helvetica";
for (var i = 0; i < n; i++) {
    var s = vals[i][0];
    var w = ctx.measureText(s).width;
    ctx.save();
    ctx.translate(p+i*r+r/2-f*0.4,p-w-2);
    ctx.rotate(3.14159/2);
    ctx.fillText(s, 0, 0);
    ctx.restore();
}
for (var i = 0; i < n; i++) {
    var s = vals[i][0];
    var w = ctx.measureText(s).width;
    ctx.fillText(s, p-w-2, p+i*r+r/2+f*0.4);
}
ctx.beginPath();
ctx.strokeStyle = "black";
for (var i = 0; i <= n; i++) {
    ctx.moveTo(p+r*i, p);
    ctx.lineTo(p+r*i, p+r*n);
    ctx.moveTo(p, p+r*i);
    ctx.lineTo(p+r*n, p+r*i);
}
ctx.stroke();
</script>
