# 前端

### 主流浏览器及内核

* IE\(trident\)
* Firefox\(Gecko\)
* Google chrome\(Webkit\)
* Opera\(presto\)

### css权重问题

1. !important
2. 行间样式
3. id
4. class \|属性\|伪类
5. 标签\|伪元素
6. 通配符

### Css复杂选择器

* 父子选择器
* 直接子元素选择器
* 并列选择器
* 分组选择器

### Css颜色设置

* 英文单词
* 颜色代码
* 颜色函数

### CSS垂直居中的国际做法

单行文本设置: line-height === 容器高度

### css中的em

em相对于font-size:单位

### 元素分类\(display: inline; block; inline-block\)

1. 行级元素:\(span, strong, em a del\)
   1. 内容决定元素的位置
   2. 不能通过css元素宽高
2. 块级元素:\(div p ul li ol form address\)
   1. 独占一行
   2. 可以通过css设置宽高
3. 行级块元素:\(img\)

### 盒子模型

border

padding

content

margin

### 定位\(position\)

* absolute\(应用层模型\):绝对定位, 脱离层 相对于有定位的父级定位, 如果没有以body定位
* relative\(保留原来位置进行定位\):  相对于自己原来的位置定位
* fixed\(固定定位\)
* 最好的定位: 以父元素为relative, 保证元素不票, 子元素使用absolute随意定位.

### 层模型

应用绝对定位后脱离原来的层, 随便定位

### 居中固定写法

```css
div {
    position: absolute;
    width: 100px;
    height: 100px;
    left: 50%;
    top: 50%;
    margin-top: -50px;
    margin-left: -50px;
}
```

### 触发BFC\(块级格式化上下文\)

只要满足以下任意一条件,将会触发BFC.

* body根元素

* 浮动元素：float：除none以为的值

* 绝对定位元素：position：absolute/fixed

* display：inline-block/table-cells/flex

* overflow：除了visible以外的值（hidden/auto/scroll\)

# 浮动元素产生的浮动流及消除

[外连接](https://www.jianshu.com/p/2a89af10c271)

1. 浮动元素产生浮动流, 块级元素看不到他们
2. 产生了bfc的元素以及文本都能看到浮动元素

### 文字溢出打点问题

* 单行文本问题

```css
p{
    white-space: nowrap;
    over-folw: hidden;
    text-overflow: ellipsit;
}
```

### 行高可以设置的值

```css
h3{
    line-height: normal; /* 浏览器与字体相关*/
    line-height: 1.5; /* 默认使用， 继承设置的值， 而不是计算后的值*/
    line-height: 200%;
    line-height: 50px;
    line-height: 5em;
}
```

### 关于宋体

```css
body{
    font-family: '宋体';
    font-family: '\5b8b\4f53';
    font-family: 'SimSun';
}
```

### @规则（样式模块化）

```css
@charset   设置样式表的编码
@import    导入其他样式文件
@meida     媒体查询
@font-face    自定义字体
```

### base标签

页面内所有路径的根路径, 全局新页面打开a标签

```html
<base href="http://www.baidu.com">
```

### &lt;h&gt;标签的应用

&lt;h1&gt; 一个页面只能一个， 用于页面头

&lt;h2&gt;副标题

&lt;h3&gt;板块标题

&lt;h4&gt;板块副标题

### 盒模型

* 标准盒模型
  * 总宽度=boder+width + padding
* IE盒模型（怪异盒模型）
  * 总宽度=width

```css
div{
    box-sizing: border-box;
}
```

### 渐变的实现

```css
div {/* IE10+ */
    background-image: linear-gradient(to right, #ff9000, #ff5000);
}
div {/* IE6, 7, 8, 9 */
    filter:
}
```

### 弹性盒模型

```css
div{
    display: flex
}
```

### 



