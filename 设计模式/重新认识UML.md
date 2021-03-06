### 什么是UML

统一建模语言（Unified Modeling Language, UML）是一种为面向对象系统的产品进行说明， 可视化和编制文档的一种标准语言，是非专利的第三代建模和规约语言。

### 事务关系的描述

* 泛华关系（Generalization）：表示一个更泛华的元素和一个更具体的元素之间的关系，同继承，用带三角的实现表示
* 实现关系（Realization）：类与接口的关系，用带三角形的虚线表示，箭头指向父类接口
* 组合关系（Combination）：整体与部分的关系，组合比聚合严格，当某个实体组合成另一个实体时，二者具有相同的生命周期，用带有实心菱形的实现表示，菱形指向整体。
* 聚合关系（Aggregation）：是整体与部分的关系，用空心菱形的实线表示：菱形指向整体
* 关联关系（Association）：是一种拥有的关系，具有方向性，用带箭头的实线表示
* 依赖关系（Dependency）：如果一个类的改动会影响到另外一个类， 则两个类是依赖关系，一般是单向的，用带箭头的虚线表示

```mermaid
classDiagram
classA --|> classB : Inheritance
classC --* classD : Composition
classE --o classF : Aggregation
classG --> classH : Association
classI -- classJ : Link(Solid)
classK ..> classL : Dependency
classM ..|> classN : Realization
classO .. classP : Link(Dashed)
```

![image-20200314091133941](重新认识UML.assets/image-20200314091133941.png)

### 类图（Class Diagrams）

类图用三个矩形表示，最上面部分标识类的名称；中间的部分标识类的属性；最下面的部分标识类的方法

1. “+” 表示 public
2. “-” 表示 private
3. “\#” 表示 protect
4. “~” 表示 default，可省略不写
5. 字段和方法返回值非必须
6. 抽象类或抽象方法为斜体表示
7. 静态类或静态方法加下划线
8. 如果是接口在类名上方加&lt;&lt;interface&gt;&gt;

### 时序图

时序图描述的是对象之间的消息发送顺序，强调时间顺序。时序图是一个二维图， 横轴表示对象，纵轴表示时间，消息在对象之间横向传递，依照时间顺序纵向排列，用箭头表示消息，用竖虚线表示生命线



