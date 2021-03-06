---
title: 少年，不要滥用箭头函数啊
date: 2016-12-08 16:30:55
tags:
---

在ES6大行其道的今天，不应用点ES6特性似乎有些政治不正确。最近刚好有个Node的项目，最低要支持到nodejs 4.0，在[node.green](1)看了下ES6的支持度，我想使用的特性基本都有支持，遂决定在新项目中采用ES6来写。

当然第一件事情就是毫不留情地消灭var，项目中能用const的地方不用let，能用let的地方不用var。

第二件事情就是使用劳动人民喜闻乐见的箭头函数替代function。当我心满意足地看到满屏的`=>`时，现实给了我一记响亮的耳光——改过之后的程序错误百出！

所以，当我们使用箭头函数时，一定要搞清楚箭头函数是什么回事，适用于什么场景。本文就针对以上问题来讨论下箭头函数。

## 箭头函数是什么？
箭头函数的语法我就不讲了，相信大家都见识过。跟我一样，大家喜欢箭头函数90%的原因是它好看。除了好看，它是不是与function等价呢？肯定不等价，因为TC39不可能仅因为好看而引入一个语法糖（class除外）。

箭头函数的渊源可以追溯到上古时期一个叫lambda演算的东西。lambda演算是数学家提出来的，有些数学家跟我们程序员一样也很懒，数学定理那么多，今天要证三角定律，明天要证勾股定律，累不累！那能不能将所有的证明问题用一个统一的体系进行形式化描述，然后由机器来完成自动推导呢？lambda演算就是干这个的，图灵也搞了一套体系叫图灵机，两者是等价的。

关于lambda演算说了这么多，好像跟今天要讲的箭头函数没什么关系？其实是有关系的，lambda演算深刻影响了箭头函数的设计。数学家们喜欢用纯函数式编程语言，纯函数的特点是没有副作用，给予特定的输入，总是产生确定的输出，甚至有些情况下通过输出能够反推输入。要实现纯函数，必须使函数的执行过程不依赖于任何外部状态，整个函数就像一个数学公式，给定一套输入参数，不管是在地球上还是火星上执行都是同一个结果。

箭头函数要实现类似纯函数的效果，必须剔除外部状态。所以当你定义一个箭头函数，在普通函数里常见的`this`、`arguments`、`caller`是统统没有的。

## 箭头函数没有`this`

箭头函数没有`this`，那下面的代码明显可以取到`this`啊：
```javascript
function foo() {
  this.a = 1
  let b = () => console.log(this.a)

  b()
}

foo()  // 1
```

以上箭头函数中的`this`其实是父级作用域中的`this`，即函数`foo`的`this`。箭头函数引用了父级的变量，构成了一个闭包。以上代码等价于：
```javascript
function foo() {
  this.a = 1

  let self = this
  let b = () => console.log(self.a)

  b()
}

foo()  // 1
```

箭头函数不仅没有`this`，常用的`arguments`也没有。如果你能获取到`arguments`，那它一定是来自父作用域的。
```javascript
function foo() {
  return () => console.log(arguments[0])
}

foo(1, 2)(3, 4)  // 1
```

上例中如果箭头函数有`arguments`，就应该输出的是3而不是1。

一个经常犯的错误是使用箭头函数定义对象的方法，如：
```javascript
let a = {
  foo: 1,
  bar: () => console.log(this.foo)
}

a.bar()  //undefined
```

以上代码中，箭头函数中的`this`并不是指向`a`这个对象。对象`a`并不能构成一个作用域，所以再往上到达全局作用域，`this`就指向全局作用域。如果我们使用普通函数的定义方法，输出结果就符合预期，这是因为`a.bar()`函数执行时作用域绑定到了`a`对象。
```javascript
let a = {
  foo: 1,
  bar: function() { console.log(this.foo) }
}

a.bar()  // 1
```

另一个错误是在原型上使用箭头函数，如：
```javascript
function A() {
  this.foo = 1
}

A.prototype.bar = () => console.log(this.foo)

let a = new A()
a.bar()  //undefined
```

同样，箭头函数中的`this`不是指向`A`，而是根据变量查找规则回溯到了全局作用域。同样，使用普通函数就不存在问题。

通过以上说明，我们可以看出，箭头函数除了传入的参数之外，真的是什么都没有！如果你在箭头函数引用了`this`、`arguments`或者参数之外的变量，那它们一定不是箭头函数本身包含的，而是从父级作用域继承的。

## 什么情况下该使用箭头函数

到这里，我们可以发现箭头函数并不是万金油，稍不留神就会踩坑。

至于什么情况该使用箭头函数，《You Don't Know About JS》给出了一个决策图：

{% asset_image arrow-function.png %}

以上决策图看起来有点复杂，我认为有三点比较重要：
1. 箭头函数适合于无复杂逻辑或者无副作用的纯函数场景下，例如用在`map`、`reduce`、`filter`的回调函数定义中；
2. 不要在最外层定义箭头函数，因为在函数内部操作`this`会很容易污染全局作用域。最起码在箭头函数外部包一层普通函数，将`this`控制在可见的范围内；
3. 如开头所述，箭头函数最吸引人的地方是简洁。在有多层函数嵌套的情况下，箭头函数的简洁性并没有很大的提升，反而影响了函数的作用范围的识别度，这种情况不建议使用箭头函数。

[1]: http://node.green/
