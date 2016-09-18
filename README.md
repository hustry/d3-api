# D3 layout api
这里列举了d3.js中其中常见layout的使用。
# histogram 柱状图
```js
var data = d3.layout.histogram()
  			.bins(5)  //将数据均分为5个区间
   			(values);
```
或者自定义分界点
```js
var data = d3.layout.histogram()
  .bins([0, 2.5, 5, 7.5, 10]) //分别以0 2.5 5 1.5 10为分界点进行分界
    (values);
```
可以自定义显示的区段
```js
var data = d3.layout.histogram()
	.range([3,7])  //只有3-7之间的数据会被显示
    .bins(x.ticks(10))
    (values);
```
针对对象数组，指定取值属性
```js
var data = d3.layout.histogram()
    .bins(x.ticks(10))
    .value(function(d){return d.value;}) //选择对象的value属性
    (values);
```
返回的data为一数组，每个元素为一数组如下：<br/>
		[0:--,1:--,2:--,... =>该区间数据列表
		dx:--,              =>该区间横坐标范围大小
		x:--,               =>该区间横坐标起始坐标
		y:--                =>该区间纵坐标
		]

# pie 饼图
默认饼图的调用方式
```js
var pie = d3.layout.pie();

```
还可以指定饼图起始与终止角度(顺时针方向)
```js
var pie = d3.layout.pie()
                   .startAngle(Math.PI/2)
                   .endAngle(Math.PI*3/2);
```
另其每一段弧由弧生成器完成
```js
var arc = d3.svg.arc()
    .innerRadius(50)
    .outerRadius(radius);

svg.selectAll("path")
    .data(pie(data))
  	.enter()
  	.append("path")
    .style("fill", function(d, i) { return color(i); })
    .attr("d", arc); //这里调用弧生成器进行绘制
```
返回的data为一数组，每个元素为一对象如下：<br/>
		{ startAngle =>该区间起始角度
		  endAngle   =>该区间终止角度
		  value      =>该区间值
		}

# stack 堆叠图
生成器：
```js
var data = d3.layout
              .stack()
              .values(function(d) { return d.values; }) //values函数指定取值属性
              (dataset);
```
dataset如下：
```js
var dataset = [
  {
    "name": "group1",
    "values":   [
    { "x": 4, "y": 3 },{ "x": 5, "y": 1 },{ "x": 6, "y": 2 },{ "x": 7, "y": 2 }
  ]
  },
  {  
    "name": "group2",
    "values": [
    { "x": 4, "y": 1 },{ "x": 5, "y": 5 },{ "x": 6, "y": 3 },{ "x": 7, "y": 2 }
  ]
  },
  {  
    "name": "group3",
    "values": [
    { "x": 4, "y": 3 },{ "x": 5, "y": 7 },{ "x": 6, "y": 2 },{ "x": 7, "y": 6 }
  ]
  }
];
```
返回的data与dataset类似，针对values属性中每个对象添加热y0属性(该组数据起始高度)：<br/>
```js
var dataset = [
  {
    "name": "group1",
    "values":   [
    { "x": 4, "y": 3,"y0":0 },{ "x": 5, "y": 1,"y0":0 },{ "x": 6, "y": 2,"y0":0 },{ "x": 7, "y": 2,"y0":0 }
  ]
  },
  {  
    "name": "group2",
    "values": [
    { "x": 4, "y": 1,"y0":3 },{ "x": 5, "y": 5,"y0":1 },{ "x": 6, "y": 3,"y0":2 },{ "x": 7, "y": 2,"y0":2 }
  ]
  },
  {  
    "name": "group3",
    "values": [
    { "x": 4, "y": 3,"y0":4 },{ "x": 5, "y": 7,"y0":6 },{ "x": 6, "y": 2,"y0":5 },{ "x": 7, "y": 6,"y0":4 }
  ]
  }
];
```
# cluster集群图
生成器：
```js
var cluster = d3.layout
                .cluster()
                .size([height, width]); //注意width与height顺序
```
输入数据：
```js
//层次对象，每个对象有name属性与children数组属性
var data={
  "name":"A",
  "children":[
      {
          "name":"B01",
          "children":[
              { "name":"C01"  },
              { "name":"C02"  },
              { "name":"C03"  }
          ]
      },
      {
          "name":"B02",
          "children":[
              { "name":"C04"  },
              { "name":"C05"  }
          ]
      }
  ]
};
```
其后获取node与link
```js
  var nodes = cluster.nodes(data);
  var links = cluster.links(nodes);
```
nodes与links均为对象数组，每个对象数据结构如下:
```js
node:{
  children:[] //后代节点数组
  depth：0    //节点所处深度
  name:"A"    //节点名称
  x:          //绘制节点x坐标(此为像素坐标，无需再使用比例尺)
  y:          //绘制节点y坐标(此为像素坐标，无需再使用比例尺)
}
link:{
  source:{node节点}   //源节点
  target:{node节点}   //目标节点
}
```
#pack图
生成器：
```js
var pack = d3.layout
             .pack()
             .size([width, height]);
```
调用方式：
```js
svg.selectAll("circle")
    .data(pack.nodes(root))
    .enter()
    .append("circle")
    .attr("cx", function(d) { return d.x; })
    .attr("cy", function(d) { return d.y; })
    .attr("r", function(d) { return d.r; })
    .attr("fill", function(d,i) { return color(i); });
```
其中pack.nodes(root)返回的对象数组中对象如下：
```js
node = {
  children:[]  //子节点
  depth:0      //节点所处层次
  r:           //节点对应圆的半径
  value:       //节点对应值
  x:           //节点对应圆圆心x坐标
  y:           //节点对应圆圆心y坐标
}
```
# force力导向图
生成器：
```js
//定义力布局
var force = d3.layout
              .force()
              .size([width, height]);
```
原始数据为data,力布局调用方式如下：
```js
var data={  //节点
  "nodes":[
    {"name":"A"},    {"name":"B"},    {"name":"C"}...
  ],
  "links":[ //连接
    {"source":1,"target":0},    {"source":2,"target":0},...
  ]
};
//加载数据，启动力布局
force.nodes(data.nodes)
     .links(data.links)
     .start();
//按照力布局的节拍移动线和点的位置，直到收敛
force.on("tick", function() {
  link.attr("x1", function(d) { return d.source.x; })
      .attr("y1", function(d) { return d.source.y; })
      .attr("x2", function(d) { return d.target.x; })
      .attr("y2", function(d) { return d.target.y; });

  node.attr("cx", function(d) { return d.x; })
      .attr("cy", function(d) { return d.y; });
});
```
启动力布局后，data.nodes,data.links数据发生变化,添加了部分属性
```js
node = {
  index:0   //节点索引号
  name:"A"  //节点名称
  x:        //节点x坐标(已为视图坐标，无需比例尺转换)
  y:        //节点y坐标(已为视图坐标，无需比例尺转换)
  weight：  //权重，表征有多少节点与之相连
}
```
