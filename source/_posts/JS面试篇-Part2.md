---
title: JS面试篇-Part2
date: 2021-11-29 20:40:28
tags: 原型 原型链 class继承 instanceof
---

JS 本身是基于原型链的语言

<!-- more -->

#### 1.class  

- constructor
- 属性
- 方法

```js
class Student {
    // constructor
    constructor(name, number){
        // this 当前的类
        this.name = name
        this.number = number
        // 属性也可以自定义，不需要传
        this.school = '阳光小学'
    }

    // 定义方法
    sayHi(){
        console.log(
            `姓名： ${this.name} 学号: ${this.number} 学校: ${this.school} `
        );
    }

}

// 通过类 new 对象/实例  也就是说用 new 关键字把这个模板赋给我们需要的变量
let stu1 = new Student('moon','1234')
let stu2 = new Student('sun', '67890')
console.log(stu1.number);
console.log(stu2.name);
stu1.sayHi();

```

#### 2.继承 

- 某些类有相同的属性和方法时，可以抽象出来归为一个父类共同继承

- extends

- super

- 扩展或重写方法

```js
// 父类
class People{
    constructor(name,favorite){
        this.name = name
        this.favorite = favorite
    }
    eat(){
        console.log(`${this.name} likes to eat ${this.favorite}`);
    }
}

// 子类1
// extends 关键字 表示继承于
class Student extends People{
    constructor(name,favorite,number){
    // super 关键字 把这些属性给父类处理 
        super(name, favorite)
        this.number = number
        this.school = "School_A"
    }
    sayHi(){
       console.log( `姓名 ${this.name} 学号 ${this.number}`)
    }
}
// 子类2
class Teacher extends People{
    constructor(name,favorite,major){
        super(name,favorite)
        this.major = major
    }
    teach(){
       console.log( ` ${this.name} 教授 ${this.major}`)
    }
}
// 实例
let stu = new Student('sam','banana','123456')
let teacher = new Teacher('Miss Wang','grape','JavaScript')
 
stu.sayHi()
stu.eat()
teacher.teach()
teacher.eat()
```

#### 3.类型判断 instanceof

`object instanceof constructor`

```js
function log(x){
    console.log(x)
}
// 判断这个变量是不是这个类构建出来的 
 log(stu instanceof People )     //true
 log(stu instanceof Student)   //true
 log(stu instanceof Object)   //true
 log(stu instanceof Teacher)   //false
 log([] instanceof Array)   //true
 log([] instanceof Object)   //true
 log({} instanceof Object)   //true
```

```js
//补充
const a = 1;
console.log( a instanceof Number) // false
console.log(typeof a); // number
const b = new Number(1);
console.log( b instanceof Number) // true
console.log(typeof b) // object
```



#### 4.原型以及原型链

```js
log(stu.__proto__) //People {}
log(Student.prototype) //People {}
log(stu.__proto__ === Student.prototype) // true
log (stu.__proto__.sayHi()) // undefined
log (stu.__proto__.name) //undefined 因为相当于 stu.__proto__ 作为this了，这个this上没有name没有number
```

##### 4.1原型关系

- 每个class都有显式原型（prototype）
- 每个实例都有隐式原型 （__proto__)
- 实例的proto指向class的prototype

图解





自己本身声明的，叫prototype

自己从别人那继承来的属性和方法，叫__proto__

自己所可以使用的全部方法和属性就是自己的 .prototype 属性



##### 4.2 原型执行规则

- 获取实例的属性或者执行方法时，现在自身属性和方法中查找
- 找不到就去隐式原型中找

##### 4.3 原型链

```js
 log(Student.prototype.__proto__) //{}

 log(People.prototype) //{}

 log(Student.prototype.__proto__ === People.prototype) //true
```





#### 5.问题

##### 5.1 如何准确判断一个变量是不是数组？

`	a instanceof Array`

##### 5.2 class的原型本质，如何理解？

- 原型链的图
- 原型的执行规则

###### 5.3 手写简易jquery，考虑插件和扩展性

​	目的：操作 DOM 节点，获取数据

```js
class jQuery{
    constructor(selector){
        const result = document.querySelectorAll(selector)
        const length = result.length
        for(let i = 0;i<length;i++){
            this[i] = result[i]
        }
        this.length = length
        this.selector = selector
    }
    get(index){
        return this[index]
    }
    // 方法遍历元素
    each(fn){
        for(let i = 0;i<this.length;i++){
            const elem = this[i]
            fn(elem)
        }
    }
    on(type,fn){
        return this.each(elem=>{
            elem.addEventListener(type,fn,false)
        })
    }
}
```

























