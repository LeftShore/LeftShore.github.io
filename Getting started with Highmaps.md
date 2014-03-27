# highcharts地图模块——highmaps 入门

## 前言

在图表领域，世界上最有名的要算highcharts了，成熟稳定，功能强大，社区也相当活跃，[github](https://github.com/highslide-software/highcharts.com)上更新频繁,并逐渐从中分离出完整的套件，如highstock、highmaps等，再一次体现了开源的力量。

如果喜欢更底层的Javascript的矢量库，或许你会钟意[raphael](raphaeljs.com/)    
本土化做的相当靠谱的2个图表也可以满足很多中小企业的需求，如[echarts](http://echarts.baidu.com/)、[kcharts](kcharts.net)，同样很强大。      


## 背景

原本项目中需要用的图表，大都是highcharts，只是之前他一直没有成熟的地图数据展示图表，即使今年推出了highmaps，也因为没有做本地化而没有中国数据，导致思路就是引用kcharts完成地图数据展示。所以希望封装一套基于highmaps的本地化的地图图表展示组件。

## highmaps 上手



```

$("#container").highcharts("Map", {
    chart: {…}
    colorAxis: {…}
    colors: [{…}]
    credits: {…}
    drilldown: {…}
    exporting: {…}
    labels: {…}
    legend: {…}
    loading: {…}
    mapNavigation: {…}
    navigation: {…}
    plotOptions: {…}
    series: [{…}]
    subtitle: {…}
    title: {…}
    tooltip: {…}
    xAxis: {…}
    yAxis: {…}
});

```

## 载入地图数据


Highmaps 的地图信息是一些包含图表生成前的数据集合的js文件，那些数据是highcharts可识别的Series、Points。在最终SVG中,会被格式化输入在path标签的d属性里,对我们这里的中国地图来说，就是负责对各个省份进行描边。    
在初始化加在地图时，可以在页面里添加script标签引入那个js文件，他会将map注册到```Highcharts.maps```对象集合里,类似这种方式：

```
Highcharts.maps.world = [
    {
        "code": "AD",
        "path": "M 4687 2398 L 4679 2402 4679 2398 Z",
        "code3": "AND"
    },
    {
        "code": "AG",
        "path": "M 3015 3131 L 3013 3136 3020 3136 Z M 3014 3116 L 3016 3122 3018 3120 Z",
        "code3": "ATG"
    },
    ...
    {
        "code": "AI",
        "path": "M 2983 3102 L 2980 3105 2986 3102 Z M 2984 3105 L 2980 3108 2984 3110 Z",
        "code3": "AIA"
    }
]
```

这个对象可以以如下方式被读入到```series.mapData```参数

```
    $('#container').highcharts('Map', {
        title : {
            text : 'Empty map'
        },
        series : [{
            mapData: Highcharts.maps.world,
            name: 'World map'
        }]
    });
```

自定义地图也可以被载入，可以通过维基的在线地区SVG创建自己的地图，[实验地址在这](http://www.highcharts.com/studies/map-from-svg.htm),所以可以通过作图软件如 Illustrator 或 Incscape 来创建SVG，然后通过刚刚的在线实验地址，将SVG转换成json数据。

Hightmaps 也有对[GeoJSON](http://en.wikipedia.org/wiki/GeoJSON)标准的基本支持,他是对地理特征描述的一个开放标准。大多数[GIS](http://baike.baidu.com/view/5201.htm)软件支持从Shapefile或KML输出这种格式。基本思路是向highcharts官网的一个国家json数据发起jsonp请求,比如[http://www.highcharts.com/samples/data/jsonp.php?filename=germany.geo.json&callback=?](http://www.highcharts.com/samples/data/jsonp.php?filename=germany.geo.json&callback=?)。关于geojson在highmaps应用中的更多细节可以看[相关文档](http://api.highcharts.com/highmaps#Highcharts.geojson),这儿有个[简单例子](http://jsfiddle.net/gh/get/jquery/1.7.2/highslide-software/highcharts.com/tree/master/samples/maps/demo/geojson/)

Highmaps also has basic support for GeoJSON, an open standard for description of geographic features. Most GIS software supports this format as export from for instance Shapefile or KML export. Read more in the API reference and see the live demo.
