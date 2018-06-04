## 类和继承

### ES5中类的继承

#### 类（构造函数）


构造函数的名字通常用作类名，构造函数是类的公有标识

``` javascript
// Person类（构造函数的名字通常用作类名，构造函数是类的公有标识）
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

类的所有实例共享一个原型对象，如果实例对象的方法可以通用，可以通过原型对象共享方法，而不需要为每一个实例对象开辟一个需要使用的实例方法。

``` javascript
// Person类（构造函数、Function类的实例对象、函数）
function Person(name) {
  this.name = name
}
// Person类是一个Function类的实例 true
console.log(Person instanceof Function)
// 只要创建一个类（构造函数、Function类的实例对象、函数），就会为类创建一个prototype属性，这个属性指向类的原型对象，在默认情况下，原型对象会自动获得一个constructor（构造函数）属性，这个属性对应类本身（构造函数）
console.log(Person.prototype.constructor === Person)

// Person类的原型对象的方法为所有实例对象共享
// 原型是类的唯一标识
Person.prototype.getName = function() {
  return this.name
}
// Person类的实例对象
var person = new Person('ziyi2')
// 通过isPrototypeOf()方法来确定原型对象是否对应当前实例对象 true
console.log(Person.prototype.isPrototypeOf(person))
var person1 = new Person('ziyi1')
// 两个实例对象引用的是同一个原型方法 true
console.log(person1.getName === person.getName)
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



