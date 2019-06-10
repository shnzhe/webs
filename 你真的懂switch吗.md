你真的懂switch吗？聊聊switch语句中的块级作用域

  最近在代码中不小心不规范的，在switch里面定义了块级变量，导致页面在某些浏览器中出错，本文讨论以下switch语句中的块级作用域。

- switch语句中的块级作用域
- switch语句中的块级作用域可能存在的问题
- 规范和检测
## 一、switch语句中的块级作用域

  ES6 或 TS 引入了块级作用域,通过let和const、class等可以定义块级作用域里的变量，块级作用域内的变量不存在变量提升，且存在暂时性死区。常见的if语句，for循环的循环体内都可以定义块级变量。那么switch语句中的块级作用域是什么呢？ 先给出结论：

***switch语句中的块级作用域，在整个switch语句中，而不是对于每一个case生成一个独立的块级作用域。***

下面来举几个例子来说明这个问题：
~~~
let number = 1;
switch(number){
  case 1:
    let name = 'Jony';
  default:
    console.log(name)
}
~~~
上述的代码会输出jony。

再看一个例子：
```
let number = 1;
switch(number){
  case 1:
    let name = 'Jony';
    break;
  case 2:
    let name = 'yu';
    break;
  default:
   console.log(name);
}
```
这样会在重复生命的错误：
```
Uncaught SyntaxError: Identifier 'name' has already been declared
```
上述两个例子说明确实switch语句中，整个switch语句构成一个块级作用域。而与case无关,每一个case并不会构成一个独立的块级作用域。

##二、switch语句中的块级作用域可能存在的问题

  我们知道了switch语句，整个switch语句的顶层是一个块级作用域，但是还要注意case的特殊性，在case中声明的变量，并不会提升到块级作用域中。
```
let number = 2;
switch(number){
  case 2:
    name = 'yu';
    break;
}
```
在这个例子中，name虽然没有声明，但是给name赋值相当于给全局的window对象复制，也就是window.name = 'yu'。不会有任何问题。

有意思的问题来了：
```
let number = 2;
switch(number){
  case 1:
    let name = 'jony';
    break;
  case 2:
    name = 'yu';
    break;
}
```
这个例子中，会报错，会报name未定义的错误。
```
Uncaught ReferenceError: name is not defined
```
原因的话，这里虽然case里面定义的块级虽然不会存在变量提升，但是会存在暂时性死区,也就是说如果let name = 'jony' 没有执行，也就是name定义的过程没有执行，那么name在整个块级作用域内都是不可用的，都是undefined。

为了证明我们的想法，接着改写上面的例子：
```
let number = 1;
switch(number){
  case 1:
    let name = 'jony';
    break;
  case 2:
    name = 'yu';
    break;
}
```
我们把number改成1,我们发现代码不会报任何的错误,因为此时let name的定义和赋值都被执行了。

##三、规范和检测

###一、什么时候会出现问题

  可能会说为什么在自己的项目中，在ES6或者TS代码中即使有上述的错误使用，也没有报错？

  笔者之前也有这样的问题，要明确的是是否你把ES6或者TS的代码直接转化成了es5，然后再调试或者发布的线上的，当let被编译成es5后，当然就不会存在上述switch中作用域的问题。但是现实中，编译成es5后的js文件可能太大，对于高版本浏览器我们希望直接使用ES6代码（通过type = module来判断浏览器对于ES6的支持性），那么这么上述问题就会出现。

###二、如何检测和规避

  那么如何避免这种情况呢，当然最好的方式，就是不要在case中定义块级变量,但是万一不小心写了上述的问题代码如何检测呢。

首先使用typescript，静态编译是不能出现错误提示的，因为这个错误是运行时异常。最好的方式是通过编写eslint的规范来解决上述的非法使用问题。
