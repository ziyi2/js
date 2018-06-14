# 设计模式

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


