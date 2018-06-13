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

> 在项目中通过对产品类的抽象使其创建业务主要负责用于创建多累产品的实例。
