---
layout: post
title:  "JavaScript编码规范"
keywords: "javascript"
description: "JavaScript编码规范"
category: JavaScript
tags: [javascript]
---
#### 类型
* 基本类型：访问基本类型时，应该直接操作类型值
  * String
  * number
  * boolean
  * null
  * undefined

```
var foo=1;
var bar=fool;
bar=9;
console.log(foo,bar);  //=>1,9
```
* 复合类型：访问复合类型时，应该操作其引用
  * object
  * array
  * function

```
var foo=[1,2];
var bar=foo;
bar[0]=9;
console.log(foo[0],bar[0]); //=>9,9
```
### 对象
* 使用字面量语法创建对象

```
var item = new Object(); // bad
var item = {}; // good
```
* 使用易读的同义词代替保留字

```
// bad
var superman = {
  class: 'alien'
};
// bad
var superman = {
  klass: 'alien'
};
// good
var superman = {
  type: 'alien'
};
```
### 数组
* 使用字面量语法创建数组

```
// bad
var items = new Array();
// good
var items = [];
```
* 添加数组元素时，使用push而不是直接添加

```
var someStack=[];
// bad
someStack[someStack.length] = 'abracadabra';
// good
someStack.push('abracadabra');
```
* 需要复制数组时，可以使用slice,jsPerf的相关文章

```
var len = items.length;
var itemsCopy = [];
var i;
// bad
for (i = 0; i < len; i++) {
  itemsCopy[i] = items[i];
}
// good
itemsCopy = items.slice();
```
* 使用slice将类数组对象转为数组

```
function trigger() {
  var args = Array.prototype.slice.call(arguments);
  ...
}
```
### 字符串
* 对字符串使用单引号

```
// bad
var name = "Bob Parr";
// good
var name = 'Bob Parr';
// bad
var fullName = "Bob " + this.lastName;
// good
var fullName = 'Bob ' + this.lastName;
```
* 超过80个字符的字符串应该使用字符串连接符进行跨行
* 注意：对长字符串过度使用连接符将会影响性能，相关的文章和主题讨论：jsPerf&Discussion

```
// bad
var errorMessage = 'This is a super long error that was thrown because of Batman. When you stop to think about how Batman had anything to do with this, you would get nowhere fast.';
// bad
var errorMessage = 'This is a super long error that was thrown because 
of Batman. When you stop to think about how Batman had anything to do 
with this, you would get nowhere 
fast.';
// good
var errorMessage = 'This is a super long error that was thrown because ' +
  'of Batman. When you stop to think about how Batman had anything to do ' +
  'with this, you would get nowhere fast.';
```
* 以编程方式创建字符串的时应该使用Array的join方法而不是通过连接符，尤其是在IE中：jsPerf
* 
```
var items;
var messages;
var length;
var i;
messages = [{
  state: 'success',
  message: 'This one worked.'
}, {
  state: 'success',
  message: 'This one worked as well.'
}, {
  state: 'error',
  message: 'This one did not work.'
}];
length = messages.length;
// bad
function inbox(messages) {
  items = '<ul>';
  for (i = 0; i < length; i++) {
    items += '<li>' + messages[i].message + '</li>';
  }
  return items + '</ul>';
}
// good
function inbox(messages) {
  items = [];
  for (i = 0; i < length; i++) {
    items[i] = '<li>' + messages[i].message + '</li>';
  }
  return '<ul>' + items.join('') + '</ul>';
}
```
### 函数
* 函数表达式
 
```
// anonymous function expression
var anonymous = function() {
  return true;
};
// named function expression
var named = function named() {
  return true;
};
// immediately-invoked function expression (IIFE)
(function() {
  console.log('Welcome to the Internet. Please follow me.');
})();
```
* 不要在非函数块中(if, while, etc)声明函数，尽管浏览器允许你分配函数给一个变量，但坏消息是，不同的浏览器用不同的方式解析它
* 注意：ECMA-262把块定义为一组语句，但函数声明不是一个语句：Read ECMA-262’s note on this issue.

```
// bad
if (currentUser) {
  function test() {
    console.log('Nope.');
  }
}
// good
var test;
if (currentUser) {
  test = function test() {
    console.log('Yup.');
  };
}
```
* 不要命名一个参数为arguments，否则它将优先于传递给每个函数作用域中的arguments对象

```
// bad
function nope(name, options, arguments) {
  // ...stuff...
}
// good
function yup(name, options, args) {
  // ...stuff...
}
```
### 属性
* 使用点表示法访问属性

```
var luke = {
  jedi: true,
  age: 28
};
// bad
var isJedi = luke['jedi'];
// good
var isJedi = luke.jedi;
```
* 使用变量访问属性时要使用下标表示法([])

```
var luke = {
  jedi: true,
  age: 28
};
function getProp(prop) {
  return luke[prop];
}
var isJedi = getProp('jedi');
```
### 变量
* 总是使用var声明变量，不然其将变为全局变量。我们要想办法避免全局空间污染

```
// bad
superPower = new SuperPower();
// good
var superPower = new SuperPower();
```
* 使用var声明每个变量，这样很容易添加新的变量声明，而不用去担心用a;替换a

```
// bad
var items = getItems(),
    goSportsTeam = true,
    dragonball = 'z';
// bad
// (compare to above, and try to spot the mistake)
var items = getItems(),
    goSportsTeam = true;
    dragonball = 'z';
// good
var items = getItems();
var goSportsTeam = true;
var dragonball = 'z';
```
* 最后声明未赋值的变量，这对于你需要根据之前已经赋值的变量对一个变量进行赋值时是很有帮助的

```
// bad
var i, len, dragonball,
    items = getItems(),
    goSportsTeam = true;
// bad
var i;
var items = getItems();
var dragonball;
var goSportsTeam = true;
var len;
// good
var items = getItems();
var goSportsTeam = true;
var dragonball;
var length;
var i;
```
* 在作用域顶端对变量赋值，这有助于避免变量声明问题和与声明提升相关的问题

```
// bad
function() {
  test();
  console.log('doing stuff..');
  //..other stuff..
  var name = getName();
  if (name === 'test') {
    return false;
  }
  return name;
}
// good
function() {
  var name = getName();
  test();
  console.log('doing stuff..');
  //..other stuff..
  if (name === 'test') {
    return false;
  }
  return name;
}
// bad
function() {
  var name = getName();
  if (!arguments.length) {
    return false;
  }
  return true;
}
// good
function() {
  if (!arguments.length) {
    return false;
  }
  var name = getName();
  return true;
}
```
### 比较运算符&相等
* 使用===和!==代替==和!=
* 比较运算符进行计算时会利用ToBoolean方法进行强制转换数据类型，并遵从一下规则
  * Objects的计算值是true
  * Undefined的计算值是false
  * Boolean的计算值是boolean的值
  * Numbers如果是-0，+0或者NaN，则计算值是false,反之是true
  * Strings如果是空，则计算值是false,反之是true
  
```
if ([0]) {
  // true
  // An array is an object, objects evaluate to true
}
```
* 使用快捷方式

```
// bad
if (name !== '') {
  // ...stuff...
}
// good
if (name) {
  // ...stuff...
}
// bad
if (collection.length > 0) {
  // ...stuff...
}
// good
if (collection.length) {
  // ...stuff...
}
```
### 语句块
* 对多行的语句块使用大括号

```
// bad
if (test)
  return false;
// good
if (test) return false;
// good
if (test) {
  return false;
}
// bad
function() { return false; }
// good
function() {
  return false;
}
```
* 对于使用if和else的多行语句块，把else和if语句块的右大括号放在同一行

```
// bad
if (test) {
  thing1();
  thing2();
}
else {
  thing3();
}
// good
if (test) {
  thing1();
  thing2();
} else {
  thing3();
}
```
### 注释
* 多行注释使用/** … */，需包含一个描述、所有参数的具体类型和值以及返回值

```
// bad
// make() returns a new element
// based on the passed in tag name
// @param {String} tag
// @return {Element} element
function make(tag) {
  // ...stuff...
  return element;
}
// good
/**
 * make() returns a new element
 * based on the passed in tag name
 *
 * @param {String} tag
 * @return {Element} element
 */
function make(tag) {
  // ...stuff...
  return element;
}
```
* 单行注释使用//，把单行注释放在语句的上一行，并且在注释之前空一行

```
// bad
var active = true;  // is current tab
// good
// is current tab
var active = true;
// bad
function getType() {
  console.log('fetching type...');
  // set the default type to 'no type'
  var type = this._type || 'no type';
  return type;
}
// good
function getType() {
  console.log('fetching type...');
  // set the default type to 'no type'
  var type = this._type || 'no type';
  return type;
}
```
* 如果你指出的问题需要重新定位或者提出一个待解决的问题需要实现，给注释添加FIXME or TODO 前缀有利于其他开发者快速理解。这些注释不同于通常的注释，因为它们是可实施的。这些实施措施就是 ```FIXME -- need to figure this outorTODO -- need to implement```
* 使用// FIXME:给一个问题作注释

```
function Calculator() {
  // FIXME: shouldn't use a global here
  total = 0;
  return this;
}
```
* 使用//TODO:给问题解决方案作注释

```
function Calculator() {
  // TODO: total should be configurable by an options param
  this.total = 0;
  return this;
}
```
### 空白
* 使用软制表符设置两个空格

```
// bad
function() {
∙∙∙∙var name;
}
// bad
function() {
∙var name;
}
// good
function() {
∙∙var name;
}
```
* 在左大括号之前留一个空格

```
// bad
function test(){
  console.log('test');
}
// good
function test() {
  console.log('test');
}
// bad
dog.set('attr',{
  age: '1 year',
  breed: 'Bernese Mountain Dog'
});
// good
dog.set('attr', {
  age: '1 year',
  breed: 'Bernese Mountain Dog'
});
```
* 在控制语句中（if,whileetc），左括号之前留一个空格。函数的参数列表之前不要有空格

```
// bad
if(isJedi) {
  fight ();
}
// good
if (isJedi) {
  fight();
}
// bad
function fight () {
  console.log ('Swooosh!');
}
// good
function fight() {
  console.log('Swooosh!');
}
```
* 用空白分隔运算符

```
// bad
var x=y+5;
// good
var x = y + 5;
```
* 用一个换行符结束文件

```
// bad
(function(global) {
  // ...stuff...
})(this);
// bad
(function(global) {
  // ...stuff...
})(this);
// good
(function(global) {
  // ...stuff...
})(this);
```
* 当调用很长的方法链时使用缩进，可以强调这行是方法调用，不是新的语句

```
// bad
$('#items').find('.selected').highlight().end().find('.open').updateCount();
// bad
$('#items').
  find('.selected').
    highlight().
    end().
  find('.open').
    updateCount();
// good
$('#items')
  .find('.selected')
    .highlight()
    .end()
  .find('.open')
    .updateCount();
// bad
var leds = stage.selectAll('.led').data(data).enter().append('svg:svg').classed('led', true)
    .attr('width',  (radius + margin) * 2).append('svg:g')
    .attr('transform', 'translate(' + (radius + margin) + ',' + (radius + margin) + ')')
    .call(tron.led);
// good
var leds = stage.selectAll('.led')
    .data(data)
  .enter().append('svg:svg')
    .classed('led', true)
    .attr('width',  (radius + margin) * 2)
  .append('svg:g')
    .attr('transform', 'translate(' + (radius + margin) + ',' + (radius + margin) + ')')
    .call(tron.led);
```
* 在语句块和下一个语句之前留一个空行

```
// bad
if (foo) {
  return bar;
}
return baz;
// good
if (foo) {
  return bar;
}
return baz;
// bad
var obj = {
  foo: function() {
  },
  bar: function() {
  }
};
return obj;
// good
var obj = {
  foo: function() {
  },
  bar: function() {
  }
};
return obj;
```
### 逗号
* 不要在语句前留逗号

```
// bad
var story = [
    once
  , upon
  , aTime
];
// good
var story = [
  once,
  upon,
  aTime
];
// bad
var hero = {
    firstName: 'Bob'
  , lastName: 'Parr'
  , heroName: 'Mr. Incredible'
  , superPower: 'strength'
};
// good
var hero = {
  firstName: 'Bob',
  lastName: 'Parr',
  heroName: 'Mr. Incredible',
  superPower: 'strength'
};
```
* 不要有多余逗号：这会在IE6、IE7和IE9的怪异模式中导致一些问题；同时，在ES3的一些实现中，多余的逗号会增加数组的长度。在ES5中已经澄清（source）

```
// bad
  var hero = {
    firstName: 'Kevin',
    lastName: 'Flynn',
  };
  var heroes = [
    'Batman',
    'Superman',
  ];
  // good
  var hero = {
    firstName: 'Kevin',
    lastName: 'Flynn'
  };
  var heroes = [
    'Batman',
    'Superman'
  ];
```
### 分号
* 这也是规范一部分

```
// bad
(function() {
  var name = 'Skywalker'
  return name
})()
// good
(function() {
  var name = 'Skywalker';
  return name;
})();
// good (guards against the function becoming an argument when two files with IIFEs are concatenated)
;(function() {
  var name = 'Skywalker';
  return name;
})();
```
### 类型分配&强制转换
* 执行强制类型转换的语句
  String:
  
  ```
 //  => this.reviewScore = 9;
// bad
var totalScore = this.reviewScore + '';
// good
var totalScore = '' + this.reviewScore;
// bad
var totalScore = '' + this.reviewScore + ' total score';
// good
var totalScore = this.reviewScore + ' total score';
  ```
* 使用parseInt对Numbers进行转换，并带一个进制作为参数

```
var inputValue = '4';
// bad
var val = new Number(inputValue);
// bad
var val = +inputValue;
// bad
var val = inputValue >> 0;
// bad
var val = parseInt(inputValue);
// good
var val = Number(inputValue);
// good
var val = parseInt(inputValue, 10);
```
* 无论出于什么原因，或许你做了一些”粗野”的事；或许parseInt成了你的瓶颈；或许考虑到性能，需要使用位运算，都要用注释说明你为什么这么做

```
// good
/**
 * parseInt was the reason my code was slow.
 * Bitshifting the String to coerce it to a
 * Number made it a lot faster.
 */
var val = inputValue >> 0;
```
* 注意：当使用位运算时，Numbers被视为64位值，但是位运算总是返回32位整型(source)。对于整型值大于32位的进行位运算将导致不可预见的行为。Discussion.最大的有符号32位整数是2,147,483,647

```
2147483647 >> 0 //=> 2147483647
2147483648 >> 0 //=> -2147483648
2147483649 >> 0 //=> -2147483647
```
* Booleans

```
var age = 0;
// bad
var hasAge = new Boolean(age);
// good
var hasAge = Boolean(age);
// good
var hasAge = !!age;
```
### 命名规范
* 避免单字母名称，让名称具有描述性

```
// bad
function q() {
  // ...stuff...
}
// good
function query() {
  // ..stuff..
}
```
* 当命名对象、函数和实例时使用骆驼拼写法

```
// bad
var OBJEcttsssss = {};
var this_is_my_object = {};
function c() {}
var u = new user({
  name: 'Bob Parr'
});
// good
var thisIsMyObject = {};
function thisIsMyFunction() {}
var user = new User({
  name: 'Bob Parr'
});
```
* 当命名构造函数或类名时，使用驼峰式写法

```
// bad
function user(options) {
  this.name = options.name;
}
var bad = new user({
  name: 'nope'
});
// good
function User(options) {
  this.name = options.name;
}
var good = new User({
  name: 'yup'
});
```
* 命名私有属性时使用前置下划线

```
// bad
this.__firstName__ = 'Panda';
this.firstName_ = 'Panda';
// good
this._firstName = 'Panda';
```
* 保存this引用时使用_this

```
// bad
function() {
  var self = this;
  return function() {
    console.log(self);
  };
}
// bad
function() {
  var that = this;
  return function() {
    console.log(that);
  };
}
// good
function() {
  var _this = this;
  return function() {
    console.log(_this);
  };
}
```
* 命名函数时，下面的方式有利于堆栈跟踪

```
// bad
var log = function(msg) {
  console.log(msg);
};
// good
var log = function log(msg) {
  console.log(msg);
};
```
* 如果文件作为一个类被导出，文件名应该和类名保持一致

```
// file contents
class CheckBox {
  // ...
}
module.exports = CheckBox;
// in some other file
// bad
var CheckBox = require('./checkBox');
// bad
var CheckBox = require('./check_box');
// good
var CheckBox = require('./CheckBox');
```
### 存取器
* 对于属性，访问器函数不是必须的
* 如果定义了存取器函数，应参照getVal() 和 setVal(‘hello’)格式.

```
// bad
dragon.age();
// good
dragon.getAge();
// bad
dragon.age(25);
// good
dragon.setAge(25);
```
* 如果属性时boolean，格式应为isVal() or hasVal()

```
// bad
if (!dragon.age()) {
  return false;
}
// good
if (!dragon.hasAge()) {
  return false;
}
```
* 创建get() and set()函数时不错的想法，但是要保持一致

```
function Jedi(options) {
  options || (options = {});
  var lightsaber = options.lightsaber || 'blue';
  this.set('lightsaber', lightsaber);
}
Jedi.prototype.set = function(key, val) {
  this[key] = val;
};
Jedi.prototype.get = function(key) {
  return this[key];
};
```
### 构造函数
* 在原型对象上定义方法，而不是用新对象重写它。重写使继承变为不可能：重置原型将重写整个基类

```
function Jedi() {
  console.log('new jedi');
}
// bad
Jedi.prototype = {
  fight: function fight() {
    console.log('fighting');
  },
  block: function block() {
    console.log('blocking');
  }
};
// good
Jedi.prototype.fight = function fight() {
  console.log('fighting');
};
Jedi.prototype.block = function block() {
  console.log('blocking');
};
```
* 方法应该返回this，有利于构成方法链

```
// bad
Jedi.prototype.jump = function() {
  this.jumping = true;
  return true;
};
Jedi.prototype.setHeight = function(height) {
  this.height = height;
};
var luke = new Jedi();
luke.jump(); // => true
luke.setHeight(20); // => undefined
// good
Jedi.prototype.jump = function() {
  this.jumping = true;
  return this;
};
Jedi.prototype.setHeight = function(height) {
  this.height = height;
  return this;
};
var luke = new Jedi();
luke.jump()
  .setHeight(20);
```
* 写一个自定义的toString()方法是可以的，只要确保它能正常运行并且不会产生副作用

```
function Jedi(options) {
  options || (options = {});
  this.name = options.name || 'no name';
}
Jedi.prototype.getName = function getName() {
  return this.name;
};
Jedi.prototype.toString = function toString() {
  return 'Jedi - ' + this.getName();
};
```
### 事件
* 当在事件对象上附加数据时（无论是DOM事件还是如Backbone一样拥有的私有事件），应传递散列对象而不是原始值，这可以让随后的贡献者给事件对象添加更多的数据，而不必去查找或者更新每一个事件处理程序。举个粟子，不要用下面的方式

```
// bad
$(this).trigger('listingUpdated', listing.id);
...
$(this).on('listingUpdated', function(e, listingId) {
  // do something with listingId
});
```
应该按如下方式：

```
// good
$(this).trigger('listingUpdated', { listingId : listing.id });
...
$(this).on('listingUpdated', function(e, data) {
  // do something with data.listingId
});
```
### 模块
* 模块应该以 ! 开始，这能确保当脚本连接时，如果畸形模块忘记导入，包括最后一个分号，不会产生错误。Explanation
* 文件应该以驼峰式命名，放在同名的文件夹中，和单出口的名称相匹配
* 定义一个noConflict()方法来设置导出模块之前的版本,并返回当前版本。
* 在模块的顶部申明’use strict';

```
// fancyInput/fancyInput.js
!function(global) {
  'use strict';
  var previousFancyInput = global.FancyInput;
  function FancyInput(options) {
    this.options = options || {};
  }
  FancyInput.noConflict = function noConflict() {
    global.FancyInput = previousFancyInput;
    return FancyInput;
  };
  global.FancyInput = FancyInput;
}(this);
```
### jQuery
* jQuery对象变量使用前缀$

```
// bad
var sidebar = $('.sidebar');
// good
var $sidebar = $('.sidebar');
```
* 缓存jQuery查询

```
// bad
function setSidebar() {
  $('.sidebar').hide();
  // ...stuff...
  $('.sidebar').css({
    'background-color': 'pink'
  });
}
// good
function setSidebar() {
  var $sidebar = $('.sidebar');
  $sidebar.hide();
  // ...stuff...
  $sidebar.css({
    'background-color': 'pink'
  });
}
```
* 使用级联$(‘.sidebar ul’)或父子$(‘.sidebar > ul’)选择器进行DOM查询
* 在范围内使用find进行jQuery对象查询

```
// bad
$('ul', '.sidebar').hide();
// bad
$('.sidebar').find('ul').hide();
// good
$('.sidebar ul').hide();
// good
$('.sidebar > ul').hide();
// good
$sidebar.find('ul').hide();
```