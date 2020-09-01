# Dubbo中的扩展点功能特性

### dubbo中的扩展点分为三种

##### 指定名字的扩展点

```java
RandomLoadBalance rd= (RandomLoadBalance)ExtensionLoader.getExtensionLoader(Loadbalance.class).getExtension("random");
```

##### 自适应扩展点

```java
ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
```

##### 激活扩展点

```java
ExtensionLoader.getExtensionLoader(Protocol.class).getActiveExtension
```



