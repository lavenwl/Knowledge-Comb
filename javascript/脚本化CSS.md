### 读写元素css属性

* dom.style.prop
    * 可读写行间样式, 没有兼容性问题, 
    * 碰到float这样的保留关键字属性前面添加css. eg: `cssFloat`
    * 符合属性必须拆解, 组合单词变成小驼峰式写法
    * 写入的值必须是字符串格式

### 查询计算样式

* window.getComputedStyle(ele, null);
* 计算样式只读
* 返回的计算样式的值都是绝对值, 没有相对单位
* IE8及以下浏览器不兼容

### 查询样式

* ele.currentStyle
* 计算样式只读
* 返回的计算样式的值不是经过转换的绝对值
* IE独有的属性

### 封装兼容性方法 getStyle(ele, prop);

```javascript
function getStyle(elem, prop) {
  if(window.getComputedStyle) {
    return window.getComputedStyle(elem, null)[prop];
  } else {
    return elem.currentStyle[prop];
  }
}
```

### 绑定事件的方式

* ele.onXXX = function(event) {}
    * 兼容性很好, 但是一个元素的同一个事件上只能绑定一次
    * 程序的this指向dom元素本身
* obj.addEventListener(type, fn, false)
    * IE9以下不兼容, 可以为一个事件绑定多个处理逻辑
    * 程序的this指向dom元素本身
* obj.attachEvent('on' + type, fn)
    * IE 独有, 一个事件可以绑定多个处理程序
    * 程序的this指向window对象
* 封装兼容性方法: addEvent(elem, type, handle)方法

```javascript
function addEvent(elem, type, handle) {
  if(elem.addEventListener) {
    elem.addEventListener(type, handle, false);
  } else {
    elem.attachEvent('on' + type, function() {
      handle.call(elem);
    });
  } else {
    elem['on' + type] = handle;
  }
}
```

### 解除事件处理程序

* ele.onclick = false / '' / null;
* ele.removeEventListener(type, fn, false);
* ele.detachEvent('on' + type, fn);
* 注: 如果绑定匿名函数, 则无法解除

### 事件处理模型

* 事件冒泡:
    * 结构上(非视觉上)嵌套关系的元素,存在事件冒泡功能,即同一事件, 自子元素冒泡向父元素(自下向上)
* 事件捕获:
    * 结构上(非视觉上)嵌套关系的元素, 会存在事件捕获的功能, 即同一事件,自父元素捕获至子元素(事件源元素, 自顶向下)
    * IE没有捕获事件
* 触发顺序, 先捕获后冒泡
* focus, blur, change, submit, reset, select 等事件不冒泡

### 取消冒泡和阻止默认事件

* 取消冒泡

    * W3C标准 event.stopPropagation(); 但不支持IE9以下版本
    * IE独有 event.cancelBubble = true
    * 封装取消冒泡的函数 stopBubble(event)

    ```javascript
    // 自己的封装
    function stopBubble(event) {
      if(event.stopPropagation) {
        event.stopPropagation()
      } else {
        event.cancelBubble = true;
      }
    }
    ```

* 阻止默认事件:

    * 默认事件有: 表单提交, a标签跳转, 右键菜单等
    * return false; 以对象属性的方式注册的事件才生效
    * event.preventDefault(); W3C标注, IE9以下不兼容
    * event.returnValue = false; 兼容IE
    * 封装阻止默认事件的函数 cancelHandler(event);

    ```javascript
    // 自己的封装
    function cancelHandler(event) { 
      if(event.preventDefault) {
        event.preventDefault();
      } else {
        event.returnValue = false;
      }
    }
    ```

### 事件分类

* 鼠标事件: click, mousedown, mousemove, mouseup, contextmenu, mouoseover, mouseout, mouseenter, mouseleave
* 用button来区分鼠标的按键 0 / 1 / 2
* DOM3标准规定: click事件只能监听左键, 只能通过mousedown和mouseup来判断鼠标键
* 如何解决mousedown和click的冲突, (根据时间间隔)

```javascript
var firstTime = 0;
var lastTime = 0;
var key = false;
document.onmousedown = function() {
  firstTime = new Date().getTime();
}

document.onmouseup = function() {
  lastTime = new Date().getTime();
  if (lastTime - firstTime < 300) {
    key = true;
  }
}

document.onclick = function() {
  if(key) {
    //...
  }
}
```



### 事件对象

* event || window.event 用于IE

* 事件源对象:

    * event.target: 火狐只有这个
    * event.srcElement IE只有这个
    * Chrome这两个都有
    * 兼容性写法

    ```javascript
    div.onclick = function(e) {
      var event = e || window.event;
      var target = event.target || event.srcElement;
    }
    ```

### 事件委托

* 利用事件冒泡, 和事件源对象进行处理
    * 优点:
        * 性能高, 不需要循环所有的元素一个个的绑定事件
        * 灵活 当有心的子元素时不需要重新绑定事件
* 实例: 打印点击<li>标签的内容

```javascript
// 正常写法
var li = document.getElementsByTagName('li');
var len = li.length;
for(var i = 0; i < len; i ++) {
  li[i].onclick = function() {
    console.log(this.innerText)
  }
}

// 高效写法
var ul = document.getElementsByTagName('ul')[0];
ul.onclick = function(e) {
  var event = e || window.event;
  var target = event.target || event.srcElement;
  console.log(target.innerText);
}
```

### 鼠标拖动DIV

```javascript
var div = document.getElementsByTagName('div')[0];

		function drag(elem) {
			var disX, disY;
			elem.onmousedown = function(e) {
				disX = e.pageX - parseInt(elem.style.left);
				disY = e.pageY - parseInt(elem.style.top);
				document.onmousemove = function(e) {
					var event = e || window.event;
					elem.style.left = event.pageX - disX + 'px';
					elem.style.top = event.pageY - disY + 'px';
				}
				document.onmouseup = function() {
					document.onmousemove = null;
				}
			}
		}

		drag(div);
```

### 键盘事件

* keydown, keyup, keypress
* keydown > keypress > keyup
* keydown和keypress的区别
    * keydown可以响应任意键盘按键, keypress只可以响应字符类键盘按键
    * keypress返回的ASCII码, 可以转换成对应的字符

### 文本操作类事件

* input, focus, blur, change
* input事件会做输入框任何修改时触发
* change事件会在输入框失去焦点是触发.

### 窗体操作事件(window)

* scroll: 窗体滚动事件
* load: 页面加载完毕后触发的事件

### 页面加载及渲染简单逻辑

* 形成DOMTree
* 形成CSSTree
* 形成randerTree = DOMTree + CSSTree
* 页面重排(reflow)的触发
    * dom节点的增删
    * dom节点的宽高变化
    * dom节点的display: none -> block变化
    * 计算样式: offsetWidth, offsetLeft获取

### js加载的缺点

* 加载js文件会阻塞文档,浏览器要等待js文件加载完成后进行页面渲染.
* 一些不修改页面DOM的工具类JS文件的加载,不需要阻塞页面的后续渲染

### 异步加载js

javascript异步加载有三种方案:

* defer异步加载, 但要等到dom文档全部解析完才会被执行, 只有IE能用, 也可以将代码写到内部
* async异步加载, 加载完就执行, async只能加载外部脚本, 不能把js写在标签内.

> 前面两种方式执行时也不阻塞页面

* 创建script, 插入到DOM中,加载完毕后callBack.

```javascript
// 需要提前知道加载的文件里面有一个tools对象
function loadScript(url, callback) {
  var script = document.createElement('script');
  script.type = "text/javascript";
  if (script.readState) { // IE
    script.onreadystatechange = function() {
      if (script.readState == "complete" || script.readyState == 'loaded') {
        tools[callback]();
      }
    }
  } else { // Safari Chrome firefox opera
   	script.onload = function() {
      tools[callback]();
    }
  }
}
```

