# 设计模式

## 概念

### 什么是模式

模式是一种可复用的解决方案，可用于解决软件设计中遇到的常见问题。

### 模式的优点


- 复用模式有助于防止在应用程序开发过程中小问题引发大问题
- 模式可以提供通用的解决方案
- 某些模式可以通过避免代码复用减少代码的总体资源占用量
- 模式会使开发沟通更快速
- 模式可以逐步改进

### 设计模式的结构

- 模式名称
- 描述
- 上下文大纲
- 问题陈述
- 解决方案
- 设计
- 实现
- 插图
- 示例
- 辅助条件
- 关系
- 已知的用法
- 讨论

### 反模式

JavaScript中的反模式示例如下

- 在全局上下文中定义大量的变量污染全局命名空间
- 向setTimeout或setInterval传递字符串，而不是函数
- 修改Object类的原型
- 以内联形式使用JavaScript
- 使用document.write

### 设计模式的类别

#### 创建型

专注于处理对象创建机制，以适合给定情况的方式来创建对象。创建对象的基本方法可能导致项目复杂性增加，而创建型模式旨在通过控制创建过程来解决这种问题。

|     设计模式 |   描述   |
| :--------| :------: |
|  Constructor(构造器) |   |
|  Factory(工厂) |   |
|  Abstract(抽象) |   |
|  Prototype(原型) |   |
| Singleton(单例) |   |
| Singleton(单例) |   |
| Builder(生成器) |   |

#### 结构型

结构型与对象组合有关，通常可以用于找出在不同对象之间建立关系的简单方法。这种模式有助于确保在系统某一部分发生变化时，系统的整个结构不需要同时改变，同时对于不适合某一特定目的而改变的系统部分，也能够完成重组。


|     设计模式 |   描述   |
| :--------| :------: |
| Iterator(装饰者) |   |
|  Facade(外观) |   |
| Flyweight(享元) |   |
| Adapter(适配器) |   |
| Proxy(代理) |   |


#### 行为

行为设计模式专注于改善或简化系统中不同对象之间的通信。

|     设计模式 |   描述   |
| :--------| :------: |
|  Iterator(迭代器) |   |
| Mediator(中介者) |   |
|Observer(观察者) |   |
|Visitor(访问者) |   |


## Constructor(构造器)模式

``` javascript
function Person() {}
var person = new Person()

// 带原型的Constructor
Person.prototype.getName = {}
```

## Module(模块)模式

- 对象字面量表示法
- Module模式
- AMD模式
- CommonJS模块
- ECMAScript Harmony模块

### 对象字面量表示法

``` javascript
let person = {
  name: '',
  age: 0,
  getName: function() {},
  getAge: function() {}
}
```

### Module模式

Module模式可以为类提供私有和公有的方法，Module模式会封装一个作用域，从而屏蔽来自全局作用域的特殊部分，使一个单独的对象拥有公有/私有的方法和变量。

Module模式使用闭包封装私有状态和组织，提供一种包装混合公有/私有方法和变量的方式，防止其泄露至全局作用域，防止与全局作用域中的方法和变量发生冲突。通过该模式只需返回一个公有API，而其他的一切则都维持在私有闭包里。

Module模式可以屏蔽处理底层时间逻辑，只暴露供应用程序调用的公有API，该模式返回的是一个对象而不是函数。


``` javascript
// 一个立即执行的匿名函数创建了一个作用域
// 全局作用域无法获取私有变量_counter
var moduleMode = (function() {
  // 私有变量
  var _counter = 0

  // 返回一个公有对象
  return {
    // 公有API
    increment: function() {
      return ++_counter
    },
    // 公有API
    reset: function() {
      _counter = 0
      return _counter
    }
  }
})()

let counter = moduleMode.increment()
console.log(counter)
counter = moduleMode.increment()
console.log(counter)
counter = moduleMode.reset()
console.log(counter)
// _counter is not defined
console.log(_counter) 
```
> Module模式的本质是使用函数作用域来模拟私有变量，在模式内，由于闭包的存在，声明的变量和方法旨在改模式内部可用，但在返回对象上定义的变量和方法，则对外部使用者可用。


Module模式也可用于命名空间

``` javascript
 var Namespace = (function() {
  // 私有变量
  var _counter = 0

  // 私有方法
  var _sayCounter = function() {
    console.log(_counter)
  }

  // 返回一个公有对象
  return {
    // 公有变量
    counter: 10,

    // 公有API
    increment: function() {
       ++_counter
       // 调用私有变量
       _sayCounter()
       return _counter
    },
    // 公有API
    reset: function() {
      _counter = 0
      _sayCounter()
      return _counter
    }
  }
})()

Namespace.increment()
Namespace.increment()
Namespace.reset()
```


### Module模式的变化

#### 引入

可以使jQuery、Underscore作为参数引入模块

``` javascript
var moduleMode = (function($, _) {
  function _method() {
    $('.container').html('test')
  }

  return {
    method: function() {
      _method()
    }
  }
})($,_)

moduleMode.method()
```

#### 引出

``` javascript

let moduleMode = (function() {
  var public = {},
      _private = 'hello'

  function _method() {
    console.log(_private)
  }    

  public.name = 'public'

  public.method = function () {
    _method()
  }

  return public
})()

console.log(moduleMode.name)
```

### Module模式的优缺点

- 优点：整洁、支持私有数据。
- 缺陷：私有数据难以维护（想改变可见性需要修改每一个使用该私有数据的地方），无法为私有成员创建自动化单元测试，开发人员无法轻易扩展私有方法。




## 工厂模式

### 简单工厂模式

简单工厂模式（静态工厂方法）主要用于创建同一类对象。该模式通过创建一个新对象然后包装增强其属性和功能实现，需要注意的是使用该模式它们的方法不能共享，不能像原型继承那样共享原型方法。

``` javascript
function createPerson(type, name) {
  var person
  var person = new Object()
  person.name = name
  person.getName = function() {
    return this.name
  }
  return person
}

let person1 = createPerson('father', 'ziyi1')
let person2 = createPerson('children', 'ziyi2')
```

也可以增强简单工厂模式, 使其可以创建不同类的对象

``` javascript
function createPerson(type, name) {
  
  var person

  switch(type) {
    case 'father':
      // 父亲差异部分
      // 例如 person = new Father(name)
      break
    case 'mother':
      // 母亲差异部分
      // 例如 person = new Mother(name)
      break   
    case 'children':
      // 孩子差异部分
      // 例如 person = new Children(name)
      break
    default:
      break
  }
  return person
}
```

### 工厂方法模式

简单工厂模式创建多类对象相对不够灵活，工厂方法模式可以轻松创建多个类的实例对象

``` javascript
function Person(type, name, age, job) {
  // 安全模式，外部可以不使用new关键字
  if(this instanceof Person) {
    return new this[type](name, age, job)
  } else {
    return new Person(type, name, age, job)
  }
}
 
Person.prototype = {
  constructor: Person,
  // Father类
  Father: function(name, age, job) {
    this.name = name
    this.age = age
    this.job = job
    console.log(`Father: ${name},${age},${job}`)
  },

  // Mother类
  Mother: function(name, age, job) {
    this.name = name
    this.age = age
    this.job = job
    console.log(`Mother: ${name},${age},${job}`)
  },

  // Children类
  Children: function(name, age, job) {
    this.name = name
    this.age = age
    this.job = job
    console.log(`Children: ${name},${age},${job}`)
  }
}


let data = [{
  type: 'Father',
  name: 'ziyi2',
  age: 44,
  job: 'web'
}, {
  type: 'Mother',
  name: 'ziyi2',
  age: 280,
  job: 'web'
}, {
  type: 'Children',
  name: 'ziyi2',
  job: 'web'
}]

for(let person of data) {
  Person(person.type, person.name, person.age, person.job)
}
```

> 在项目中通过对产品类的抽象使其创建业务主要负责用于创建多类产品的实例。

### 抽象工厂模式


抽象类是一种声明但是不能使用的类，JavaScript中有一个保留的关键字abstract对应抽象概念，抽象类在实例化时应该抛出错误。抽象类主要用于定义产品簇，并声明一些必备的方法，如果子类没有重写这些方法，那么子类实例化后调用这些方法时应该抛出错误。

抽象类中定义的方法只是显性的定义一些功能，但没有具体的实现，不能使用抽象类创建真实的使用实例对象。

``` javascript

function objectCreate(o) {
  function F() {}
  F.prototype = o
  return new F()
}
                                                                                                
/** 
 * @Author: zhuxiankang 
 * @Date:   2018-06-14 09:17:50  
 * @Desc:   抽象工厂方法
 * @Parm:   subClass -> 子类
 *          abstractClass -> 抽象类 
 */
function AbstractFactory (subClass, abstractClass) {
  if(typeof AbstractFactory[abstractClass] == 'function') {
    // 注意和寄生组合式继承中var prototype = objectCreate(AbstractFactory[abstractClass].prototype)的区别
    // 这里是不仅继承了抽象类的原型对象的方法
    // 还继承了抽象类的实例对象的方法和属性
    // 继承构造函数中没有继承抽象类的实例对象的方法和属性
    var prototype = objectCreate(new AbstractFactory[abstractClass]())
    prototype.constructor = subClass
    subClass.prototype = prototype
  } else {
    throw new Error(abstractClass + ' undefined!')
  }
}

// Person抽象类
AbstractFactory.Person = function() {
  // 实例属性会被子类继承
  this.type = 'person'
}

// Person抽象类声明的抽象方法(抽象方法不能被Person抽象类的实例使用)
AbstractFactory.Person.prototype = {
  // constructor: AbstractFactory.Person,

  getType: function() {
    return new Error('abstract method "getType" can not be called!')
  },

  getName: function() {
    return new Error('abstract method "getName" can not be called!')
  }
}

// 创建其他抽象类
// AbstractFactory.?.prototype


// 创建继承抽象类的子类
function Father(name) {
  this.name = name
}


function Mother(name) {
  this.name = name
}

// 抽象工厂方法实现对抽象类的继承
AbstractFactory(Father, 'Person')
AbstractFactory(Mother, 'Person')


// 需要注意放在AbstractFactory工厂方法之后
// 因为工厂方法里重写了子类的prototype原型对象
Mother.prototype.getType = function() {
  return this.type
}


var father = new Father('ziyi2')
// person
console.log(father.type) 
// Error: abstract method "getType" can not be called!
console.log(father.getType())

var mother = new Mother('ziyi2')
// person 根据原型链，子类的该方法重写了从父类继承的方法
console.log(mother.getType())
```


> 关于寄生组合式继承请查看[js类和继承](https://ziyi2.github.io/2018/06/05/js%E7%B1%BB%E5%92%8C%E7%BB%A7%E6%89%BF.html#more)。抽象工厂模式中的抽象类创建的不是一个真实的对象实例，而是一个类簇，抽象类指定了类的结构，区别于简单工厂模式创建单一对象，工厂方法模式创建多类对象。不过这种模式应用的并不广泛，因为JavaScript中不支持抽象化创建于虚拟方法。


## 建造者模式

工厂模式可有效的创建可复用的实例对象，关心的是最终创建的对象是什么，不关心创建的过程，因此通过工厂模式得到的都是对象实例或者类簇。建造者模式相对比工厂模式复杂一些，关心的是创建对象的过程，例如之前的工厂模式我们关心的创建一个Person类，创建它的共同基本信息，例如name和age以及job等，但是建造者模式不仅要关注Person类的创建过程，还要关注这个Person更多细节，比如穿什么衣服，兴趣爱好是什么，在创建Person类实例对象的基础上，还可以自由组合其他更多信息。

比如要创建一个人的简历模板，让其可以自由选择展示的信息


``` javascript

// Person类
function Person(name, age) {
  this.name = name
  this.age = age
}

Person.prototype.getName = function() {
  return this.name
}

Person.prototype.getAge = function() {
  return this.age
}

// Job类
function Job(job) {
  switch(job) {
    case 'teacher':
      this.job = '教师'
      this.jobDesc = '数学教师'
      break
    
    case 'doctor':
      this.job = '医生'
      this.jobDesc = '骨科医生'
      break
    
    case 'coder':
      this.job = '程序员'
      this.jobDesc = 'Web前端程序员'
      break

    default:
      this.job = job
      this.jobDesc = '不清楚您的职位的相关描述'
      break
  }
}


Job.prototype.changeJobDesc = function(desc) {
  this.jobDesc = desc
} 

Job.prototype.changeJob = function(job) {
  return this.job = job
}

// Hobby类
function Hobby(hobby) {
  this.hobby = []
  this.hobby.push(hobby)
}

Hobby.prototype.addHobby = function(hobby) {
  this.hobby.concat(hobby)
}

Hobby.prototype.getHobby = function() {
  return this.hobby.split(',')
}

// Skill类
function Skill(skill) {
  this.skill = []
  this.skill.push(skill)
}

// ...

// 简历类
function Resume(name, age) {
  this.person = new Person(name, age)
  this.person.job = new Job()
  this.person.hobby = new Hobby()
  this.person.skill = new Skill()
}

// 创建简历
let resume = new Resume('ziyi2', 28)
console.log(resume)

/*
person : Person {
  age : 28
  hobby: Hobby {
    hobby: Array(1)
  }
  job: Job {
    job: undefined, 
    jobDesc: "不清楚您的职位的相关描述
  }
  name:"ziyi2"
  skill: Skill {
    skill: Array(1)
  }
}  
*/

```
> 创建的对象更复杂，是一个复合对象。


## 原型模式

原型模式可以让多个构造函数对应的实例对象共享同一个原型对象的属性和方法，具体查看[ES5中类的继承](https://ziyi2.github.io/2018/06/05/js%E7%B1%BB%E5%92%8C%E7%BB%A7%E6%89%BF.html#more)。


如果创建实例对象的构造函数相对复杂，耗时较长，此时可以不用new关键字去复制这些基类，可以通过这些对象属性或方法进行复制来实现创造，这是原型模式最初的思想，通过原型继承的特点，首先创建一个原型模式的对象复制方法

``` javascript

// 基于已经存在的模板对象克隆新对象
// 需要注意模板引用类型的属性进行了浅复制
// 因此模板对象中的数组类型数据会被所有实例对象共享引用
function prototypeExtend() {
  var F = function() {},
      args = arguments,
      i = 0,
      len = arguments.length

  for(; i<len; i++) {
    for(var key in arguments[i]) {
      F.prototype[key] = arguments[i][key]
    }
  }   
  return new F()
}


let person = prototypeExtend({
  name: '111',
  age: 28
}, {
  getName: function() {
    return this.name
  }
}, {
  getAge: function() {
    return this.age
  }
})

console.log(person.name)
console.log(person.getName())
```


## 单例模式（单体模式）

单例模式是只允许实例化一次的对象类，单例模式更多的用于命名空间。

``` javascript
// Family命名空间
var Family = {
  data: {
    names: [],
    ages: []
  },

  get: {
    getNames: function() {
      return this.names
    },

    getAges: function() {
      return this.ages
    }
  },

  set: {
    setNames: function(names) {
      this.names = names
    },

    setAges: function(ages) {
      this.ages = ages
    }
  }
}
```

例如模块分明的设计

``` javascript
var Ziyi2 = {
  event: {},
  dom: {},
  style: {},
  data: {}
}
```

也可以创建静态变量，实现static关键字功能

``` javascript

var Family = (function() {
  // 私有变量
  var _names = []

  // 闭包，对外抛出改变私有变量和获取私有变量的方法
  return {
    getNames: function() {
      return _names
    },
    setNames: function(names) {
      _names = names
    }
  }
// 立即执行的匿名函数创建了局部作用域  
})()

Family.setNames(['1','2','3'])
console.log(Family.getNames())
```
>该单例模式在创建Family对象的时候立即执行了匿名函数，因此可以立即得到getNames和setNames方法，因此可以说是立即创建了单例对象。


有的时候单例对象需要被延迟创建（"惰性创建"）

``` javascript


var Family = (function() {

  // 单例引用, 私有属性
  var _instance = null

  // 单例对象
  function Single() {

    // 私有变量
    var _names = []

    // 对外抛出的公有方法, 闭包
    return {
      getNames: function() {
        return _names
      },
      setNames: function(names) {
        _names = names
      }
    }
  }

  // 获取单例对象接口，闭包
  return function() {
    if(!_instance) {
      _instance = Single()
    }
    return _instance
  }
})()


// 此时Family是一个function,并没有创建单例对象
console.log(Family)

// 此时创建了单例对象
console.log(Family())

Family().setNames(['1','2','3'])
console.log(Family().getNames())
```

> 如果只需要被实例化一次，可以使用单例模式而不是其他创建型设计模式（工厂模式、建造者模式、原型模式），从而节省系统资源。




## 外观模式

为复杂的子系统借口提供更高级统一接口，通过这个接口使得对子系统接口的访问更容易，在JavaScript中有时也会用于对底层结构兼容性做统一封装来简化用户使用。

``` javascript
var Browser = {
  event: {
    add: function(dom, type, fn) {
      if(dom.addEventListener) {
        dom.addEventListener(type, fn, false)
      } else if(dom.attachEvent) {
        dom.attachEvent('on'+type, fn)
      } else {
        dom['on'+ type] = fn
      }
    },

    remove: function() {
      // ...
    },

    self: function(event) {
      return event || window.event
    },

    target: function(event) {
      return event.target || event.srcElement
    },

    preventDefault: function(event) {
      let event = this.event.self(event)
      event.preventDefault 
      ? event.preventDefault() 
      : event.returnValue = false
    }
  },

  id: function(dom, id) {
    return dom.getElementById(id)
  },

  html: function(dom, id, html) {
    this.id(dom, id).innerHTML = html
  }
}
```

> 通过外观模式对接口的二次封装隐藏其复杂性，可以简化用户的使用。


## 适配器模式

将一个类（对象）的接口（方法或属性）转化成另外一个接口，从而满足用户的需求。

例如将之前的外观模式适配jQuery库

``` javascript
var Browser = {
  event: {
    add: function(dom, type, fn) {
      $(dom).on(type, fn)
    },

    remove: function(dom, type, fn) {
      $(dom).off(type, fn)
    }

    // ....
  },

  id: function(dom, id) {
    return $(id).get(0)
  }
}
```

> 需要注意适配的时候为了做到兼容参数此时不能变。


除了代码适配，还可以进行参数适配。当一个函数传入参数较多时，很难记住参数的顺序，因此可以通过传入对象来进行参数适配

``` javascript
function fn(name, age, job, desc) {}

function adapter(obj) {
  var _adapter = {
    name: '',
    age: 0,
    job: 'web',
    desc: ''
  }

  // 适配传入的参数
  for(var key in _adapter) {
    _adapter[key] = obj[key] || _adapter[key]
  }
}
```




