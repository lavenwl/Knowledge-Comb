参考链接：[30天精通RxJS](https://blog.jerry-hong.com/series/rxjs/thirty-days-RxJS-01/)

# 前置知识

### Promise与Observable

* 为了处理页面日渐复杂的异步操作, javascript提供了, callback, Promise, async/wait等操作来应对.但异步处理还是非常困难
* RxJS框架的产生来解决各种异步处理, Observable是此框架的核心类之一

### 异步常见问题

* 竞争条件(Race Condition):  更新与获取同一个资源的两个请求会因为请求的到达时间的不同, 而产生不同的结果.
* 内存泄漏(Memory Leak): 在单页应用内监控事件没有及时的移除.
* 复杂状态(Complex State): 当来回切换菜单时, 临时点击的页面请求需要判断状态来决定后续的执行.
* 异常处理(Excepiton Handling): javascript语法中的try/catch语法可以捕获同步代码的异常, 但是不能捕获异步代码返回的异常.

### RxJS

RxJS是一套基于Obsevable sequences来组合异步行为和事件基础程序的库.

也被称为`Functional Reactive Programming`, 更确切的说是Funtional Programming 及Reactive Programming两个编程思想的结合.

> 可以吧RxJS想成异步行为的Lodash

### Rx(Reactive Extension)

Rx最早由微软开发的LinQ扩展而来.后面有开源社区工程师维护, 提供多语言支持: javascript, java, c#, Python, Ruby.....

> LinQ: 读作link, 全名是Language-Integrated Query.

### Functional Reactive Programming

functional Reactive Programming是一种编程范式(program paradigm).其涵盖了Functional Programming and Reactive Programming的思想.

### Functional Programming

面向方法(Function)编程, 就像Object-oriented Programming(OOP)一样, 是一种编程范式.

* Functional Programming基本条件: 函数为一等公民(first Class) . 函数可以复制给变量, 可以作为参数传入另外一个函数, 可以当一个返回值.
* Functional Programming重要特征: 
    * Expression(表达式), no Statement(陈述式)
    * Pure Function: 一个函数,对于相同的参数永远返回一个值, 没有副作用(Side Effect)
* Side Effect
    * 发送HTTPRequest
    * 打印log
    * 获得使用者input
    * queryDOM对象
* Referential transparency: 这种不依赖任何外部状态, 只依赖传入的参数的特性也成为`引用透明`
* 利用参数保存状态
* Functional Programming的优势
    * 可读性高
    * 可维护性高
    * 易于平行/并行处理

### Reactive Programming

当资源发生变化时, 由资源自动告诉我发生了什么

* 发生变动=> 异步: 不知道什么时候发生, 反正发生了跟我讲
* 由变动者自动告知我=> 我不需要通知我的代码

### Functional Programming 通用函数

* ForEach: 迭代每一特元素, ES5原生支持的方法

* Map是下面基本思路的一个抽象: ES5原生支持这个方法

    * 我们会让每一个集合都有一个map方法
    * 这个方法会让使用者自定义传入一个callback function
    * 这个callback回传用户预期的元素

    ```javascript
    // 我們希望每一個陣列都有 map 這個方法，所以我們在 Array.prototype 擴充 map function
    Array.prototype.map = function(callback) {
      var result = []; // map 最後一定會返回一個新陣列，所以我們先宣告一個新陣列
      
      this.forEach(function(element, index) {
    	  // this 就是呼叫 map 的陣列
    	  result.push(callback(element, index));
    	  // 執行使用者定義的 callback， callback 會回傳使用者預期的元素，所以我們把它 push 進新陣列
      })
      
      return result;
    }
    ```

* Filter: 是下面基本思路的一个抽象

    * 迭代每一个元素
    * 判断元素是否符合条件, 符合则加入新的集合中

    ``` javascript
    Array.prototype.filter = function(callback) {
    	var result = [];
    	this.forEach((item, index) => {
    		if(callback(item, index))
    			result.push(item);
    	});
    	return result;
    }
    ```

* ConcatAll: 摊平二位数组

    ``` javascript
    Array.prototype.concatAll = function() {
      var result = [];
      
      // 用 apply 完成
      this.forEach((array) => {
        result.push.apply(result, array);
      });
    
      // 用兩個 forEach 完成
      // this.forEach((array) => {
      //   array.forEach(item => {
      //     result.push(item)
      //   })
      // });
      
      // 用 ES6 spread 完成
      // this.forEach((array) => {
      //   result.push(...array);
      // })
      
      return result;
    };
    ```

### Observer Pattern

> 事件(event)与监听者(listener)的互动中实现解耦(decoupling)

``` javascript
// ES5语法实现订阅者模式
function Producer() {
	
	// 這個 if 只是避免使用者不小心把 Producer 當作函式來調用
	if(!(this instanceof Producer)) {
	  throw new Error('請用 new Producer()!');
	  // 仿 ES6 行為可用： throw new Error('Class constructor Producer cannot be invoked without 'new'')
	}
	
	this.listeners = [];
}

// 加入監聽的方法
Producer.prototype.addListener = function(listener) {
	if(typeof listener === 'function') {
		this.listeners.push(listener)
	} else {
		throw new Error('listener 必須是 function')
	}
}

// 移除監聽的方法
Producer.prototype.removeListener = function(listener) {
	this.listeners.splice(this.listeners.indexOf(listener), 1)
}

// 發送通知的方法
Producer.prototype.notify = function(message) {
	this.listeners.forEach(listener => {
		listener(message);
	})
}



// ES6实现订阅者模式
class Producer {
	constructor() {
		this.listeners = [];
	}
	addListener(listener) {
		if(typeof listener === 'function') {
			this.listeners.push(listener)
		} else {
			throw new Error('listener 必須是 function')
		}
	}
	removeListener(listener) {
		this.listeners.splice(this.listeners.indexOf(listener), 1)
	}
	notify(message) {
		this.listeners.forEach(listener => {
			listener(message);
		})
	}
}


// 订阅者使用实例
var egghead = new Producer(); 
// new 出一個 Producer 實例叫 egghead

function listener1(message) {
	console.log(message + 'from listener1');
}

function listener2(message) {
	console.log(message + 'from listener2');
}

egghead.addListener(listener1); // 註冊監聽
egghead.addListener(listener2);

egghead.notify('A new course!!') // 當某件事情方法時，執行
```

### Iterator Pattern

iterator是一个对象, 他就是一个指针指向一个结构化的序列(sequence), 这个序列会有资料结构中的所有元素(element)

* javascript原生iterator

    > javascript到了ES6才有原生的Iterator

    ```javascript
    var arr = [1, 2, 3];
    
    var iterator = arr[Symbol.iterator]();
    
    iterator.next();
    // { value: 1, done: false }
    iterator.next();
    // { value: 2, done: false }
    iterator.next();
    // { value: 3, done: false }
    iterator.next();
    // { value: undefined, done: true }
    ```

    Javascript的iterator方法只有一个next方法, 这个方法返回两个结果:

    1. 在返回最后一个元素前: { done: false, value: elem}
    2. 在返回最后一个元素之后: {done: true, value: undefined}

* 自己实现一个Iterator

    ```javascript
    // ES5 实现一个迭代器模式
    function IteratorFromArray(arr) {
    	if(!(this instanceof IteratorFromArray)) {
    		throw new Error('請用 new IteratorFromArray()!');
    	}
    	this._array = arr;
    	this._cursor = 0;	
    }
    
    IteratorFromArray.prototype.next = function() {
    	return this._cursor < this._array.length ?
    		{ value: this._array[this._cursor++], done: false } :
    		{ done: true };
    }
    
    
    // ES6实现迭代器模式
    class IteratorFromArray {
    	constructor(arr) {
    		this._array = arr;
    		this._cursor = 0;
    	}
      
    	next() {
    		return this._cursor < this._array.length ?
    		{ value: this._array[this._cursor++], done: false } :
    		{ done: true };
    	}
    }
    ```

    > 迭代器模式虽然简单, 但同时带来了两个优势:
    >
    > 1. 它渐进式取数据的特性可以拿来做`渐进式运算(Lazy evaluation)`, **让我们可以用它来处理大量数据**
    > 2. iterator本身是序列, 所以可以实现所有集合的运算方法像: map, filter...

### 延迟计算(Lazy evaluation)

延迟计算或说call-by-need, 是一种运算策略(evaluation strategy), 简单说, 我们延迟一个表达式的有运算时机,知道我们真正需要做运算.

### 什么是Observable

Observable其实就是两个设计模式思想的结合, Observable具备**生产者推送资料**`Observer Pattern`的特性, 同事能像序列一样拥有**序列处理资料**`Iterator Pattern`的方法(map, filter...) 简单说, Observable就像一个集合, 里面的元素会随着时间推送.

# RxJS应用

### 整体认识

> 一定分清`Observable`与`Observer`的区别
>
> * Observable可以同时处理**同步跟非同步**的行为
> * Observer是一个对象, 这个对象拥有三个方法, 分别是next, complete, error

RxJS就是**一个核心三级重点**

* 一核心: Observable加上相关的Operators(map, filter...),其他的三个重点都是围绕这个核心转.
* 三个重点
    * Observer
    * Subject
    * Schedulers

### 建立一个Observable

> 基本的方法是通过create创建, 但是这不是一种常规的使用方式, 最常规的使用方式是使用creation operator像是from, of, fromEvent, fromPromise等.

```typescript
var observalbe = Rx.Observable.create(function(observer) {
  observer.next('这里传给订阅函数');
  observer.next('Jerry');
  observer.next('Anna');
})
// 通过订阅得到返回值
observable.subscribe(function(value) {
  console.log(value);
})
```

### Observer(观察者)

Observable可以被订阅(subscribe), 或者说被观察, 而订阅Observable对象的对象称为观察者(Observer)

### Creation Operator

> Observable有许多创建实例的方法, 称为creation Operator, 一下是常用的creation operator

* create

* of : 当我们需要**同步的**的传递几个值时, 可以使用`of`这个operator

    ```typescript
    var source = Rx.Observable.of('Jerry', 'Anna');
    source.subscribe({
      next: function(value) {
        console.log(value); // 依次输出Jerry, Anna
      }
    });
    ```

    

* from: 当我们想将一个数组的元素传递进去可以使用`from`这个operator

    ```typescript
    var arr = ['Jerry', 'Anna', 12, '30天'];
    var source = Rx.Observable.from(arr);
    source.subscribe({
      next: function(value) {
        console.log(value); // 依次输出数组内的数据
      }
    })
    ```

    >* 可列举的参数都可以传递, 因为ES6可列举的类型多了, 所以fromArray方法被移除
    >* 也可以传递字符串(string), 观察者将会按字节依次输出.
    >
    >* 可以传递一个Promise对象, 正常返回next, 异常发挥error, 也可以使用`fromPromise`会有相同的结果
    >
    >    ```typescript
    >    var source = Rx.Observable.from(new Promise(resolver, reject) => {
    >      setTimeout(() => {
    >        resolver('hello this is from promise');
    >      });
    >    });
    >    
    >    source.subscribe({
    >      next: function(value) {
    >        console.log(value); // 这里将会输出promise传出的值
    >      }
    >    });
    >    ```
    >
    >    

* fromEvent: 我们可以利用Event建立Observable, 使用`fromEvent`这个方法

    ```typescript
    var source = Rx.Observable.fromEvent(document.body, 'click');
    source.subscribe({
      next: function(value) {
        console.log(value);
      }
    });
    ```

    > 监听普通对象的时间调用的方法使用`fromEventPattern` 详情参见[网页](https://blog.jerry-hong.com/series/rxjs/thirty-days-RxJS-06/)

* fromPromise

* never: 返回一个无穷的Observable, 一直存在, 不作任何事

* empty: 返回一个空的Observable对象, 直接调用complete方法

* throw: 抛出一个错误, 调用error方法.

* interval: 定时器

    ```typescript
    var source = Rx.Observable.interval(1000);
    source.subscribe({
      next: function(value) {
        console.log(value);// 每隔一秒种输出秒数
      }
    })
    ```

* timer: 发令枪

    * 支持一个参数, 固定时间后执行一次结束
    * 支持两个参数, 固定时间(第一个参数)后开始执行, 后面每个固定时间(第二个参数)执行一次
    * 两个参数中的第一个参数可以支持一个Date格式的参数.

### Subscription

消除订阅可以调用subscription对象的unsubscribe方法

```typescript
var source = Rx.Observable.timer(1000, 1000);
var subscription = source.subscribe({
	next: function(value) {
		console.log(value);
	}
});

setTimeout(() => {
  subscription.unsubscribe();
}, 5000);
```

### Operator

* 每一个Operator返回一个新的Observable
* 我们可以透过`create`的方法建立各种operator

### Marble Diagrams

> Marble Diagrams 相關資源：http://rxmarbles.com/

用来表示并行执行的程序的方法

### Transformation Operators

* map: 对每一个值进行相应操作

    ```javascript
    var source = Rx.Observable.interval(1000);
    var newest = source.map(x => x + 1);
    
    newest.subscribe(console.log); ---1---2---3---4
    ```

* mapTo: 替换每一个值为固定值

    ```javascript
    var source Rx.Observable.interval(1000);
    var newest = source.mapTo(2);
    
    newest.subscribe(console.log); ---2---2---2---2---2
    ```

* filter: 过滤符合条件的对象

    ```javascript
    var source = Rx.Observable.interval(1000);
    var newest = source.filter(x => x % 2 === 0);
    newest.subscribe(console.log); ---0---2---4---6
    ```

* take: 获取前几个元素后结束

    ```typescript
    var source = Rx.Observable.interval(1000);
    var example = source.take(3);
    example.subscribe(console.log); ---0---1---2|
    ```

* first: 获取送出的第一个元素后结束

    ```typescript
    var source = Rx.Observable.interval(1000);
    var example = source.first();
    example.subscribe(console.log);---0|
    ```

* takeUntil: 在某事件发生前, 让一个observable直接送出完成

    ```typescript
    var source = Rx.Observable.interval(1000);
    var click = Rx.Observable.fromEvent(document.body, 'click');
    var example = source.takeUntil(click);
    example.subscribe(console.log); ---0---1---2-[click]-|
    ```

* concatAll: 返回的每一个元素本身是一个Observable对象, 可以摊平进行操作

    ```typescript
    var click = Rx.Observable.fromEvent(document.body, 'click');
    var source = click.map( e => Rx.Observable.of(1, 2, 3));
var example = source.concatAll();
    example.subscribe(console.log); ---(123)---(123)---
    ```
    
    > 这里需要注意的一点, source发送的三个Observable对象需要一个执行完后执行下一个.
    
* 简单的拖拉操作

    ```typescript
    const dragDom = document.getElementById('drag');
    const body = document.body;
    
    const mouseDown = Rx.Observable.fromEvent(dragDom, 'mousedown');
    const mouseUp = Rx.Observable.fromEvent(body, 'mouseup');
    const mouseMove = Rx.Observable.fromEvent(body, 'mousemove');
    
    mouseDown.map(event => mouseMove.takeUntil(mouseUp))
    	.concatAll()
    	.map(event => ({x: event.clientX, y: event.clientY}))
      .subscribe(pos => {
        dragDom.style.left = pos.x + 'px';
        dragDom.style.top = pos.y + 'px';
      })
    ```

* skip: 略过前几个送出的元素的operator

    ```typescript
    var source = Rx.Observable.interval(1000);
    var example = source.skip(3);
    example.subscribe(console.log); ---2---3---4---
    ```

* takeLast: 取最后几个operator

    ```typescript
    var source = Rx.Observable.interval(1000).take(6);
    var example = source.takeLast(3);
    example.subscribe(console.log); ---------(345)|
    ```

    > 这里需要注意的一点是, takeLast只有等到所有的Observable执行完毕才知道最后的两个是什么, 然后最后一次性返回

* last: take(1) => first;  takeLast(1) => last();

* concat: 把多个Observable对象合并成一个

    ```typescript
    var source = Rx.Observable.interval(1000).take(3);
    var source2 = Rx.Observable.of(3)
    var source3 = Rx.Observable.of(4,5,6)
    var example = source.concat(source2, source3);
    
    example.subscribe(console.log); ---0---1---2(345)|
    ```

    > 跟concatAll一样, 要等前一个Observable执行完毕后执行下一个Observable
    >
    > concat也可以用作静态方法的使用

* startWith: 可以在Observable执行前先输出什么值, 这里优点像concat, 但这里参数不是Observable, 而是一个具体要发送的元素

    ```typescript
    var source = Rx.Observable.interval(1000);
    var example = source.startWith(0);
    
    example.subscribe(console.log); // (0)---0---1---2--....
    ```

* merge: 同concat用于合并Observable, 但行为上不同, merge允许事件同事执行.

    ```typescript
    var source = Rx.Observable.interval(500).take(3);
    var source2 = Rx.Observable.interval(300).take(6);
    var example = source.merge(source2);
    
    example.subscribe(console.log);
    
    source : ----0----1----2|
    source2: --0--1--2--3--4--5|
                merge()
    example: --0-01--21-3--(24)--5|
    ```

    > merge的操作如果OR, 只要有一个执行, 则触发. merge可以用于同步操作.

* combineLatest: 获取到各个Observable最后送出的值, 再输出成一个值

    ```typescript
    var source = Rx.Observable.interval(500).take(3);
    var newest = Rx.Observable.interval(300).take(6);
    
    var example = source.combineLatest(newest, (x, y) => x + y);
    
    example.subscribe(console.log);
    
    source : ----0----1----2|
    newest : --0--1--2--3--4--5|
    
        combineLatest(newest, (x, y) => x + y);
    
    example: ----01--23-4--(56)--7|
    ```

    > 可以使用在多变因子的计算结果上, 如果其中一个变动,则触发此方法, 进行重新计算

* zip: 会取每个Observable相同顺序的元素并传入callback, 也就是每一个observable的第n个元素会放在一起执行callback

    ```typescript
    var source = Rx.Observable.interval(500).take(3);
    var newest = Rx.Observable.interval(300).take(6);
    
    var example = source.zip(newest, (x, y) => x + y);
    
    example.subscribe(console.log);
    
    
    source : ----0----1----2|
    newest : --0--1--2--3--4--5|
        zip(newest, (x, y) => x + y)
    example: ----0----2----4|
    ```

    > * 经常用于输出一些demo使用
    >
    >     ```typescript
    >     var source = Rx.Observable.from('hello');
    >     var source2 = Rx.Observable.interval(100);
    >     
    >     var example = source.zip(source2, (x, y) => x);
    >     
    >     source : (hello)|
    >     source2: -0-1-2-3-4-...
    >             zip(source2, (x, y) => x)
    >     example: -h-e-l-l-o|
    >     ```
    >
    > * 平时不要乱使用zip, 因为如果一个快, 一个慢的情况下会一直缓存元素.

* withLatestFrom: 运作同conbineLatest, 但他有一个主要的Observable, 只有主要的Observable送出值时, 才会执行callback.

    ```typescript
    var main = Rx.Observable.from('hello').zip(Rx.Observable.interval(500), (x, y) => x);
    var some = Rx.Observable.from([0,1,0,0,0,1]).zip(Rx.Observable.interval(300), (x, y) => x);
    
    var example = main.withLatestFrom(some, (x, y) => {
        return y === 1 ? x.toUpperCase() : x;
    });
    
    example.subscribe(console.log);
    
    
    main   : ----h----e----l----l----o|
    some   : --0--1--0--0--0--1|
    
    withLatestFrom(some, (x, y) =>  y === 1 ? x.toUpperCase() : x);
    
    example: ----h----e----l----L----O|
    ```

    > * main 送出值时, 如果some没有输出任何值, 也是不会执行的
    > * main送出值时, 回去判断some最后一次输出的值, 执行callback方法

* scan: Observable 版本的reduce

    ```typescript
    // reduce 演示 返回: 回传阵列
    var arr = [1, 2, 3, 4];
    var result = arr.reduce((origin, next) => { 
        console.log(origin)
        return origin + next
    }, 0);
    
    console.log(result)
    // 0
    // 1
    // 3
    // 6
    // 10
    
    // scan 演示
    var source = Rx.Observable.from('hello')
                 .zip(Rx.Observable.interval(600), (x, y) => x);
    
    var example = source.scan((origin, next) => origin + next, '');
    
    example.subscribe(console.log);
    
    source : ----h----e----l----l----o|
        scan((origin, next) => origin + next, '')
    example: ----h----(he)----(hel)----(hell)----(hello)|
    ```

* buffer: buffer是一个家族, 总共有五个operators

    * buffer `常用`
    * bufferCount`常用`
    * bufferTime`常用`
    * bufferToggle
    * bufferWhen

    ```typescript
    var source = Rx.Observable.interval(300);
    var source2 = Rx.Observable.interval(1000);
    var example = source.buffer(source2);
    //  example = source.bufferTime(1000);
    // var example = source.bufferCount(3);
    
    example.subscribe(console.log);
    
    
    source : --0--1--2--3--4--5--6--7..
    source2: ---------0---------1--------...
                buffer(source2)
    example: ---------([0,1,2])---------([3,4,5])   
    ```

    > 实际使用中, 我们可以使用buffer来做某一个事件的过滤, 比如某一个事件必须在固定时间内鼠标点击两次后才能执行

* delay: 可以延迟observable一开始发送元素的时间点.

    ```typescript
    var source = Rx.Observable.interval(300).take(5);
    
    var example = source.delay(500);
    
    example.subscribe(console.log);
    
    
    source : --0--1--2--3--4|
            delay(500)
    example: -------0--1--2--3--4|
    ```

* delayWhen: 与delay差不多, 但是可以影响每一个元素的发送时间

    ```typescript
    var source = Rx.Observable.interval(300).take(5);
    
    var example = source.delayWhen(
                      x => Rx.Observable.empty().delay(100 * x * x)
                  );
    
    example.subscribe(console.log);
    
    
    source : --0--1--2--3--4|
        .delayWhen(x => Rx.Observable.empty().delay(100 * x * x));
    example: --0---1----2-----3-----4|
    ```

* debounce:跟buffer, bufferTime一样, Rx有debounce, debounceTime,一个传observable, 一个传入的是毫秒数.

    ```typescript
    var source = Rx.Observable.interval(300).take(5);
    var example = source.debounceTime(1000);
    
    example.subscribe(conosle.log);
    
    source : --0--1--2--3--4|
            debounceTime(1000)
    example: --------------4|  
    ```

    > debounce运作方式是每次收到的元素, 他会首先把元素缓存住, 如果在固定时间收到了其他元素, 则清除之前的数据, 重新计时, 否则, 执行后续操作.
    >
    > 常用于防止事件抖动.(输入框验证是否合法的场景)

* throttle: 跟debounce一样, throttle也有throttleTime方法, 分别传入observable和时间毫秒数. 跟debounce不一样的是throttle会先放出元素, 然后等待一段默认的时间. 等到时间过了, 会再放出元素.

    > 防止事件抖动, (鼠标移动, 控制事件的频率场景)
    >
    > * throttle像是控制最高抖动频率
    > * debounce则是必须等待的时间

    ```typescript
    var source = Rx.Observable.interval(300).take(5);
    var example = source.throttleTime(1000);
    
    example.subscribe(console.log);
    
    // 0
    // 4
    // complete
    ```

* distinct: 过滤相同值, 只留一条记录

    ```typescript
    var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'b'])
                .zip(Rx.Observable.interval(300), (x, y) => x);
    var example = source.distinct()
    
    example.subscribe(console.log);
    
    source : --a--b--c--a--b|
                distinct()
    example: --a--b--c------|
      
    // 传入一个selector callback function 传入我们需要对比的属性
    var source = Rx.Observable.from([{ value: 'a'}, { value: 'b' }, { value: 'c' }, { value: 'a' }, { value: 'c' }])
                .zip(Rx.Observable.interval(300), (x, y) => x);
    var example = source.distinct((x) => {
        return x.value
    });
    
    example.subscribe(console.log);
    // {value: "a"}
    // {value: "b"}
    // {value: "c"}
    // complete
    
    // 传入第二个参数, 用来刷新缓存的set数据
    var source = Rx.Observable.from(['a', 'b', 'c', 'a', 'c'])
                .zip(Rx.Observable.interval(300), (x, y) => x);
    var flushes = Rx.Observable.interval(1300);
    var example = source.distinct(null, flushes);
    
    example.subscribe(console.log);
    
    source : --a--b--c--a--c|
    flushes: ------------0---...
            distinct(null, flushes);
    example: --a--b--c-----c|
    ```

* distinctUntilChanged: 过滤相同的数据, 但是只会跟最后发出的元素进行对比

    ```typescript
    var source = Rx.Observable.from(['a', 'b', 'c', 'c', 'b'])
                .zip(Rx.Observable.interval(300), (x, y) => x);
    var example = source.distinctUntilChanged()
    
    example.subscribe(console.log);
    
    // a
    // b
    // c
    // b
    // complete
    ```

* catch: RxJS中可以用catch直接处理错误, 也可以回传一个observable来送出新的值

    ```typescript
    var source = Rx.Observable.from(['a','b','c','d',2])
                .zip(Rx.Observable.interval(500), (x,y) => x);
    
    var example = source
                    .map(x => x.toUpperCase())
                    .catch(error => Rx.Observable.of('h'));
    								// 也可以在遇到错误时结束
    								// .catch(error => Rx.Observable.empty());
    
    example.subscribe(console.log);
    
    source : ----a----b----c----d----2|
            map(x => x.toUpperCase())
             ----a----b----c----d----X|
            catch(error => Rx.Observable.of('h'))
    example: ----a----b----c----d----h|      
    ```

    * catch的callback可以接受第二个参数, 这个参数接受当前的observable, 我们可以回传当前的observable来重新执行

        ```typescript
        var source = Rx.Observable.from(['a','b','c','d',2])
                    .zip(Rx.Observable.interval(500), (x,y) => x);
        
        var example = source
                        .map(x => x.toUpperCase())
                        .catch((error, obs) => obs);
        
        example.subscribe(console.log);
        
        source : ----a----b----c----d----2|
                map(x => x.toUpperCase())
                 ----a----b----c----d----X|
                catch((error, obs) => obs)
        example: ----a----b----c----d--------a----b----c----d--..
        ```

* retry: 我们可以把上面接收错误obserable并回传重新执行的逻辑简化为retry

    ```typescript
    var source = Rx.Observable.from(['a','b','c','d',2])
                .zip(Rx.Observable.interval(500), (x,y) => x);
    
    var example = source
                    .map(x => x.toUpperCase())
                    .retry();
    								// 也可以设置重试次数, 不设置会一直重试
    								// .retry(1);
    
    example.subscribe(console.log);
    
    
    ```

    > 这里我们可以使用在HTTP request失败的场景

* retryWhen: 可以将返回的错误封装成一个observable对象, 并对这个异常的observable对象进行相应的处理, 待这个observable处理完毕时, 再执行后续的原生observable对象的相应逻辑.

    ```typescript
    var source = Rx.Observable.from(['a','b','c','d',2])
                .zip(Rx.Observable.interval(500), (x,y) => x);
    
    var example = source
                    .map(x => x.toUpperCase())
                    .retryWhen(errorObs => errorObs.delay(1000));
    								// 实际使用中的错误通知的处理
    								// .retryWhen(errorObs => errorObs.map(err => fetch('...')));
    
    example.subscribe(console.log);
    
    
    source : ----a----b----c----d----2|
            map(x => x.toUpperCase())
             ----a----b----c----d----X|
            retryWhen(errorObs => errorObs.delay(1000))
    example: ----a----b----c----d-------------------a----b----c----d----...
    ```

    > * 实际使用中, 我们经常使用catch来做订阅延迟.
    >
    > * retryWhen, 我们经常用来处理错误通知

* repeaat: 有时候我们需要在没有报错的情况下也要重复执行, 可以使用repeat

    ```typescript
    var source = Rx.Observable.from(['a','b','c'])
                .zip(Rx.Observable.interval(500), (x,y) => x);
    
    var example = source.repeat(2);
    // 我们可以不给参数让他无限重复下去
    // var example = source.repeat();
    
    example.subscribe(console.log);
    
    
    source : ----a----b----c|
                repeat(2)
    example: ----a----b----c----a----b----c|
    ```

* 摊平操作

    * concatAll: 需要处理的observable需要一个一个依次处理

    * switch: 如果有新的observable传出, 则退订前面的observable, 总是以最新的observable执行

        ```typescript
        var click = Rx.Observable.fromEvent(document.body, 'click');
        var source = click.map(e => Rx.Observable.interval(1000));
        
        var example = source.switch();
        example.subscribe(console.log);
        
        
        click  : ---------c-c------------------c--.. 
                map(e => Rx.Observable.interval(1000))
        source : ---------o-o------------------o--..
                           \ \                  \----0----1--...
                            \ ----0----1----2----3----4--...
                             ----0----1----2----3----4--...
                             switch()
        example: -----------------0----1----2--------0----1--...
        ```

    * mergeAll: 执行所有的observable, 合并执行

        ```typscript
        var click = Rx.Observable.fromEvent(document.body, 'click');
        var source = click.map(e => Rx.Observable.interval(1000));
        
        var example = source.mergeAll();
        example.subscribe(console.log);
        
        
        click  : ---------c-c------------------c--.. 
                map(e => Rx.Observable.interval(1000))
        source : ---------o-o------------------o--..
                           \ \                  \----0----1--...
                            \ ----0----1----2----3----4--...
                             ----0----1----2----3----4--...
                             switch()
        example: ----------------00---11---22---33---(04)4--...
        ```

        > mergeAll可以传入一个数字参数, 用来代表可以同事处理observable的数量, 如果传入的是"1", 那么就跟concatAll一个效果.

* concatMap: 其实就是map加上concatAll的简单写法

    ```typescript
    var source = Rx.Observable.fromEvent(document.body, 'click');
    
    var example = source
                    .map(e => Rx.Observable.interval(1000).take(3))
                    .concatAll();
    // 简单写法
    // var example = source
    //               .concatMap(
    //                    e => Rx.Observable.interval(100).take(3)
    //                );
                    
    example.subscribe(console.log);
    
    source : -----------c--c------------------...
            concatMap(c => Rx.Observable.interval(100).take(3))
    example: -------------0-1-2-0-1-2---------...
    ```

    > * 优化Request请求, 当需要顺序执行数据加载时使用
    >
    > * 第二个参数是一个selector callback, 这个callback有四个参数
    >
    >     * 外部observable发出的元素
    >     * 内部observable发出的的元素
    >     * 外部observable发出的元素的index
    >     * 内部observable发出的元素的index
    >
    >     ```typescript
    >     function getPostData() {
    >         return fetch('https://jsonplaceholder.typicode.com/posts/1')
    >         .then(res => res.json())
    >     }
    >     var source = Rx.Observable.fromEvent(document.body, 'click');
    >     
    >     var example = source.concatMap(
    >                     e => Rx.Observable.from(getPostData()), 
    >                     (e, res, eIndex, resIndex) => res.title);
    >     
    >     example.subscribe(console.log);
    >     ```

* switchMap: 实际是一个map, 加上switch简化写法

    ```typescript
    var source = Rx.Observable.fromEvent(document.body, 'click');
    
    var example = source
                    .map(e => Rx.Observable.interval(1000).take(3))
                    .switch();
    // 简化写法
    // var example = source
    //                .switchMap(
    //                  e => Rx.Observable.interval(100).take(3)
    //                );
    example.subscribe(console.log);
    
    
    source : -----------c--c-----------------...
            concatMap(c => Rx.Observable.interval(100).take(3))
    example: -------------0--0-1-2-----------...
    ```

    > * 可以优化Request请求: 连续点击发送相同的请求需要过滤的情况
    > * 有第二个参数selector callback 同上

* mergeMap: 实际是一个map加上mergeAll的简化写法

    ```typescript
    var source = Rx.Observable.fromEvent(document.body, 'click');
    
    var example = source
                    .map(e => Rx.Observable.interval(1000).take(3))
                    .mergeAll();
    // 简单的写法
    // var example = source
    //                .mergeMap(
    //                    e => Rx.Observable.interval(100).take(3)
    //                );
                    
    example.subscribe(console.log);
    
    
    source : -----------c-c------------------...
            concatMap(c => Rx.Observable.interval(100).take(3))
    example: -------------0-(10)-(21)-2----------...
    ```

    > * 优化Request HTTP请求, 并行加载无关资源的情况.
    > * 存在第二个参数selector callback 同上
    > * 可以传递第三个参数, 一个数字, 用来设置并行处理量

* switchMap, mergeMap, concatMap这三个的共同特征: 可以自动转换Promise对象为Observable, 这样我们就不用Observable.from转一次了

    ```typescript
    function getPersonData() {
        return fetch('https://jsonplaceholder.typicode.com/posts/1')
        .then(res => res.json())
    }
    var source = Rx.Observable.fromEvent(document.body, 'click');
    
    var example = source.concatMap(e => getPersonData());
                                        //直接回傳 promise 物件
    
    example.subscribe(console.log);
    ```

* window: window是一个家族, 总共有五个operators

    > 参见[链接]()

    * window
    * windowTime
    * windowCount
    * windowToggle
    * windowWhen

* groupBy: 合并相同的条件

    ```typescript
    var source = Rx.Observable.interval(300).take(5);
    
    var example = source
                  .groupBy(x => x % 2);
                  
    example.subscribe(console.log);
    
    // GroupObservable { key: 0, ...}
    // GroupObservable { key: 1, ...}
    
    
    source : ---0---1---2---3---4|
                 groupBy(x => x % 2)
    example: ---o---o------------|
                \   \
                \   1-------3----|
                0-------2-------4|
    ```

### Observable运作方式

* 延迟运算
* 渐进式取值

### Subject

* subject同时是Observable, 又是Observer
* Subject会对内部的observers清单进行组播(multicast)

```typescript
var source = Rx.Observable.interval(1000).take(3);

var observerA = {
    next: value => console.log('A next: ' + value),
    error: error => console.log('A error: ' + error),
    complete: () => console.log('A complete!')
}

var observerB = {
    next: value => console.log('B next: ' + value),
    error: error => console.log('B error: ' + error),
    complete: () => console.log('B complete!')
}

var subject = new Rx.Subject()

subject.subscribe(observerA)

source.subscribe(subject);

setTimeout(() => {
    subject.subscribe(observerB);
}, 1000);

// "A next: 0"
// "A next: 1"
// "B next: 1"
// "A next: 2"
// "B next: 2"
// "A complete!"
// "B complete!"
```

23节

