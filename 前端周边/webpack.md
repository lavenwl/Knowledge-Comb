[官方中文文档](https://www.webpackjs.com/concepts/)



`webpack` 是一个现代 `JavaScript` 应用程序的静态模块打包器，当 `webpack` 处理应用程序时，会递归构建一个依赖关系图，其中包含应用程序需要的每个模块，然后将这些模块打包成一个或多个 `bundle`。

[带阅读](https://segmentfault.com/a/1190000021953371)

### 核心概念

* 入口(entry): 构建依赖图的开始
* 输出(output): 在哪里构建bundles, 以及如何命名这些文件, 默认./dist
* loader: 处理非javascript的文件.
* plugins: 插件的应用范围比loader更加广泛. loader应用在转换某些模块. 插件可应用在打包优化和压缩, 定义环境中的变量.

### 如何使用loader

1. loader的基本格式

    ```json
    {
      test: xxx,
      use: xxx,
      options: {}
    }
    
    // test: 需要转换的文件们地址
    // use: 使用哪一个loader
    // options: 单独的配置
    
    // use属性的三种写法:
    //	* 字符串 use: 'babel-loader'
    //  * 对象   use: {loader: 'babel-loader', options: {}}
    //  * 数组   use: ['babel-loader']
    ```

2. 实战使用babel-loader, 用来将js转换为低版本的代码

    ```
    # babel-loader@8.1.0
    npm install --save-dev babel-loader
    
    # @babel/core：babel的核心，包含所有的核心 API
    # @babel/preset-env：将 js 引入的新语法转换为 ES5 的语法（不包括新增的全局变量、方法等）
    # @babel/plugin-transform-runtime：用于构建过程中的代码转换
    npm install --save-dev @babel/core @babel/preset-env @babel/plugin-transform-runtime
    
    # @babel/runtime：实际导入项目代码的功能模块
    # @babel/runtime-corejs3：配合 @babel/plugin-transform-runtime 避免全局污染
    npm install --save @babel/runtime @babel/runtime-corejs3
    ```

    