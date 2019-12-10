---
layout: post
title: var, let or const 무엇을 써야 할까 ?
categories: [Javascript]
description: var, let or const
keywords: Javascript, ES6, ES5
---

> 모든 예제 코드는 [Github](https://github.com/CYDP0127/javascript-exercises)에서 확인 가능합니다.

좋은 개발자란 무엇이라고 생각하시나요? 정답은 없겠지만 굳이 고르자면 코드를 간결하고 누가 봐도 작성자의 의도를 파악하기 쉽게 작성하는 개발자가 좋은 개발자라고 생각합니다.

개발을 하다보면 종종 잠깐 쓰이다 말 변수인데 새로 선언하기도 귀찮고 특히나 이름 짓기가 너무 어려워서 그냥 하나의 변수를 다용도로 사용하고 싶어 지는 유혹(?)이 올 때가 있습니다. 하지만 이런 식의 변수 사용은 코드 작성자의 의도를  파악하기 어려울 뿐만 아니라 유지 보수에도 많은 시간과 비용이 들게 됩니다.

따라서 이런 상황을 피하기 위해  자바스크립트 ES6에서 부터 제공되는 const 사용을 지향해야 합니다.

### const

> const 변수의 특징은 처음 선언과 동시에 할당이 되면 재 할당이 불가능합니다.

``` javascript
const number = 1;
number = 2;

Output:
>TypeError: Assignment to constant variable.
```
const를 기능적으로 바라보았을 땐 단순히 변수의 재할당이 불가능하다 이지만 const를 사용한다는 뜻은 하나의 변수는 하나의 기능만 한다는 뜻을 내포하고 있습니다.

다른 개발자가 const로 선언한 변수와 변수명을 보면 해당 변수는 하나의 목적을 위해 만들어졌다는 것이 파악되기 때문에 협업과 유지 보수에 있어서 많은 이점을 가져올 수 있습니다.

### const 예외

한번 할당한 const의 경우 값의 수정이 불가능 하지만 예외는 존재합니다

![](/images/posts/common/confused-frog.jpg)

(변경이 안된다면서 이건 또 뭔 소리야 ?)
``` javascript
const person = {
  name: 'Daniel',
  age: 28,
};
person.name = 'Andrew';

const people = []
people.push(person)

console.log(person, people)

Output:
{ name: 'Andrew', age: 28 } [ { name: 'Andrew', age: 28 } ]
```
integer, string 과 같은 JS의 기본 타입을 할당한 경우 변경이 불가능하지만 call by reference 를 사용하는 object나 array 같은 경우는 자유롭게 값을 수정할 수 있습니다.

변수가 선언되고 나면 object나 array의 경우 변수는 해당 값의 메모리 주소를 가지고 있기 때문에 실제로 object내 값의 변화에 대해서는 관여하지 않게 됩니다.

많은 분들이 const가 constant 또는 [immutable](https://developer.mozilla.org/en-US/docs/Glossary/immutable)이라고 생각하지만 위와 같은 성질 때문에 완전한 immutable이라고 말할 수는 없습니다.

### const 를 immutable로 사용하기

ES5부터 지원되는 Object.freeze()를 이용하면 const object를 immutable변수 할당이 가능합니다.
``` javascript
const foo = Object.freeze({ 'bar': 27 });
foo.bar = 42; // TypeError exception
console.log(foo.bar);
```
또는, [immutable.js](https://github.com/immutable-js/immutable-js) 와 같은 라이브러리를 통해 해결 가능합니다.

### 그렇다면 var과 let의 차이는?

항상 const 변수만 사용하면 좋겠지만, const를 사용할 수 없는 경우가 종종 오기도 합니다.

for문 안에서 써야 하는 경우나, 복잡한 수학 공식을 계산하는 경우 등 변수 값이 계속해서 바뀌어야 하는 경우는 let을 사용하면 됩니다.

var 과 let 둘 다 한번 선언하고 여러번 할당이 가능한 변수지만 둘의 가장 큰 차이점은 var은 function scoped 변수이고, let은 block scoped인 변수라는 점 입니다.

예제1
``` javascript
Input:
console.log(x);
var x=5;
console.log(x);

Output:
undefined
5
```
예제1 var의 경우 변수가 선언이 되지 않았지만 호출이 된다면 현재 scope에서 해당 변수명이 존재하는지 찾고 없다면 다시 상위에서, 또 없다면 그 상위에서 찾으며 scope를 한 단계씩 올라가며 찾게 됩니다.

예제2
```javascript
Input:
console.log(x);
let x=5;
console.log(x);

Output:
ReferenceError: x is not defined
```

하지만 예제2 let의 경우 선언과 할당이 이루어진 후에 해당 블록 내에서만 사용이 가능합니다.
```javascript
for(var i=0; i<10; i++){
  console.log(i); // Output: 0, 1, 2, ... 9
}
console.log(i); // Output: 10
const i = 1; // SyntaxError: Identifier 'i' has already been declared


for(let j=0; j<10; j++){
  console.log(j); // Output: 0, 1, 2, ... 9
}
console.log(j); // ReferenceError: j is not defined
const j = 1; // 정상적으로 변수선언 가능
```
var과 let 비교의 또다른 예제 입니다. 

첫번째 for문에서 선언된 var i 는 for문 밖에서도 사용이 가능하며 이후 i 를 선언한 경우 이미 선언된 변수라는 에러 메시지를 출력합니다.

두번째 for문에서 선언된 let의 경우 for문 안에서는 정상 동작하지만 for문을 벗어난 시점 부터 존재하지 않는 변수로 처리됩니다.

let을 사용한다는 뜻은 해당 변수의 값이 변경 될 수 있지만, 변수가 선언된 블록 내에서만 사용 한다는 뜻을 내포하고 있습니다. 따라서 const를 사용할 수 없는 상황이라면 let을 사용하는 것이 좋습니다.

### 끝으로

불가피한 상황이 아닌 이상 항상 const를 사용하는 것이 좋습니다. 추후에 다루어 볼 내용이지만, 함수형 프로그래밍 (Functional Programming) 형태로 코드를 작성하게 되는 경우 변수를 재할당해야 하는 대부분의 상황을 해결 할 수 있습니다. :)

### Reference

[https://medium.com/javascript-scene/javascript-es6-var-let-or-const-ba58b8dcde75](https://medium.com/javascript-scene/javascript-es6-var-let-or-const-ba58b8dcde75)

[https://github.com/goldbergyoni/nodebestpractices#-37-prefer-const-over-let-ditch-the-var](https://github.com/goldbergyoni/nodebestpractices#-37-prefer-const-over-let-ditch-the-var)

[https://www.geeksforgeeks.org/difference-between-var-and-let-in-javascript/](https://www.geeksforgeeks.org/difference-between-var-and-let-in-javascript/)

[https://mathiasbynens.be/notes/es6-const](https://mathiasbynens.be/notes/es6-const)
