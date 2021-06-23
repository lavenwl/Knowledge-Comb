### assert 关键字

> 具体参考[网页](https://blog.51cto.com/lavasoft/43735)

1. assert关键字是适用于开发过程中的测试断点, 不建议生产使用.
2. assert语句需要开启-ea参数启动有效, 否则没有效果.详单与注释.

> 注意: Spring自带的类Assert可以使用在代码中的判空功能上

### StringBuilder与StringBuffer的区别

1. String的拼接是重新构建新的对象
2. StringBuffer与StringBuilder的拼接不用构建新的对象
3. StringBuffer是线程安全的, StringBuilder是线程不安全的
4. StringBuilder的性能比StringBuffer要快

