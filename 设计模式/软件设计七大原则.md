### 开闭原则

开闭原则（Open-Closed Principle, OCP）是指一个软件实体，如类，模块，函数应该对扩展开放，对修改关闭。

### 依赖倒置原则

依赖倒置原则（Dependence Inversion Principle, DIP）是指设计代码结构时，高层模块不应该依赖底层模块，二者都应该依赖其抽象。抽象不应该依赖细节，细节应该依赖抽象。

### 单一职责原则

单一职责原则（Simple Responsibility Principle, SRP）是指不要存在多于一个导致类变更的原因。

### 接口隔离原则

接口隔离原则（Interface Segregation Principle, ISP）是指用多个专门的接口，而不使用单一的总接口，客户端不应该依赖它不需要的接口。

### 迪米特法则

迪米特法则（Law of Demeter LoD）是指一个对象应该对其他对象保持最少的了解，又叫最少知道原则（Least Knowledge Principle, LKP）尽量降低类与类之间的耦合。

### 里氏替换原则

里氏替换原则（Liskov Substitution Principle, LSP）是指如果对每一个类型为T的对象o,都有类型为T2的对象o2，使得以T定义的所有程序P在所有对象o都替换成o2时，程序P的行为没有发生变化，那么类型T2是类型T1的子类型。（子类可以扩展父类的功能， 但不能改变父类原有的功能）

### 合成复用原则

合成复用原则（Composite/Aggregate Reuse Principle, CARR）是指尽量是用对象组合（has-a）/ 聚合（contanis-a）而不是继承关系达到软件复用的目的。



