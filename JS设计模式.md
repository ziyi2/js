## 类和继承

### ES5中类的继承

#### 类（构造函数）


构造函数的名字通常用作类名，构造函数是类的公有标识

``` javascript
// Person类
function Person(name) {
  // 实例属性
  this.name = name
  // 实例方法
  this.getName = function() {
    return this.name
  }
}

// 类创建的对象叫类的实例对象
var person = new Person('ziyi2')
```

类的实例对象都有一个不可枚举的属性constructor属性指向类（构造函数）

``` javascript
// true
console.log(person.constructor === Person)
```

构造函数是类的公有标识，但是原型是类的唯一标识

``` javascript
// 检测实例对象所在的类 true
// 实际上instanceof运算符并不会检测person是否由Person()构造函数初始化而来
// 而是会检查person是否继承自Person.prototype
console.log(person instanceof Person)
// 继承至原型链顶端的Object类 true
console.log(person instanceof Object)
```

所有实例的实例方法并不是同一个

``` javascript
var person1 = new Person('ziyi1')
// 创建一个实例对象就会创建一个新的对象物理空间 false
console.log(person1.getName === person.getName)
```

如果不用new关键字，Person是一个普通的全局作用域中的函数

``` javascript
Person('ziyi2')
// 全局作用域中调用函数的this指向window全局对象 ziyi2
console.log(window.name)
// ziyi2
console.log(window.getName())
```

注意类的属性和方法和实例的属性和方法的区别

``` javascript
// 类属性
Person.age = 23
// 类方法
Person.getAge = function() {
  return this.age
}
```

构造函数创建实例对象的过程和工厂模式类似

``` javascript
function createPerson(name) {
  var person = new Object()
  person.name = name
  person.getName = function() {
    return this.name
  }
  return person
}

let person1 = createPerson('ziyi1')
let person2 = createPerson('ziyi2')
// false
console.log(person1.getName === person2.getName)
```

> 工厂模式虽然抽象了创建具体对象的过程，解决了创建多个相似对象的问题，但是没有解决对象的识别问题，即如何知道对象的类型，而类（构造函数）创建的实例对象可以识别实例对象对应哪个原型对象（需要注意原型对象是类的唯一标识，当且仅当两个对象继承自同一个原型对象，才属于同一个类的实例，而构造函数并不能作为类的唯一标识）。

构造函数的创建过程

- 创建一个新对象
- 将构造函数的作用域赋给新对象（this新对象）
- 执行构造函数中的代码
- 返回新对象（最终返回的就是new出来的实例对象，因此this指向实例对象）

#### 原型


##### 原型特性


``` javascript
// Person类（构造函数、Function类的实例对象、函数）
function Person(name) {
  this.name = name
}
// Person类是一个Function类的实例 true
// Person继承自Function
console.log(Person instanceof Function)
```

只要创建一个类（构造函数、Function类的实例对象、函数），就会为类创建一个prototype属性，这个属性指向类的原型对象，在默认情况下，原型对象会自动获得一个constructor（构造函数）属性，这个属性对应类本身（构造函数）

```javascript
console.log(Person.prototype.constructor === Person)
```

类的所有实例共享一个原型对象，如果实例对象的方法可以通用，可以通过原型对象共享方法，而不需要为每一个实例对象开辟一个需要使用的实例方法。

``` javascript
// 原型对象的方法
Person.prototype.getName = function() {
  return this.name
}
// Person类的实例对象
var person = new Person('ziyi2')
var person1 = new Person('ziyi1')
// 两个实例对象引用的是同一个原型方法 true
console.log(person1.getName === person.getName)
```

当调用构造函数创建新实例后，该实例内部将包含一个指向构造函数原型对象的[[Prototype]] 内部属性，脚本中没有标准的方式访问[[Prototype]]，但在一些浏览器诸如Firefox、Safari、Chrome在每个对象上都支持属性__proto__，这个引用存在于实例与构造函数的原型对象之间，调用构造函数创建的实例都有[[Prototype]]属性，但是无法访问，可以通过isPrototypeOf()方法来确定原型对象是否对应当前实例对象

``` javascript
// true
console.log(person.__proto__ === Person.prototype)
// true
console.log(Person.prototype.isPrototypeOf(person))
```


 读取原型链的方法和属性时，会向上遍历搜索，首先搜索实例对象本身有没有同名属性和方法，有则返回，如果没有，则继续搜索实例对象对应的原型对象的方法和属性。

```javascript

// Person类的原型对象的属性
// 需要注意原型对象具有动态性
// 先创造实例对象后修改原型对象也能够立即生效
Person.prototype.age = 28
// 获取原型对象的属性 28
console.log(person.age)
// 检测属性是否存在于实例对象中 false
console.log(person.hasOwnProperty('age'))
// 给实例对象赋值属性
person.age = 11
// 获取了实例对象的属性 11
console.log(person.age)
// 检测属性是否存在于实例对象中 true
console.log(person.hasOwnProperty('age'))
// 删除实例对象的属性
delete person.age
// 仍然可以重新获取原型对象的属性 28
console.log(person.age)
// 检测属性是否存在于实例对象中 false
console.log(person.hasOwnProperty('age'))


// 判断属性存在于实例对象还是原型对象（也可能属性不存在）
function hasPrototypePrpperty(obj,property){
  // in可以检测到实例对象和原型对象中的属性
  return !obj.hasOwnProperty(property) && (property in obj)
}

// for...in循环 
// 返回所有能够通过对象访问、可枚举的(enumerated)属性，包括实例和原型对象的中的属性
for(let key in person) {
  // name、age
  // name是实例对象的属性
  // age是原型对象的属性
  console.log(key)
}

// Object.keys() 
// 返回所有可枚举的实例对象的属性
// ['name']
console.log(Object.keys(person))
```


创建类的时候默认会给类创建一个prototype属性，是类的原型对象的引用，也可以重写改类的原型对象

``` javascript
function Person(name) {
  this.name = name
}

Person.prototype = {
  getName: function() {
    return this.name
  }
}

let person = new Person('ziyi2')
// 此时Person类原型对象是一个新的对象
// 注意和Person.prototype.getName的区别
// 这个新的对象的constructor属性对应Object类
console.log(Person.prototype.constructor === Object)


Person.prototype = {
  // 使新的原型对象的constructor属性对应Person类
  constructor: Person,
  getName: function() {
    return this.name
  }
}

// true
console.log(Person.prototype.constructor === Person)

// 注意未重写原型对象之前的实例仍然指向未重写前的原型对象
for(let key in person) {
  // name,getName
  console.log(key)
}

let person1 = new Person('ziyi1')

// 重写的原型对象的constructor属性变成了可枚举属性
for(let key in person1) {
  // name,constructor,getName
  console.log(key)
}

// 将constructor属性配置成不可枚举属性
Object.defineProperty(Person.prototype,"constructor",{
  enumerable:false,
  value:Person
})

for(let key in person1) {
  // name,getName
  console.log(key)
}
```

##### 原型的弊端


原型对象的基本类型数据的属性（存放的是具体的值，因此每个实例对象的该属性值的改变互不影响）的共享对于实例对象而言非常便捷有效，但是原型对象的引用类型属性不同，原型对象的引用类型的属性存放的是一个指针(C语言中的指针的意思，指针存放的是一个地址，并不是存放一个具体的值，因为类似数组等值在一个32bit的物理块中是放不下的，肯定是放在一个连续的物理块中，因此需要一个地址去读取这些连续的物理块)，指针最终指向的是一个连续的物理块，因此通过原型对象的引用类型修改的值都是改变这个物理块中的值，因此所有实例对象的该指向都会发生变化。



``` javascript
function Person(name) {
  this.name = name
}

Person.prototype = {
  constructor: Person,
  getName: function() {
    return this.name
  },
  age: 28,
  names: ['ziyi', 'ziyi1', 'ziyi2']
}

var person = new Person()
person.names[0] = 'ziyi_modify'
person.age = 11
var person1 = new Person()
// ["ziyi_modify", "ziyi1", "ziyi2"]
console.log(person1.names)
// 28
console.log(person1.age)
```

##### 组合构造函数和原型模式


构造函数定义实例属性，原型对象定义共享的方法和基本数据类型的属性

``` javascript
function Person(name) {
  this.name = name
  this.names = ['ziyi', 'ziyi1', 'ziyi2']
}

Person.prototype = {
  constructor: Person,
  getName: function() {
    return this.name
  }
}

var person = new Person()
person.names[0] = 'ziyi_modify'
var person1 = new Person()
// ['ziyi', 'ziyi1', 'ziyi2']
console.log(person1.names)
```

#### 继承

继承分为接口继承和实现继承，ECMAScript只支持实现继承，实现继承主要依靠原型链。


##### 原型链

假设有两个类（Son类和Father类），Son类对应一个原型对象，通过Son类创建的实例对象都包含一个指向Son类原型对象的内部指针[[Prototype]](大部分浏览器支持实例对象的__proto__属性访问原型对象)。假如让Son类的原型对象引用Father类的实例对象，则Son类的原型对象将包含一个指向Father类的原型对象的内部指针[[Prototype]]，从而使Son类的原型对象可以共享Father类原型对象和实例对象（注意这里也包括实例对象）的方法和属性，从而又使Son类的实例对象可以共享Father类原型对象的方法和属性，这就是原型链。

原型链实现了方法和属性的继承，此时Son类是子类，继承了Father类这个父类。

``` javascript
function Father() {
  this.names = ['ziyi','ziyi1', 'ziyi2']
}
function Son() {}

Father.prototype.getName = function() {
  return Father.name
}

// Son类的原型对象包含了Father类原型对象的方法和属性
// 同时也包含了Father类实例对象的实例方法和实例属性
Son.prototype = new Father()

// 重写了Son类的原型对象
// Son.prototype.constructor !== Son
// true
console.log(Son.prototype.constructor === Father)

let son = new Son()

// 继承Father类实例对象的引用类型属性 
// ["ziyi", "ziyi1", "ziyi2"]
console.log(son.names)

// 继承Father类原型对象的方法
console.log(son.getName())
```

> 读取对象的属性和方法时，会执行搜索，首先搜索实例对象本身有没有同名的属性和方法，有则返回, 如果没有找到，那么继续搜索原型对象，在原型对象中查找具有给定名字的属性和方法。

实现原型链，本质上就是扩展了原型搜索机制

- 搜索实例
- 搜索实例的原型（该原型同时也是另一个类的实例）
- 搜索实例的原型的原型
- ...

一直向上搜索，直到第一次搜索到属性或者方法为止，搜索不到，到原型链的末端停止。


该继承方法有一个缺陷，具体可以查看原型的弊端，在原型中使用引用类型的属性，在所有的实例对象中的该属性都引用了同一个物理空间，一旦空间的值发生了变化，那么所有实例对象的该属性值就发生了变化。

``` javascript
// son实例对象修改Father类实例对象（Son类的原型对象）的引用类型属性
son.names.push('ziyi3')
let son1 = new Son()
// son1中仍然引用的是Father类实例对象（Son类的原型对象）的引用类型属性
// ["ziyi", "ziyi1", "ziyi2", "ziyi3"]
console.log(son1.names)
```


##### 借用构造函数（伪造对象或经典继承）

为了避开原型中有引用类型数据的问题，做到子类继承（这里的继承只是创建和父类相同的实例对象的属性和方法）父类的实例对象的实例方法和实例属性，在子类的构造函数中调用父类的构造函数，从而使子类的this对象在父类构造函数中执行，并最终返回的是子类的this对象（我们知道子类的this对象在构造函数的执行过程中都是开辟新的对象空间，因此引用类型的实例属性都是不同的指针地址）。


``` javascript
function Father() {
  this.names = ['ziyi','ziyi1', 'ziyi2']
}
function Son() {
  // 使用new关键字执行的构造函数this指向实例对象
  // 注意如果不用new关键字执行this指向全局对象window
  // 这里的Father类当做一个普通的执行函数
  // 此时只是让Son类的实例对象创建了和Father类实例对象一样的实例属性和实例方法
  Father.call(this)

  // Father.call(this)类似于在Son构造函数中执行
  // this.names = ['ziyi','ziyi1', 'ziyi2']
}

let son = new Son()
son.names.push('ziyi3')
// ["ziyi", "ziyi1", "ziyi2", "ziyi3"]
console.log(son.names)

let son1 = new Son()
// ['ziyi','ziyi1', 'ziyi2']
console.log(son1.names)
```

如果此时父类有自己的实例属性，而子类也有自己的实例属性


``` javascript
function Father(name,age) {
  this.name = name
  this.age = age
}
function Son() {
  Father.apply(this, arguments)
  this.job = arguments[2]
}

let son = new Son('ziyi2', 28, 'web')
// {name: "ziyi2", age: 28, job: "web"}
console.log(son)
```






### ES6中类的继承

在ES6中,首先创建父类的实例对象this,在子类的构造函数中使用super方法修改父类的this,从而得到与父类同样的实例属性和方法。如果要实现继承，必须在子类的构造函数中调用super方法，否则子类得不到this对象。

``` javascript
class Father {
  constructor(value) {
    this.value = value
  }
}

class Son extends Father {
  constructor(value, key) {
    // super子类的构造函数里表示父类的构造函数
    // 调用父类的constructor(value)
    super(value)
    this.key = key
  }
}

let son = new Son('value', 'key')
// value
console.log(son.value)
```



