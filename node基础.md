# node基础

## 1.node文件加载顺序

> 在node使用的CommonJS中 `require('xx')` 是我们最常用的引入命令 

1、加载都是从缓存开始的

​	在node中会将之前加载过的包进行缓存，因为从缓存中读取速度是最快的。

​	在缓存中 **核心缓存**的搜索优先级 > **文件缓存**

2、当缓存找不到时才会去加载模块

​	对于模块间也有优先级 核心模块 > 文件模块 > 自定义模块，我们来看看为什么

| 核心模块                                                     | 文件模块                                                   | 自定义模块                                                   |
| ------------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| 二进制文件，直接加载入内存，一般是c++和部分js代码编译后得到的二进制文件 | 按照路径找具体文件速度较快，比如`require(../src/index.ts)` | 会按照预设的路径去搜索很耗时间，例如['/widgets/app/node_modules'<br>,'/widgets/node_modules',<br>'/widgets/node_modules'] |

![node文件加载顺序](https://raw.githubusercontent.com/caifeng123/pictures/master/node%E6%96%87%E4%BB%B6%E5%8A%A0%E8%BD%BD%E9%A1%BA%E5%BA%8F.png)

## 2.编译-文件模块

1、按文件名读取

```
.js - 通过fs模块读取后执行
.node - 通过dlopen() 方法加载c++编译后的文件 [因此只需要加载与执行无需编译]、
.json - 通过fs读取，JSON.parse() 解析
其他文件 - 当做.js执行
```

2、CommonJS中的文件

> 对于CommonJS往往会有require、exports、module、__filename、dirname5个变量，但却没定义，从何而来？
>
> 并且要是每个文件的变量定义相同，为什么不会出现相互污染变量值的情况？

事实上，在编译过程中，node对js文件进行头尾包装

```js
(function (exports, require, module, __filename, __dirname){
	const math = require('math');
	exports.random = () => math.random();
  // 或者
  module.exports = {
    random: () => math.random();
  }
})
```

> 此时又有疑问，为什么要exports和module两个导出方式呢？

实际上 exports === module.exports 的,那肯定有小伙伴觉得下面是一样的

```js

(function (exports, require, module, __filename, __dirname){
	const math = require('math');
  exports = {
    random: () => math.random();
  }
  module.exports = {
    random: () => math.random();
  }
})
```

其实不然，传入exports是作为形参传入，你将整个对象进行赋值那么将原先地址指向都改变了，因此要么改变exports内的属性，要么将module的exports替换



3.`.node`文件主要用在加载执行c++代码（已被编译）

- 先将JS核心模块转为字符数组存储在c/c++代码中，由于是js代码 未编译，不能直接执行。当启动node后，将js核心模块代码加载入内存中
- lib下的js文件，也经过头尾包装，向外exports

| 区别     | JS核心模块                              | Js普通模块           |
| -------- | --------------------------------------- | -------------------- |
| 引入方式 | process.binding('natives')              | require('xxx')       |
| 存放位置 | ./lib                                   | 自定义位置 例如./src |
| 被引用者 | 核心模块提供基本api给其他js上层模块使用 | 正常代码使用         |
| 缓存位置 | NativeModule._cache                     | Module._cache        |

## 3、C/C++扩展模块

> 对于JS的位运算很弱（js仿照java实现，对于java位运算是int型操作，js数字默认是double，需要先进行一次转换）
>
> 此时使用C/C++模块能快速解决问题
>
> 1、能够兼容C/C++语言实现
>
> 2、C/C++效率高于js



## 4、前后端环境 & commonJS, AMD, CMD

> commonJS在设计之初就是为了想要在任何地方都能运行，虽然大部分也确实多亏了前后端环境的类似，大部分模块共用。但在部分情况下还是会有差异，例如http模块

对于node来说，上述有提到 其实require('xxx') 就是相当于头尾包裹后 加载入文件一个封闭方法。明显是同步的加载

前端，模块不可能使用同步加载的方式，否则会影响用户体验甚至会让页面长时间卡顿与白屏

> 最终 AMD胜出 全称是Asynchronous module definition

```js
// AMD 写法
// 需要将依赖项写入第一项，再以参数引入
// 最终要return 出exports对象
define(['dp1', 'dp2'], function(dp1, dp2){
	var exports = {};
	exports.hello = function(){
		// do something
	}
	return exports;
});

// CMD 写法
// 接近commonjs写法
define(function(require, exports, module){
	// just like commonjs environment
});
// 需要导入时 直接require('xx')即可
```

> 为了兼容多个规范和环境，就出现了下面的方法

```js
(function(name, definition) {
	// 检测是否为amd/cmd
  const hasDefine = typeof define === 'function';
  const hasExports = typeof module !== 'undefined' && module.exports;
  if (hasDefine) {
  	// amd/cmd
    define(definition);
  } else if (hasExports) {
    // node 模块
    module.exports = definition();
  } else {
  	// 说明在浏览器环境中 有bom
    this[name] = definition();
  }
})('hi',function(){
  const hi = function(){};
	return hi;
})
```



