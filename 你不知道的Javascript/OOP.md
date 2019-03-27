## 面向对象

#### 继承

继承意味着复制操作，但是 Javascript 不会复制，它通过对象关联，来实现继承。

#### 属性屏蔽

```js
var anotherObject = { a: 2 };
var myObject = Object.create(anotherObject);
anotherObject.a; // 2
myObject.a; // 2
anotherObject.hasOwnProperty("a"); // true
myObject.hasOwnProperty("a"); // false
myObject.a++; // 隐式屏蔽! anotherObject.a; // 2

myObject.a; // 3
myObject.hasOwnProperty("a"); // true
```

a++ 相当于 a = a + 1; 在上述代码中相当于 myObject.a = myObject.a + 1;所以为 myObject 对象创建了 a 属性。

#### new

new 操作符赋予了函数构建对象的能力，函数本身并没有这种能力。

#### Function.constructor

```js
function Foo() {}
Foo.prototype = {};

var a = new Foo();
a.constructor === Foo; // true
Foo.prototype.constructor === Foo; // true
a.__proto__ === Foo.prototype; // true
```

a.constructor 指向 Foo 函数，是因为 a 的原型指向了 Foo 的原型对象，而 Foo 原型的构造函数是 Foo 本事，a 实际通过原型链访问到了 constructor 属性。通过打印对象 a，👋 会发现 a 本身并没有 constructor 属性。

**_constructor 的迷惑性_** ❌

```js
function Foo() {
  /* .. */
}
Foo.prototype = {
  /* .. */
}; // 创建一个新原型对象
var a1 = new Foo();

a1.constructor === Foo; // false!
a1.constructor === Object; // true!
```

当手动修改了 Foo 函数的原型对象，再通过 Foo 创建一个对象 a1 时，a1 的 constructor 值居然不是 Foo，而指向了 Object。对于 Foo 原型对象的重新赋值导致原型丢失了 constructor 属性，原型链再上一级的是 Object.prototype，所以通过 a1 获取 constructor 得到了 Object。  
**_可见，constructor 本身和构造者这个名称并不相符，它也不表示对象的构造器_**

#### Object.create

```js
var a = Object.create(null); // {}
```

Object.create(null)可以创建一个完全空的对象，不会有原型链，不会关联到 Object 函数，很适合于存储数据。
