# 如何使JS提高运行性能

## 数据的存取

Js中的存储方案

- 字面量：字符串、数字、布尔值、对象、数组、函数、正则表达式、null、undefined
- 本地变量：var定义的数据存储单元
- 数组元素：存储在js数组对象内部，以数字作为索引
- 对象成员：存储在js对象内部，以字符串作为索引

> 从字面量和局部变量中存取数据性能差异微不足道，访问数组和对象成员性能代价高一些。如果在乎运行速度，尽量使用字面量和局部变量，减少数组项和对象成员的使用。


### 作用域链

通过原型链知道函数都是Function对象的实例，Function对象和其他对象一样，拥有可以编程访问的属性，其中一个内部属性[[Scope]]包含了函数被创建的作用域中的对象的集合，这个集合被称为作用域链，作用域链决定了哪些数据可以被函数访问。作用域中的每个对象被称为可变对象，每个可变对象都以键值对的形式存在。

``` html
<script>
  function add(num1, num2) {
    var sum = num1 + num2
    return sum
  }
</script>
```

当add函数被初始化时（注意不是在add被执行的时候），在作用域链的顶部插入一个全局对象，代表着全局范围内定义的变量的集合，这些变量在add函数内可以被访问。

| add函数   |   作用域链   |   全局对象   |
| :-------- | :--------| :------ |
| [[Scope]]  |   0  |   this/window/document/add(注意此时add本身也是全局变量) |

> add函数的[[Scope]]属性指向了作用域链，作用域链中的首个对象指向全局对象（全局变量的集合）。

当add函数被执行时，会创建一个执行环境（执行上下文）的内部对象，一个执行环境定义了一个函数执行时的环境（函数每次的执行环境都是独一无二的），多次调用同一个函数会导致创建多个执行环境，函数执行完毕执行环境会被销毁。全局执行环境直到关闭网页或浏览器才会被销毁。



每个执行环境都有自己对应的作用域链，用于解析标识符。执行环境被创建的时候，作用域链初始化为当前运行函数的[[Scope]]属性中的对象。这些对象按照出现在函数中的顺序被复制到执行环境的作用域链中（前面说的全局对象，是在初始化的时候就在[[Scope]]中创建完成，而这里所说的是在执行函数中的除全局对象以外的和执行函数相关的局部变量、命名参数、参数集合以及this)，作用域链中复制的出现在函数中的对象(局部变量、命名参数、参数集合以及this)的新对象被称为活动对象。活动对象会被推入作用域链的最前端（后面是当前的执行环境的父执行环境的活动对象...一直到最后是全局对象）

| add函数   |   作用域链   |   活动对象和全部对象   |
| :-------- | :--------| :------ |
| [[Scope]]  |   0  |  this/arguments/num1/num2/sum  |
| [[Scope]]  |   1  |   this/window/document/add(注意此时add本身也是全局变量) |

函数在执行过程中，每遇到一个变量，都会经历一次标识符解析过程以决定从哪里获取或存储数据。该过程主要是依靠作用域链进行解析，首先搜索当前执行环境的活动对象，如果找到标识符对应的变量则终止搜索，否则继续搜索作用域链的下一个对象，搜索的过程会持续进行，直到全局对象，如果在全局对象中都没有匹配标识符的变量，那么该变量是未定义的（undefined）。

> 搜索是会消耗性能的，可以清楚的知道当前执行环境的活动对象的搜索是最快的，而全局对象的搜索则是最耗性能的。并且可以发现如果存在相同的变量，根据作用域链的搜索顺序决定先搜索到的变量被执行。如果某个跨作用域链的值在当前执行环境中被引用不止一次时，可以存储到局部变量中从而提升搜索性能。


需要注意作用域链本质上是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。


### 改变作用域链

#### with

尽量不要使用with，可能会使局部变量的访问代价更大，因为with会创建新的作用域。


#### catch


try代码如果发生错误，执行过程会自动跳转到catch子句，然后把异常对象推入一个变量对象并置于作用域的首位，在catch代码块内部，函数所有的局部变量将会放在第二个作用域链对象中。

为了尽量简化代码使得catch子句对性能的影响最小化，推荐的做法是将错误委托给一个函数来处理

``` html
<script>
  try {
    // ...
  } catch(err) {
    fn2(err)
  }
</script>
```

> 函数fn2是catch子句中唯一执行的代码，该函数接受错误产生的异常对象为参数，由于只执行一条语句，且没有局部变量的访问，作用域的临时改变不会影响代码性能。需要注意with或catch或eval子句被称为动态作用域，动态作用域只存在代码执行过程，无法通过静态分析，因此只有确实必要时才推荐使用动态作用域。

### 闭包、作用域和内存

闭包是指有权访问另一个函数作用域中的变量的函数，创建闭包的常见方式就是在一个函数内部创建另一个函数。

闭包可以访问局部作用域之外的数据，可能会导致性能问题。通常函数在执行完毕后会销毁执行环境，内存中仅保存全局作用域Global variable object，但是闭包的情况不同

``` html
<script>
  function fn() {
    var id = "ziyi2"
    //一个匿名的内部函数
    return function(obj1,obj2) {
      var value1 = obj1[id];
      var value2 = obj2[id];
      return value1 - value2
    }
  }
</script>
```

| fn执行环境  |   作用域链   |   活动对象和全部对象   |
| :-------- | :--------| :------ |
| [[Scope]]  |   0  |  this/arguments/id  |
| [[Scope]]  |   1  |  this/window/fn |



| 匿名函数（闭包）  |   作用域链   |   活动对象和全部对象   |
| :-------- | :--------| :------ |
| [[Scope]]  |   0  |  this/arguments/id  |
| [[Scope]]  |   1  |  this/window/fn |

当fn被执行的时候，fn执行环境对应的活动对象被创建（包含id变量），成为作用域链中的最前面的对象，需要注意此时匿名函数被创建（fn执行的过程中创建了匿名函数），匿名函数的[[Scope]]属性被初始化为fn执行环境对应的活动对象和全局对象（书上说是fn的作用域链，这里表述并不清楚，因为fn的作用域链是会被销毁的，但是fn的活动对象不会）。如果此时匿名函数被其他变量进行引用，那么尽管fn执行完毕，但是fn执行环境对应的活动对象并不会被销毁（执行环境中的作用域链仍然会被销毁，活动对象仍然存在内存中），因为很简单啊，匿名函数执行的时候还需要访问fn执行环境的活动对象啊（意味着使用闭包需要更多的内存开销）。

当被引用的匿名函数执行时，会创建新的执行环境，它的作用域链与属性[[Scope]]中所引用的两个对象一起被初始化，新的执行环境会创建新的活动对象并把它插入作用域链的首位


| 匿名函数（闭包）  |   作用域链   |   对象   | 类型   |
| :-------- | :--------| :------ | :------ |
| [[Scope]]  |   0  |  this/obj1/obj2/value1/value2/arguments | 闭包活动对象 |
| [[Scope]]  |   1  |  this/arguments/id  | fn活动对象 |
| [[Scope]]  |   2  |  this/window/fn | 全局对象  |

需要注意的是闭包产生了新的作用域，fn活动对象被排在了闭包活动对象之后，此时访问fn活动对象不可避免的会带来性能损失（此时如果fn活动对象以及全局对象访问比较频繁，则仍然可以开辟局部变量进行缓存，提升执行速度）。

``` html
<script>
  var fn1 = fn()
  var result = fn1({ziyi2:3},{ziyi2:4});
  console.log(result)
  // 解除对匿名函数的引用（以便释放内存）
  // 通知垃圾回收例程将其清除，匿名函数的作用域链被销毁
  fn1 = null 
</script>
```

### 对象成员

- 原型
- 原型链

搜索方法和属性的时候和作用域链类似，先搜索实例对象的属性和方法，如果未找到，则继续搜索原型对象的属性和方法。因此搜索原型对象的属性和方法会带来一定的性能损耗（属性或方法在原型链中存在的位置越深，则消耗的性能越大）。需要注意的是读取实例对象的数据会比读取对象字面量或局部变量的数据更消耗性能。

> 此时仍然可以通过缓存，将对象成员赋值给局部变量从而提升读写性能。

### 总结

如何提升存取性能

- 1. 访问字面量和局部变量的速度最快，访问数组元素和对象成员相对较慢。
- 2. 由于局部变量存在于作用域链的起始位置，因此访问局部变量比访问跨作用域变量更快，全局变量的访问速度最慢。
- 3. 避免使用with和catch，除非是有必要的情况下。
- 4. 嵌套的对象成员会明显影响性能，尽量少用，例如window.loacation.href。
- 5. 属性和方法在原型链中的位置越深，则访问它的速度也越慢。
- 6. 通常来说，需要访问多次的对象成员、数组元素、跨作用域变量可以保存在局部变量中从而提升js执行效率。


## 算法和流程控制

代码数量少并不意味着运行速度就快。代码数量多也不以为这运行速度一定慢。

### 循环

代码的执行时间大部分消耗在循环中。

#### 循环类型

- for
- while
- do...while
- for...in...（性能最差）
- for...of...（可以想象性能就算优化过也会比普通循环速度慢）

> for...in...速度慢的原因是不仅遍历实例的属性还会遍历原型链中继承而来的属性。

#### 循环性能


##### 性能测试

###### 循环次数较少

``` html
<script>
  let arr = new Array(1).fill(1)
  
  // for测试
  for(let i=0; i<10; i++) {
    console.time('for')
    for(let i=0, len=arr.length; i<len; i++) {
      arr[i]
    }
    console.timeEnd('for')
  }
  // for...in测试
  for(let i=0; i<10; i++) {
    console.time('for...in...')
    for(let index in arr) {
      arr[index]
    }
    console.timeEnd('for...in...')
  }
  // for...of测试
  for(let i=0; i<10; i++) {
    console.time('for...of...')
    for(let index of arr.keys()) {
      arr[index]
    }
    console.timeEnd('for...of...')
  }


/*
  for: 0.007080078125ms
  for: 0.004150390625ms
  for: 0.001953125ms
  for: 0.002685546875ms
  for: 0.001953125ms
  for: 0.0029296875ms
  for: 0.004150390625ms
  for: 0.0029296875ms
  for: 0.001953125ms

  for...in...: 0.011962890625ms
  for...in...: 0.005859375ms
  for...in...: 0.008056640625ms
  for...in...: 0.005859375ms
  for...in...: 0.0048828125ms
  for...in...: 0.003662109375ms
  for...in...: 0.004150390625ms
  for...in...: 0.0048828125ms
  for...in...: 0.003173828125ms

  for...of...: 0.01416015625ms
  for...of...: 0.01806640625ms
  for...of...: 0.003662109375ms
  for...of...: 0.005126953125ms
  for...of...: 0.0048828125ms
  for...of...: 0.004150390625ms
  for...of...: 0.003662109375ms
  for...of...: 0.005126953125ms
  for...of...: 0.004150390625ms
*/
</script>

```

> 数量极少的时候for速度最快，for...in...和for...of...性能差不多，在循环该测试是在chrome浏览器下。

###### 循环次数较多


``` html
<script>

/*
  for: 4.967041015625ms
  for: 4.348876953125ms
  for: 0.0458984375ms
  for: 0.045166015625ms
  for: 0.05224609375ms
  for: 0.044677734375ms
  for: 0.051025390625ms
  for: 0.044921875ms
  for: 0.0498046875ms

  for...in...: 14.004150390625ms
  for...in...: 17.953125ms
  for...in...: 10.078857421875ms
  for...in...: 10.843994140625ms
  for...in...: 10.205810546875ms
  for...in...: 9.93798828125ms
  for...in...: 11.42626953125ms
  for...in...: 10.885986328125ms
  for...in...: 9.726806640625ms
  for...in...: 10.239013671875ms

  for...of...: 5.726318359375ms
  for...of...: 10.739990234375ms
  for...of...: 0.39697265625ms
  for...of...: 0.39404296875ms
  for...of...: 0.358154296875ms
  for...of...: 0.39013671875ms
  for...of...: 0.313232421875ms
  for...of...: 0.344970703125ms
  for...of...: 0.341064453125ms
  for...of...: 0.346923828125ms
*/
</script>

```

> 当循环次数10000甚至是100000以上时，这里以100000次为例，发现for循环性能明显高于for...of，而for...of的性能则又高于for...in...。但是在循环10000次以下的时候，又发现for...of的性能略低于for...in...，但是for...of...的性能是有极大优化空间的，所以for...of和for...in还是推荐使用for...of...。


#####  使用for循环仍然可以优化


``` html
<script>
  let arr = new Array(100000).fill(1)

  // for测试 ++测试
  console.time('for ++')
  for(let i=0, len=arr.length; i<len; i++) {
    arr[i]
  }
  console.timeEnd('for ++')

  // for测试 -- 测试
  console.time('for --')
  for(let i=arr.length; i--; ) {
    arr[i]
  }
  console.timeEnd('for --')

/*
for ++: 1.636962890625ms
for --: 1.109130859375ms
*/
</script>

```

> 但是到百万级别次数甚至以上的时候发现居然还是++耗时少，因此这里觉得没有必要对++ --过多执着。


##### 减少迭代次数

循环的性能主要和以下两者有关

- 每次迭代处理的事务
- 迭代的次数

事务可能很难被减少，但是迭代的次数是可以通过代码减少的，一种有效的办法就是使用“Duff装置”（Duff's Device），Duff装置是一种循环体展开技术，它使得一次迭代实际上执行了多次迭代的操作

``` html
<script>
  function duffDevice(arr) {
    let length = arr.length
    let iterations = Math.floor(length / 8)
    let remain = length % 8
    let index = 0
    
    // 执行剩余次数
    if(remain) {
      do {
        arr[index ++]
      } while(--remain)
    } 

    if(!iterations) return
    
    // 8次迭代转换成1次
    do {
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
    } while(--iterations)
  }
</script>

```


``` html
<script>
  function duffDevice(arr) {
    let length = arr.length
    let iterations = Math.floor(length / 8)
    let remain = length % 8
    let index = 0
    // 执行剩余次数
    if(remain) {
      do {
        arr[index ++]
      } while(--remain)
    } 
    if(!iterations) return
    // 8次迭代转换成1次
    do {
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
      arr[index ++]
    } while(--iterations)
  }



let arr = new Array(1000000).fill(1)
// for测试 ++测试
console.time('for ++')
for(let i=0, len=arr.length; i<len; i++) {
  arr[i]
}
console.timeEnd('for ++')

console.time('for ++')
for(let i=0, len=arr.length; i<len; i++) {
  arr[i]
}
console.timeEnd('for ++')

console.time('for ++')
for(let i=0, len=arr.length; i<len; i++) {
  arr[i]
}
console.timeEnd('for ++')

// Duff 测试
console.time('Duff Device')
duffDevice(arr)
console.timeEnd('Duff Device')

console.time('Duff Device')
duffDevice(arr)
console.timeEnd('Duff Device')

console.time('Duff Device')
duffDevice(arr)
console.timeEnd('Duff Device')

/*

for ++: 3.614013671875ms
for ++: 3.067138671875ms
for ++: 2.9521484375ms
Duff Device: 2.632080078125ms
Duff Device: 1.958740234375ms
Duff Device: 0.416015625ms
*/

</script>
```

> 发现使用Duff装置后确实能提升性能。


##### 基于函数的迭代

需要注意类似于forEach等方法，尽管基于函数的迭代提供了更为便利的迭代方法，但仍然比基于循环的迭代要慢，因为每个数组项调用外部方法打来的开销是速度慢的主要原因。


``` html
<script>

let arr = new Array(1000000).fill(1)

// for测试 ++测试
console.time('for ++')
for(let i=0, len=arr.length; i<len; i++) {
  arr[i]
}
console.timeEnd('for ++')

console.time('for ++')
for(let i=0, len=arr.length; i<len; i++) {
  arr[i]
}
console.timeEnd('for ++')

console.time('for ++')
for(let i=0, len=arr.length; i<len; i++) {
  arr[i]
}
console.timeEnd('for ++')


console.time('map')
arr.map(item => item)
console.timeEnd('map')

console.time('map')
arr.map(item => item)
console.timeEnd('map')

console.time('map')
arr.map(item => item)
console.timeEnd('map')


console.time('forEach')
arr.forEach(item => item)
console.timeEnd('forEach')

console.time('forEach')
arr.forEach(item => item)
console.timeEnd('forEach')

console.time('forEach')
arr.forEach(item => item)
console.timeEnd('forEach')

/*
for ++: 4.32177734375ms
for ++: 4.050048828125ms
for ++: 4.538818359375ms
map: 175.8759765625ms
map: 179.677001953125ms
map: 186.087890625ms
forEach: 17.961181640625ms
forEach: 18.033935546875ms
forEach: 17.775146484375ms
*/
</script>

```

### 条件语句

在条件比较多的情况下switch语句比if-else性能高。

> 大多数语言对switch语句的实现采用branch table（分支表）索引来进行优化，同时在js中switch语句比较值采用全等操作符，不会发生类型转换的性能损耗。当判断多余两个离散值时使用switch是更佳选择。

#### 优化if-else

将最可能出现的条件放在最前面，从而减少条件判断次数


### 查找表

if-else和switch比使用查找表慢很多，javascript中可以使用数组和普通对象来构建查找表，特别是在条件语句数量很大的时候。

``` html
<script>
  const value = 3
  switch(value) {
    case 0:
    return 0
    case 1:
    return 1
    case 2:
    return 2
    case 3:
    return 3
    case 4:
    return 4
    case 5:
    return 5
    case 6:
    return 6
    defualt:
    return -1
  }
  // 建立数组作为查找表进行查找操作
  const arr = [0,1,2,3,4,5,6]
  arr[value]
</script>
```

> 整个过程变成数组查询或者对象成员查询，不用书写任何判断语句。

### 递归

递归可能会导致栈溢出，各个浏览器的调用栈大小不同，最常见的栈溢出原因是不正确的终止条件。为了能在浏览器中安全地工作应尽量将递归替换成迭代或者Memoization(Tabulation,主要用于加速程序计算的优化技术，使得函数避免重复演算之前被处理过的参数，而返回已缓存的结果。)

- 使用迭代替换递归
- 使用memoization优化递归性能


## 定时器优化性能

UI线程：大多数浏览器让一个单线程共用于执行JavaScript和更新用户界面。任何时候只能执行一种操作，如果JavaScript代码正在执行，那么用户界面可能无法响应输入，或者UI界面正在更新，那么JavaScript代码无法执行。


### 浏览器UI线程

用于执行JavaScript和更新用户界面的进程通常被称为“浏览器UI线程”。 UI线程的工作基于一个简单的队列系统，任务会被保存在队列中直到进程空闲（空闲了以后会把下一个任务提取出来运行，任务包括执行JavaScript代码、执行UI更新[包括重绘和重排]）。

#### 运行多久合理

单个JavaScript操作花费的总时间最大值不应该超过100ms，超过100ms用户会觉得自己与界面失去联系。

#### 使用定时器让出时间片段

如果有些任务确实需要超过100ms才能执行完毕，那么应该使用时间切片分段完成执行任务，从而在时间切片的空闲中可以使得UI有时间段进行更新（执行JavaScript的同时间断性的让出UI线程的控制权用来更新用户界面）。

> 执行JavaScript(固定的时间段) -> 更新用户界面 -> 执行JavaScript(固定的时间段)-> 更新用户界面 ...

为了在执行耗时极大的JavaScript代码时可以让出UI线程的控制权，可以使用定时器。使用setTimeout或setInterval定时器时，会告诉JavaScript引擎先等待一段时间（这段时间可以进行其他队列任务的操作，包括UI界面更新等），然后添加一个JavaScript任务到UI线程队列。需要注意使用定时器的定时时间只是告知何时被加入UI线程的任务队列，并不是何时去执行这段代码，因此真正执行定时器任务的时间一定是大于等于定时时间。


``` html

<script>
  console.time('setTimeout 10ms')
  setTimeout(() => {
    console.timeEnd('setTimeout 10ms')
  }, 10)

  console.time('setTimeout 2ms')
  setTimeout(() => {
    console.timeEnd('setTimeout 2ms')
  }, 2)

  console.time('setTimeout 0ms')
  setTimeout(() => {
    console.timeEnd('setTimeout 0ms')
  }, 0)

/*
setTimeout 0ms: 16.01416015625ms
setTimeout 2ms: 17.387939453125ms
setTimeout 10ms: 18.31591796875ms
*/
</script>
```

> 真正执行的时间一定是大于定时时间的。



``` html

<script>
  console.time('setTimeout 10ms')
  setTimeout(() => {
    console.timeEnd('setTimeout 10ms')
  }, 10)

  console.time('setTimeout 0ms')
  setTimeout(() => {
    console.timeEnd('setTimeout 0ms')
  }, 0)

  console.time('setTimeout 2ms')
  setTimeout(() => {
    console.timeEnd('setTimeout 2ms')
  }, 2)

  console.time('map')
  let arr = new Array(1000000).fill(1)
  arr.map(item => item)
  console.timeEnd('map')

  /**
  map: 226.447998046875ms
  setTimeout 0ms: 244.458984375ms
  setTimeout 2ms: 244.644287109375ms
  setTimeout 10ms: 244.8408203125ms
  */
</script>
```

> 此时基本上定时器失去了定时能力，因为三个定时任务被加入队列任务后，都必须等待JavaScript的map循环执行完毕后才能执行，由于定时时间已经超时，因此三个定时任务一等到UI线程空闲就立马一个个排队执行，需要注意的是加入队列的执行顺序仍然按照定时时间的长短来决定。


``` html
<script>
  let counter = 0
  console.time('setInterval 10ms')
  let time = setInterval(() => {
    counter ++
    console.timeEnd('setInterval 10ms')
    

    let arr = new Array(1000000).fill(1)
    arr.map(item => item)

    console.time('setInterval 10ms')

    if(counter > 5) {
      clearInterval(time)
    }
  }, 10)
</script>

```

> setInterval函数会重复添加JavaScript任务到UI队列，需要注意不同的地方是如果UI队列中已经存在同一个setInterval函数创建的任务，那么后续相同的任务不会被添加到UI队列中(一定是当前setInterval函数创建的任务执行完毕后才会新添加任务的UI队列)。


### 使用定时器处理数组

- 处理过程不需要同步
- 数据不需要按顺序处理

如果满足以上两者条件，可以使用定时器分解任务（防止大数据量数组计算导致UI队列无法更新视图产生停滞现象）。

> 需要注意UI更新任务的时间如果小于25ms会不够用，因此定时的时长至少应大于25ms。

之前所说的JavaScript代码的执行时长不应该超过100ms，超过100ms用户会觉得自己与界面失去联系。如果需要处理大数据量数组，可以使用定时器保持50ms左右的运行时间，保证不会影响用户体验（50ms的时间足够UI更新视图）。

可以通过记录运行时间来处理


``` html
<script>

  // 每50ms执行一次数组计算，每50ms执行一次视图更新
  // 使用定时器切分时间片
  function timeQueueProcess(items, process, callback) {

    let todo = items.concat()

    // 数组深拷贝
    setTimeout(() => {

      let start = +new Date()
      do{
        console.time('shift')
        process(todo.shift())
        console.timeEnd('shift')
      // 执行时间控制在50ms左右  
      } while(todo.length > 0 && (+new Date() - start < 50))
      
      // 如果此次items没有执行完毕
      // 让出UI队列的控制权给UI更新视图
      // 50ms的时间足够视图更新  
      console.log(todo)
      todo.length ? setTimeout(() => {
        timeQueueProcess(todo, process, callback)
      }, 0) : callback(items)    
    }, 0)
  }


  let array = new Array(100000).fill(1)

  timeQueueProcess(array, (item) => { item }, () => {
    console.log('定时器任务执行完毕')
  })

  let bool = true
  let $button = document.getElementById('button')
  let $div = document.getElementById('div')
  $button.onclick = () => {
    $div.style.backgroundColor = bool ?  'blue' : 'red'
    bool = !bool
  }

</script>
```

> 需要注意这种使用方式是页面上只有一个定时器的时候比较适用，如果有多个定时器容易出现性能问题，因为UI线程只有一个。

### Web Workers

通过Web Workers API可以使代码运行在UI线程之外。JavaScript和UI共享同一个进程，所以可以互相访问，但是Web Workers API由于创建独立的线程，因此和UI线程之间需要通过通信才能传递信息，UI线程中的大数据量处理可以通知Web Workers API进行处理，处理完毕后又可以返回处理结果告知UI线程进行处理，需要注意的是Web Workers API的兼容性很差，只有在IE11中兼容。


## 编程优化

### 避免双重求值

避免使用代码字符串执行脚本语言，例如使用eval()、Function构造函数、setTimeout()和setInterval()。

``` html
<script>
  let num1 = 5
  let num2 = 6
  result = eval("num1 + num2")
  sum = new Function("arg1", "arg2", "return arg1 + arg2")
  setTimeout("sum = num1 + num2", 100)
</script>
```

> 双重求值是一项代价昂贵的操作，比直接包含代码执行速度慢许多。


### 使用位操作

在位操作中，数字被转换为有符号32位格式，每次运算符会直接操作该32位数以得到结果，尽管需要转换，但是比布尔操作相比要快得多。


``` html
<script>
  let arr = [0,1,2,3,4,5,6]
  for(let i=0,len=arr.length; i<len; i++) {
    i%2 ? console.log('奇树', arr[i]) :console.log('偶数', arr[i])
    i&1 ? console.log('奇树', arr[i]) :console.log('偶数', arr[i])
  }
</script>
```


``` html
<script>
  let arr = new Array(10000000).fill(1)
  console.time('%')
  for(let i=0,len=arr.length; i<len; i++) {
    i%2 ? arr[i] : arr[i]
  }
  console.timeEnd('%')

  console.time('&')
  for(let i=0,len=arr.length; i<len; i++) {
    i&1 ? arr[i] : arr[i]
  }
  console.timeEnd('&')

/**
%: 19.138916015625ms
&: 16.6689453125ms
*/
</script>
```


