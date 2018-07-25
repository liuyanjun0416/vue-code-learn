# 第一章

第一部分首先来分析下vue的目录结构以及整体的一个架构和进行入口的分析

### 一、目录

首先vue的源码部分从src目录入手

其目录依次是

* compiler
* core
* platforms
* server
* sfc
* shared

1. compiler  编译器，这个目录内主要为vue中的templete进行解析，根据对应生成AST，然后针对不同的格式进行解析，然后在转成对应的代码
2. core vue的核心代码，主要包括vdom、components、global-api、observer、instance、util。
3. platforms 其中包括web、weex。针对不同平台的一些特殊处理
4. server 服务端渲染的主要逻辑
5. sfc 处理单组件功能
6. shared 公共方法逻辑变量

### 二、入口

入口的话分为两种一种是代码的入口，一种是编译的入口。

首先看下package.json中的scripts build，编译是使用rollup打包工具进行编译的，我们以vue.js打包后的结果为入口，其打包的配置如下

```javascript
'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  }
```

里面一个是入口为web/entry-runtime-with-compiler.js，还有一个alias，he是一个entity-decoder

所以进一步分析entry-runtime-with-compiler，里面做了一个事情，再封装了Vue.prototype.$mount的方法，在原来$mount的方法的基础上加了一些提示解析了template/el 并且转换成render方法\( resolve template/el and convert to render function\)，这里的解析就依赖于上面所说的编译器\(compiler\)，将templete转成成render方法去渲染。



