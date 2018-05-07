## 模块化
模块就是实现特定功能的一组方法。

1.简单模块化形式
函数。

缺点：全局污染，命名冲突

字面量
缺点：暴露了模块成员，外部都可以改。

立即执行函数，返回外部调用的对象。

放大模式传入参数（jquery），


2.模块规范
加载模块有两种异步加载和同步加载，或动态加载和静态加载。

AMD，CMD，closure，CommonJS，ES6
## commonjs
nodejs模块系统参照commonjs，它是服务端js编程。服务端require加载js是同步的，而浏览器环境加载js不适合同步。
所有的模块都存放在本地硬盘，可以同步加载完成，等待时间就是硬盘的读取时间。但是，对于浏览器，这却是一个大问题，
因为模块都放在服务器端，等待时间取决于网速的快慢，可能要等很长时间，浏览器处于"假死"状态。
静态加载，
Commonjs 输出的是一个值的拷贝，一旦输出一个值，模块内部的变化影响不到这个值。

循环加载：
commonjs一个模块就是一个脚本文件。require命令第一次加载该脚本就会执行整个脚本，然后内存中生成一个对象。
{
id:'',
exports:'',
loaded:true
....
}
可参看webpack，需要用到这个模块时，从exports属性上取值。
commonjs重要特性：加载时执行。
如果模块被“循环加载”，就只输出已经执行的部分，未执行的部分不会输出。如果A引用了B，B又引用了A，另一个文件先加载A再加载B，A先输出部分，遇到
加载B，就去加载B，然后执行B时遇到A，因为A还没有执行完，exports只返回部分值，此时A不执行，还是B继续执行，执行完成后B将执行权返回给A，A继续执行完毕。



webpack:

```javascript
(function(modules) { // webpackBootstrap
     // The module cache 模块缓存对象
      var installedModules = {};
      // The require function require函数
     function __webpack_require__(moduleId) {
 
      // Check if module is in cache 检查模块是否已缓存，如果缓存直接exports导出模块外暴内容（函数等）
         if(installedModules[moduleId]) {
             return installedModules[moduleId].exports;
         }
        // Create a new module (and put it into the cache) 创建一个新的模块然后存入缓存，
         var module = installedModules[moduleId] = {
             i: moduleId,
            l: false,
             exports: {}
         };

        // Execute the module function 执行模块函数，会将模块外暴的方法等挂到module的.exports.
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

        // Flag the module as loaded
       module.l = true;

         // Return the exports of the module
         return module.exports;
    }

    // expose the modules object (__webpack_modules__)
    __webpack_require__.m = modules;

    // expose the module cache
    __webpack_require__.c = installedModules;

     // define getter function for harmony exports
    __webpack_require__.d = function(exports, name, getter) {
        if(!__webpack_require__.o(exports, name)) {
             Object.defineProperty(exports, name, {
                 configurable: false,
                 enumerable: true,
                 get: getter
             });
        }
   };

    // getDefaultExport function for compatibility with non-harmony modules
   __webpack_require__.n = function(module) {
         var getter = module && module.__esModule ?
            function getDefault() { return module['default']; } :
            function getModuleExports() { return module; };
        __webpack_require__.d(getter, 'a', getter);
        return getter;
     };
 
     // Object.prototype.hasOwnProperty.call
      __webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };
 
      // __webpack_public_path__
     __webpack_require__.p = "";
 
     // Load entry module and return exports
     return __webpack_require__(__webpack_require__.s = 22);
  }) 
 ([
 /***/ (function(module, exports) {
 .......................................
  }),((function(module, exports, __webpack_require__) {})...])
```






commonjs,webpack,requirejs

## AMD
AMD是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。
它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，
都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。
1.加载
require([module], callback);
2.模块定义
define([module], callback);

requirejs
（1）实现js文件的异步加载，避免网页失去响应；

（2）管理模块之间的依赖性，便于代码的编写和维护。
  	
    1.定义模块
    define([],fun)
    fun函数return 导出对象。
    2.
    
    3.加载非规范的模块
    require.config({
　　　　shim: {
　　　　　　'backbone': {
　　　　　　　　deps: ['underscore', 'jquery'],
　　　　　　　　exports: 'Backbone'
　　　　　　}

　　　　}

　　});
    deps表示依赖，exports导出
    
    
 ```javascript
 //main.js
 require.config({

    paths : {
//		"jquery" : [ "http://apps.bdimg.com/libs/jquery/1.11.3/jquery.min", "js/jquery" ],
        "a" : [ "./a" ],
        "b" : "./b"
    }
});

//a.js
define(function(){
        return {name:'第一步'}
    }
)
//b.js
define(["a"],function(a){
        console.log(a.name)

    return {
            name:"第二步"
    }
 })
```

 
 
1、模块依赖加载之后，如何调用回调函数

2、加载依赖之后，如何将接口暴露给回调函数

3、如何解决循环依赖的问题()
用scope模式传参方式； //待查
用pubsub解耦； 
用require(“A”)的方式： 
//Inside b.js:
define(["require", "a"],
    function(require, a) {
        //"a" in this case will be null if a also asked for b,
        //a circular dependency.
        return function(title) {
            return require("a").doSomething();
        }
    }
);

//Inside b.js:
define(function(require, exports, module) {
    //If "a" has used exports, then we have a real
    //object reference here. However, we cannot use
    //any of a's properties until after b returns a value.
    var a = require("a");

    exports.foo = function () {
        return a.bar();
    };
});

https://www.cnblogs.com/terrylin/p/3347073.html

4、如何解决重复加载的问题（.loaded=true）

## CMD
// CMD
define(function(require, exports, module) {   
var a = require('./a')   a.doSomething()   
// 此处略去 100 行   var b = require('./b') 
// 依赖可以就近书写   b.doSomething()   
// ...
})

https://github.com/seajs/seajs/issues/242


ES6
ES6输出的是值的引用，使用import命令时不会去执行模块，只会生成一个动态的只读引用，指向的地址是只读的不能重新赋值，
需要时再到模块中取值，模块中的变量绑定其所在模块；
commonjs输出的是值的拷贝，模块内部变化不会影响这个值。

ES6模块的设计思想是尽量静态化，编译时就能确定模块的依赖关系。
ES6模块是动态引用，遇到import命令不会去执行模块，只是生产一个指向被加载模块的引用，需要开发者自己保证真正取值时能够取到。


1.加载模块
import具有提升效果，会提升到整个模块的头部首先执行。import会执行所加载的模块。
同步：import {foo} from module1  
import 'lodash'
import * as area from './area'  加载整体模块
异步：
System.import('some_module')  
	  	.then(some_module => {
		  	// Use some_module
	  	})
	  	.catch(error => {
		  	...
	  	});

2.导出模块

export语句输出的值是动态绑定，绑定器所在的模块。export default 默认输出命令，引入模块时可以指定任意名称。
一个模块只能有一个模块输出，
export function abc(){}//export 一个命名的function  
export default function(){} //export default function 

继承：
//parentjs
export function aa(){}

//childjs
export * from 'parentjs'//export * 表示输出parent模块的所有属性和方法。
export default function(){}








http://cnodejs.org/topic/5641bfbc184a5f7a5b5077fd
http://www.ruanyifeng.com/blog/2012/10/javascript_module.html
