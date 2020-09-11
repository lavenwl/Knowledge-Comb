### SPI(Service Provider Interface)

> Spring SPI 功能基于Java的SPI功能实现

Java 的SPI需要满足下面的条件:

* 需要在classpath目录下创建一个META-INF/services目录
* 在该目录下创建一个扩展点全路径的文件
* 在此文件中填写这个扩展点的实现(文件编写格式为UTF-8)
* 使用ServiceLoader进行加载

Spring的SPI需要满足下面的条件:

* 需要在classpath目录下创建一个META-INF/spring.factory
* SpringFactoryLoader进行加载