# 设计模式

## 工厂模式

### 简单工厂模式

简单工厂模式（静态工厂方法）用于创建同一类对象。该模式通过创建一个新对象然后包装增强其属性和功能实现，需要注意的是使用该模式它们的方法不能共享，不能像原型继承那样共享原型方法。

``` javascript
function createPerson(type, name) {
  var person = new Object()
  person.name = name
  person.getName = function() {
    return this.name
  }

  switch(type) {
    case 'father':
      // 父亲差异部分
      break
    case 'mother':
      // 母亲差异部分
      break   
    case 'children':
      // 孩子差异部分
      break
    default:
      break
  }

  return person
}

let person1 = createPerson('father', 'ziyi1')
let person2 = createPerson('children', 'ziyi2')
```

### 工厂方法模式


``` javascript
class Person {
  constructor(type, name, age, job) {
    if(new target !== Person) {
      
    }
  }

  Father(...args) {

  }

  Mother(...args) {

  }

  Children(...args) {

  }
}


```
