---
layout:     post
title:      随便聊聊
subtitle:   2017 情人节快乐~ 
date:       2017-02-14
author:     BY
header-img: img/post-bg-2019.jpg
catalog: true
tags:
    - js
    - 技巧
    - CleanCode
---

# JavaScript优雅写if条件语句5大技巧
@(译文)[JavaScript]
>该篇是 Tips to write better Conditionals in JavaScript 的译文，[原文链接](https://dev.to/hellomeghna/tips-to-write-better-conditionals-in-javascript-2189)。

-------------------

[TOC]

### 什么是条件判断
在任何编程语言中，程序需要根据给定的条件做进一步的处理。
比如，在一游戏中，玩家的生命指数降为0了，那么游戏将结束。在一款天气app中，在早晨的时候，app背景是日出；在夜晚的时候，app背景则为星空。在这篇文章中，我将介绍如何写优雅的if条件判断语句。
### 1. Array.includes
对于多条件判断善用Array.includes
比如:
```javascript
 function printAnimals(animal) {
   if (animal === 'dog' || animal === 'cat') {
      console.log(`I have a ${animal}`);
    }
   }

   console.log(printAnimals('dog')); // I have a dog
```
当只有两种animal时，上面的代码看起来没有什么问题。但假如又有其他animal，将不得不在现有的条件中，用 || 运算符追加更多的animal，代码将慢慢变得不简洁及难以维护。

解决方法：

用Array.includes重写上面的条件判断。
```javascript
 function printAnimals(animal) {
   const animals = ['dog', 'cat', 'hamster', 'turtle']; 

   if (animals.includes(animal)) {
     console.log(`I have a ${animal}`);
   }
  }

  console.log(printAnimals('hamster')); // I have a hamster
```
上面的代码片段中，创建了一个animals 数组，将animal从 if 语句中抽取出来了。如果有更多的animal满足条件，只需要将动物放入到animals数组中即可。
也可以将animals 数组声明为全局变量，这样就可以在更多的地方重用animals。通过Array.includes进行条件判断，代码是简洁、通俗易懂、便于维护。
### 2. 提前 exit / return
这是一个使你代码精炼、整洁的有效技巧。我记得当我第一天正式工作的时候，我尝试用提前 exit写条件判断。
让我们回顾一下之前的例子，去添加更多的animal，万一这些动物不是一个简单的string,而是有特定属性的对象。
现在程序要求如下：
- 没有animal，抛出错误
- 打印animal的类型
- 打印animal的名字
- 打印animal的性别就退出程序

```javascript
const printAnimalDetails = animal => {
  let result; // declare a variable to store the final value

  // condition 1: check if animal has a value
  if (animal) {

    // condition 2: check if animal has a type property
    if (animal.type) {

      // condition 3: check if animal has a name property
      if (animal.name) {

        // condition 4: check if animal has a gender property
        if (animal.gender) {
          result = `${animal.name} is a ${animal.gender} ${animal.type};`;
        } else {
          result = "No animal gender";
        }
      } else {
        result = "No animal name";
      }
    } else {
      result = "No animal type";
    }
  } else {
    result = "No animal";
  }

  return result;
};

console.log(printAnimalDetails()); // 'No animal'

console.log(printAnimalDetails({ type: "dog", gender: "female" })); // 'No animal name'

console.log(printAnimalDetails({ type: "dog", name: "Lucy" })); // 'No animal gender'

console.log(
  printAnimalDetails({ type: "dog", name: "Lucy", gender: "female" })
); // 'Lucy is a female dog'
```
你感觉上面的代码写的如何？
它可以正常运行，但是又臭又长，如果没有语法高亮，在排查if语句是否正确闭合，就可能花费很长时间。假如这代码有更复杂的逻辑，无法想象的一大坨 if..else语句！
我们可以用三目运算符、&&重构以上代码。我们用多个return写更加优美的代码。
``` javascript
const printAnimalDetails = ({type, name, gender } = {}) => {
  if(!type) return 'No animal type';
  if(!name) return 'No animal name';
  if(!gender) return 'No animal gender';

// Now in this line of code, we're sure that we have an animal with all //the three properties here.

  return `${name} is a ${gender} ${type}`;
}

console.log(printAnimalDetails()); // 'No animal type'

console.log(printAnimalDetails({ type: dog })); // 'No animal name'

console.log(printAnimalDetails({ type: dog, gender: female })); // 'No animal name'

console.log(printAnimalDetails({ type: dog, name: 'Lucy', gender: 'female' })); // 'Lucy is a female dog'
```
在上面的重构中，运用了解构和默认参数，当我们将undefined 作为实参时，仍然有一个空对象｛｝默认参数值可以被解构。
在平时工作中，可以善用解构和默认参数。
又比如：
```javascript
function printVegetablesWithQuantity(vegetable, quantity) {
  const vegetables = ['potato', 'cabbage', 'cauliflower', 'asparagus'];

  // condition 1: vegetable should be present
   if (vegetable) {
     // condition 2: must be one of the item from the list
     if (vegetables.includes(vegetable)) {
       console.log(`I like ${vegetable}`);

       // condition 3: must be large quantity
       if (quantity >= 10) {
         console.log('I have bought a large quantity');
       }
     }
   } else {
     throw new Error('No vegetable from the list!');
   }
 }

 printVegetablesWithQuantity(null); //  No vegetable from the list!
 printVegetablesWithQuantity('cabbage'); // I like cabbage
 printVegetablesWithQuantity('cabbage', 20); 
 // 'I like cabbage`
 // 'I have bought a large quantity'
```
我们注意到：
- 一个 if/else 语句过滤非法条件
- 3层 if 语句嵌套(条件1,2&3)

一个通用技巧：条件不满足时，就退出程序。
```javascript
function printVegetablesWithQuantity(vegetable, quantity) {

  const vegetables = ['potato', 'cabbage', 'cauliflower', 'asparagus'];

   // condition 1: throw error early
   if (!vegetable) throw new Error('No vegetable from the list!');

   // condition 2: must be in the list
   if (vegetables.includes(vegetable)) {
      console.log(`I like ${vegetable}`);

     // condition 3: must be a large quantity
      if (quantity >= 10) {
        console.log('I have bought a large quantity');
      }
   }
 }
```
上面的代码优化，我们减少了 if 语句嵌套。这种代码风格可读性比较好，尤其当你有很多 if 语句的时候。
通过判断取反和提前退出程序，我们可以进一步减少 if 语句嵌套。看下面的进一步代码优化，留意 condition 2:
```javascript
  function printVegetablesWithQuantity(vegetable, quantity) {

  const vegetables = ['potato', 'cabbage', 'cauliflower', 'asparagus'];

   if (!vegetable) throw new Error('No vegetable from the list!'); 
   // condition 1: throw error early

   if (!vegetables.includes(vegetable)) return; 
   // condition 2: return from the function is the vegetable is not in 
  //  the list 


  console.log(`I like ${vegetable}`);

  // condition 3: must be a large quantity
  if (quantity >= 10) {
      console.log('I have bought a large quantity');
  }
 }
```
对 condition 2 判断取反，上面代码不再有 if 语句嵌套。当我们有大量的判断的时候，并且在某一条件不满足时，就不将进行下一步处理时，判断取反，提前退出技巧是及其有用的。

我们通过 Return Early，减少代码嵌套。但是不要在不适用提前退出的场景下，勉强使用 Return Early。
### 3. 用对象字面量或 Map 代替 Switch 语句
下面的代码示例，我们想输出某种颜色的水果：
```javascript
function printFruits(color) {
  // use switch case to find fruits by color
  switch (color) {
    case 'red':
      return ['apple', 'strawberry'];
    case 'yellow':
      return ['banana', 'pineapple'];
    case 'purple':
      return ['grape', 'plum'];
    default:
      return [];
  }
}

printFruits(null); // []
printFruits('yellow'); // ['banana', 'pineapple']
```
上面的代码是没有问题的，但是有点冗长，我们可以通过对象字面量，用更加简洁的语法实现上述程序：
```javascript
// use object literal to find fruits by color
  const fruitColor = {
    red: ['apple', 'strawberry'],
    yellow: ['banana', 'pineapple'],
    purple: ['grape', 'plum']
  };

function printFruits(color) {
  return fruitColor[color] || [];
}
```
另一种优化，用 Map 实现：
```javascript
// use Map to find fruits by color
  const fruitColor = new Map()
    .set('red', ['apple', 'strawberry'])
    .set('yellow', ['banana', 'pineapple'])
    .set('purple', ['grape', 'plum']);

function printFruits(color) {
  return fruitColor.get(color) || [];
}
```
Map 是自 ES2015引入的 object 类型，Map 可以存放键值对。
使用 Array.filter 也可以实现相同的程序：
```javascript
 const fruits = [
    { name: 'apple', color: 'red' }, 
    { name: 'strawberry', color: 'red' }, 
    { name: 'banana', color: 'yellow' }, 
    { name: 'pineapple', color: 'yellow' }, 
    { name: 'grape', color: 'purple' }, 
    { name: 'plum', color: 'purple' }
];

function printFruits(color) {
  return fruits.filter(fruit => fruit.color === color);
}
```
### 4. 默认参数值和解构
当编写JavaScript代码时，我们总是对值判断是否为 null/undefined ，然后赋一个默认值。如果不判断值是否为 null/undefined，程序很可能编译失败。
```javascript
function printVegetablesWithQuantity(vegetable, quantity = 1) { 
// if quantity has no value, assign 1

  if (!vegetable) return;
    console.log(`We have ${quantity} ${vegetable}!`);
  }

  //results
  printVegetablesWithQuantity('cabbage'); // We have 1 cabbage!
  printVegetablesWithQuantity('potato', 2); // We have 2 potato!
```
假如 vegetable 是一个 object，那么如何赋一个默认参数值？
``` javascript
function printVegetableName(vegetable) { 
    if (vegetable && vegetable.name) {
     console.log (vegetable.name);
   } else {
    console.log('unknown');
   }
 }

 printVegetableName(undefined); // unknown
 printVegetableName({}); // unknown
 printVegetableName({ name: 'cabbage', quantity: 2 }); // cabbage
```
在上面的代码，当传了一个有效的 vegetable 时，我们打印 vegetable 的name，当传了一个无效的实参时，我就打印 unknown。
通过默认参数值和解构，我们可以避免 if (vegetable && vegetable.name) {} 判断：
```javascript
  // destructing - get name property only
  // assign default empty object {}

  function printVegetableName({name} = {}) {
   console.log (name || 'unknown');
 }


 printVegetableName(undefined); // unknown
 printVegetableName({ }); // unknown
 printVegetableName({ name: 'cabbage', quantity: 2 }); // cabbage
```
因为我们仅仅只需要 name 属性 ，我们可以用 { name } 解构这个参数，然后 name 就可以直接作为一个变量在代码中使用，而不再使用 vegetable.name。
我们也将空对象 {} 作为参数的默认值，如果没有默认值的话，运行 printVegetableName(undefined) 将报错 - Cannot destructure property 'name' of 'undefined' as it is undefined，因为 undefined 没有name属性。
### 5. 用 Array.every & Array.some 进行判断
运用 array 方法可以简化代码，下面的代码片段，判断 fruits 颜色是不是都是 red：
```javascript
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  let isAllRed = true;

  // condition: all fruits must be red
  for (let f of fruits) {
    if (!isAllRed) break;
    isAllRed = (f.color == 'red');
  }

  console.log(isAllRed); // false
}
```
这代码看着有点长，我们可以用 Array.every 简化代码：
```javascript
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
  ];

function test() {
  // condition: short way, all fruits must be red
  const isAllRed = fruits.every(f => f.color == 'red');

  console.log(isAllRed); // false
}
```
同理，如果我们想判断 fruits 其中有颜色为 red 的 fruit，可以用 Array.some 一行代码实现：
```javascript
const fruits = [
    { name: 'apple', color: 'red' },
    { name: 'banana', color: 'yellow' },
    { name: 'grape', color: 'purple' }
];

function test() {
  // condition: if any fruit is red
  const isAnyRed = fruits.some(f => f.color == 'red');

  console.log(isAnyRed); // true
}
```
### 总结
越来越冗长的判断，在未来的某一天终将变成shi:hankey:山。从现在开始，让我们一起尝试使用上述5大技巧，写更简洁、更易维护的代码。
### 原文
[Tips to write better Conditionals in JavaScript](https://dev.to/hellomeghna/tips-to-write-better-conditionals-in-javascript-2189)

---------




