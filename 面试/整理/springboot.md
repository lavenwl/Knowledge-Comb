### springboot如何继承现有项目

springboot在继承其他项目时使用starter方式. 如果自己的项目没有springboot的starter, 那么可以自己写一个starter, starter的意义是可以在springboot启动的时候利用springboot的自动装配机制, 加载相关项目相关类到springIOC容器内部, 便于后面的spring容器内部操作.



