# CSS Grid Layout 学习

### 引入
开始之前先引入一个问题，如何在不改变 html 的情况下是实现不同的页面布局？例如照片墙，页面的主框架布局（如圣杯布局），也就是相同的 dom 层次结构如何实现不同的页面布局？

### 概念
#### 什么是网格布局
> 网格是一组相交的水平线和垂直线，它定义了网格的列和行。我们可以将网格元素放置在与这些行和列相关的位置上。
网格布局是一个基于网格的二维的布局系统，有别于 `Flexbox`(一维布局系统)，`Flexbox` 解决的 *行或列* 单维度上布局的问题，而 Grid 是解决 ***行和列*** 二维层面的布局。
#### 网格容器（Grid Container）
> 通过在元素上声明 `display：grid` 或 `display：inline-grid` 来创建一个网格容器。在下面的例子中 `.container` 就是网格容器。
```
<div class="container">
  <div class="item item-1"></div>
  <div class="item item-2"></div>
  <div class="item item-3"></div>
</div>
```

#### 网格元素（Grid Item）
> 网格容器的 ***直系子元素*** 将成为网格元素。例如上面例子中的 `.item-1 .item-2 .item-3`。有一点需要说明的是，*网格元素的子元素并不是网格元素*。

#### 网格线（Grid Line）
> 构成网格结构的风格线，包括水平的*行网格线（row grid lines）*和垂直的*列网格线（column grid lines）*，如下图紫色线条为第二列网格线。
![](https://github.com/ShaoWeibin/images/blob/master/grid-line.png?raw=true)
#### 网格轨道（Grid Track）
> 两条*相邻*网格线之间的空间，即网格行或列。如下图第二条和第三条行网格线之前的紫色区域。
![](https://github.com/ShaoWeibin/images/blob/master/grid-track.png?raw=true)
#### 网格单元格（Grid Cell）
> 两个*相邻*的行网格线和两个*相邻*的列网格线交汇的空间即为网格单元格。如下图第一条第二条行网格线和第二第三列网格线交汇的紫色区域。
![](https://github.com/ShaoWeibin/images/blob/master/grid-cell.png?raw=true)
#### 网格区域（Grid Area）
> 两个*不相邻*的行网格线和两个*不相邻*的列网格线所包围的空间为网格区域，网络区域有任意数量的网络单元格组成。如下图第一第三条行网格线和第二第四列网格线所包围的紫色区域。
![](https://github.com/ShaoWeibin/images/blob/master/grid-area.png?raw=true)

### 开始布局
#### 第一步 
```javascript
<div class="container">
  <div class="item item-1" style={{background: this.color()}}>1</div>
  <div class="item item-2" style={{background: this.color()}}>2</div>
  <div class="item item-3" style={{background: this.color()}}>3</div>
  <div class="item item-4" style={{background: this.color()}}>4</div>
  <div class="item item-5" style={{background: this.color()}}>5</div>
  <div class="item item-6" style={{background: this.color()}}>6</div>
  <div class="item item-7" style={{background: this.color()}}>7</div>
  <div class="item item-8" style={{background: this.color()}}>8</div>
</div>
```
```javascript
.container {
  display: grid;
  padding: 48px;
}

.item {
  margin: 8px;
  padding: 16px;
  font-size: 40px;
  color: #ff0055;
}
```
第一步只是把 `container` 的 `display: grid`，并未做其他的控制，所有的 `div` 堆叠展示，其实只是个一维布局的效果。如下所示：
![](https://github.com/ShaoWeibin/images/blob/master/grid-layout-1.png?raw=true)

#### 第二步 两行四列
为了实现二维布局，`grid-template-columns` 和 `grid-template-rows` 正式登场，如下代码定义了一个两行四列的网格。行轨道宽为 `300px`，第一第二第三第四列轨道宽分别为 `200px 300px 400px 500px`，`css` 代码如下：
```javascript
.container {
  display: grid;
  padding: 48px;
  grid-template-rows: 300px 300px; // 行定义
  grid-template-columns: 200px 300px 400px 500px; // 列定义
}

.item {
  margin: 8px;
  padding: 16px;
  font-size: 80px;
  color: #ff0055;
  display: flex;
  justify-content: center;
  align-items: center;
}
```
![](https://github.com/ShaoWeibin/images/blob/master/grid-layout-2.png?raw=true)

#### 第三步 定义网格元素的大小
要定义 *网格元素* 的大小，就需要用到 `grid-row` 和 `grid-clomn` 属性。这里需要特殊说明的是，`grid-row` 和 `grid-column` 的值是对应的 *网格线* 的序号，从 1 开始。
```javascript
.container {
  height: 100%;
  display: grid;
  padding: 48px;
  grid-template-rows: 1fr 1fr 1fr 1fr;
  grid-template-columns: 1fr 1fr 1fr 1fr;
}

.item {
  margin: 8px;
  padding: 16px;
  font-size: 80px;
  color: #ff0055;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 200px;
}

.item-1 {
  grid-column-start: 1;
  grid-column-end: 4;
  /* 等价于 */
  /* grid-column: 1 / 5; */

  grid-row: 1 / 4;
}

.item-2 {
  grid-row: 1 / 3;
}

.item-4 {
  grid-row: 4 / 6;
}

.item-6 {
  grid-column: 3 / 5;
  grid-row: 4 / 8;
}

.item-7 {
  grid-row: 4 / 8;
}
```
![](https://github.com/ShaoWeibin/images/blob/master/grid-layout-4.png?raw=true)

#### fr 单位
> 轨道可以使用任何长度单位进行定义。 网格还引入了一个另外的长度单位来帮助我们创建灵活的网格轨道。新的 `fr` 单位代表网格容器中可用空间的一等份。
下一个网格定义将创建两列相等宽度的轨道，这些轨道会随着可用空间增长和收缩。
```javascript
.container {
  display: grid;
  padding: 48px;
  grid-template-columns: 1fr 1fr; // 等宽列定义
}
```
![](https://github.com/ShaoWeibin/images/blob/master/grid-layout-3.png?raw=true)

#### 圣杯布局
使用 `Grid` 布局实现圣杯布局相当简单，核心代码只有几行。
```javascript
/* 指定网格为三行三列，第一行，第三行，第一列，第三列轨道宽度固定 */
grid-template-rows: 200px 1fr 200px;
grid-template-columns: 200px 1fr 200px;

/* header 和 footer 占据整行 */
.header, .footer {
  grid-column: 1 / 4;
}
```
以下是圣杯布局的所有代码。
```javascript
<div class="container">
  <div class="item header" style={{background: this.color()}}>header</div>
  <div class="item nav" style={{background: this.color()}}>nav</div>
  <div class="item main" style={{background: this.color()}}>main</div>
  <div class="item aside" style={{background: this.color()}}>aside</div>
  <div class="item footer" style={{background: this.color()}}>footer</div>
</div>
```

```javascript
.container {
  height: 100%;
  display: grid;
  grid-template-rows: 200px 1fr 200px;
  grid-template-columns: 200px 1fr 200px;
}

.item {
  margin: 8px;
  padding: 16px;
  font-size: 80px;
  color: #ff0055;
  display: flex;
  justify-content: center;
  align-items: center;
}

.header, .footer {
  grid-column: 1 / 4;
}
```
![](https://github.com/ShaoWeibin/images/blob/master/grid-layout-5.png?raw=true)

### 浏览器支持情况
主流浏览器（Chrome，Safari，Firefox，Edge）都已支持。
![](https://github.com/ShaoWeibin/images/blob/master/css%20grid%20%E6%B5%8F%E8%A7%88%E5%99%A8%E6%94%AF%E6%8C%81%E6%83%85%E5%86%B5.png?raw=true)

### 总结
`CSS Grid` 的属性较多，有些属性和 `Flexbox` 的属性类似，掌握起来比较容易，在此也就不一一说明，通过以上几个简单的用法，能掌握 `Grid` 布局的思想和基本用法，此文的目的已达到。
最后回到最初的问题，我想答案已经很明确了，使用 `Grid` 布局时，`dom` 的的层级结构是采用 *扁平化* 的方式，不同于普通的布局方式，通过复杂的 `dom` 嵌套才能实现二维布局。使用 `Grid` 的优点显而易见，同一份 `dom` 结构，便可实现不同的个性化布局，是不是一件很酷的事情？

### 示例仓库
https://github.com/ShaoWeibin/Javascript/tree/master/grid-layout
### 参考资料
https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Grid_Layout
https://www.css88.com/archives/8510
https://www.css88.com/archives/8506

