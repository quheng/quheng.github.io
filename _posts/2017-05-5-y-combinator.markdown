---
layout: post
title: "YAy-combinator"
date: 2017-05-05
categories: computer
---

Yet Another y-combinator。

y-combinator 还是挺酷的一个概念，mit cs 的 logo ![](https://encrypted-tbn1.gstatic.com/images?q=tbn:ANd9GcRiGfnbNBMtdIo1jN-T1IAAasnkoEm4QBeWaHNM335jCGnoeuvI9A) 以及著名的孵化器 ![y-combinator](http://www.ycombinator.com/) ![](http://www.ycombinator.com/images/ycombinator-logo-fb889e2e.png)

总的来说，y-combinator 是用来在不扩充 lambda 演算定义的情况下实现递归。在大多数实际编程语言的实现上是不会使用 y-combinator 的，而是会根据现代计算机的特点对 lambda 演算进行扩充。但是思考 y-combinator 是一件很有意思的事情。

首先我们先来看一下什么是 lambda 演算，很简单，只有三条规则：
1. 变量(Variable): a, 用于表示数学或者逻辑值
2. 抽象(Abstraction): (λx.M), 函数的定义，输入是 x，输出是 M，变量 x 的使用范围在 M 内，M 是 lambda term
3. 应用(Application): (M N), 使用参数 N 求函数 M 的值，M N 都是 lambda term

很简单的规则，“定义变量”，构造函数，函数求值。注意这里的 “变量” 更确切的说是 “常量”，lambda 演算只规定了 “定义”，它没有规定 “修改”，也就是说 “变量” 一旦被定义就无法修改。这也就是说，一旦用 `a` 表示了 `b`，那么 `a` 就永远是 `b`，可以进行替换，“引用透明” 大致讲的就是这个事情。

接下来我们来看一个阶乘的例子，用 ES6+ 表示，下同：

```
var f = (n) => (n === 1? 1 : n * f(n - 1)) 
```

so far so good，有什么问题么？这里使用了 `var f`，在不扩充 lambda 演算（修改变量）的情况下，我们无法这样定义。`var f` 由 `(n) => (n === 1? 1 : n * f(n - 1))` 定义，而后面的 `f` 就没有定义了。在可以修改变量的情况下，我们可以这样实现：
```
var f = ''
f = (n) => (n === 1? 1 : n * f(n - 1))
```
下面我们就试着在不引入 “赋值” 的情况下如何实现递归。

“赋值” 版阶乘结合 lambda 演算的三条规则，可以较为自然的得到一个想法，我把 `f` 作为参数传进去：
```
var p = (f) => (
    (n) => (n === 1? 1 : n * f(n - 1))
)
```
看起来可行，可是我们如何得到 `f` 呢？ 思考 `p` 有什么性质： `p(f) = f`，也就是说我们要找的 `f` 是 `p` 的不动点（若 f(x) = x，则称 x 是 f(x) 的不动点）。如果我们能构造出一个函数 `y`，它能够找到任何函数的不动点，便解决了这个问题。函数 `y` 被称作 `y-combinator`。

我们构造函数 `g`，满足 `g(g) = f`，展开来写作
```
g = (self) => f
```
注意，g 只能将 g 传进去。

我们把 `f = p(f)` 以及 `f = g(g)`，带入得到

```
g = (self) => p((self(self)))
```

至此，我们得到了一个和 `f` 无关的 `g`，并且可以用 `g(g)` 来表示 `f` 了，完成 `y-combinator` 的构建：

```
y = (p) => {
	var g = (self) => p((self(self)))
	return g(g)
}
```

输入 `p`，输出 `p` 的不动点，保证 `p(y(p)) = y(p)`。

值得注意的是，由于 js 是 applicative language(函数在调用的时候立即求值，而不是等到需要的时候再求值)，上述代码会造成死循环，需要将 `p((self(self))` 包裹一下，延迟 `self(self)` 的求值:

```
const y = (p) => {
	const g = (self) => (x) => p((self(self)))(x)
	return g(g)
}
```

这又被称作 `z-combinator`。

测试一下：
```
const p = (f) => (n) => (n === 1? 1 : n * f(n - 1))

y(p)(10)  // 3628800
``` 

有人说这里不是又用了 `var` 来定义 `g`，和最开始的写法有什么区别。区别在于这里的 `g` 和 `(self) => (x) => p((self(self)))(x)` 是严格等价的，相当于取了一个名字，表达起来方便，我们可以把 `g` 替换掉得到：

```
((p) => ((self) => (x) => p((self(self)))(x))((self) => (x) => p((self(self)))(x)))(p)(10)  // 3628800
```
 
no more var, no more const, no more define, just fat.

