### 划分层与结构

#### 着重考虑的几点：

* 内容
* 颜色块
* 间距
* 边框

#### 样式重置

reset.css

```css
body, p, h1, h2, h3, h4{
    margin: 0;
}
ul{
    margin: 0;
    padding: 0;
    list-style: none;
}
img{
    border: none;
    vertical-align: middle;
}
a{
    text-decoration: none;
    color: #3c3c3c;
}
i{
    font-style: normal;
}
input, button{
    margin: 0;
    padding: 0;
}
button{
    outline: none;
}
body{
    font:12px/1.5 tahoma, arial, 'Hiragino Sans GB', '\5b8b\4f53', sans-serif;
    color: #3c3c3c;
    background-color: #f4f4f4;
}

/* 预定义样式 */

/* 清除浮动(三件套) */
.clearfix:after{
    content: '';
    display: block;
    clear: both;
}

fl{
    float: left;
}

fr{
    float: right;
}

.leayer{
    width: 1190px;
    margin: 0 auto;
}
```



