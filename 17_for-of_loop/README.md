##17. for-of 루프

###17.1 개요
for-of 는 for-in 과 forEach()를 대체하고 새로운 이터레이션 프로토콜을 지원하는 ES6에 새롭게 추가된 루프이다. 
이터러블 객체(배열, 문자열, Map, Set 등)의 반복을 위해 사용한다. (이터러블과 이터레이터 챕터를 참고하라)
```javascript
const iterable = ['a', 'b'];
for (const x of iterable) {
    console.log(x);
}

// 출력:
// a
// b
```
break 와 continue 도 for-of 루프 안에서 사용가능하다 :
```javascript
for (const x of ['a', '', 'b']) {
    if (x.length === 0) break;
    console.log(x);
}

// 출력:
// a
```
배열의 루프를 돌면서 요소(element)와 인덱스(index)의 접근(the square brackets before of mean that we are using destructuring):
```javascript
const arr = ['a', 'b'];
for (const [index, element] of arr.entries()) {
    console.log(`${index}. ${element}`);
}

// 출력:
// 0. a
// 1. b
```

Looping over the [key, value] entries in a Map (the square brackets before of mean that we are using destructuring):
```javascript
const map = new Map([
    [false, 'no'],
    [true, 'yes'],
]);
for (const [key, value] of map) {
    console.log(`${key} => ${value}`);
}

// 출력:
// false => no
// true => yes
```
###17.2 for-of 루프의 소개
for-of 는 배열, 문자열, Map, Set 과 같은 이터러블 데이터 구조의 루프에 사용 가능합니다. 어떻게 이것이 가능한지는 이터러블과 이터레이터 챕터를 참고하세요. 그러나 for-of 루프를 사용한다면 이 모든것을 다 알 필요는 없습니다.
```javascript
const iterable = ['a', 'b'];
for (const x of iterable) {
    console.log(x);
}

// 출력:
// a
// b
```
for-of는 본문을 실행하기 전에, 이터러블의 아이템들을 루프 한번에 하나씩 찾고 변수 x에 할당합니다. x의 스코프는 루프안이며 오직 이 루프 안에서만 존재합니다.

break 와 continue 도 사용할 수 있습니다.:
```javascript
for (const x of ['a', '', 'b']) {
    if (x.length === 0) break;
    console.log(x);
}

// 출력:
// a
```
for-of 는 아래의 이점들을 결합한다:

일반적인 루프 : break/continue; 제러레이터에서 유용한
forEach() 메소드 : 간결한 문법

###17.3 위험: for-of는 오직 이터러블 값에만 동작한다.
피연산의 주절은 이터러블이어야만 한다. 이 말인즉슨, plain 객체들을 이터레이트 하기 위해서는 helper 함수가 필요하다는 것이다.(Plain objects are not iterable 챕터를 참고하라) 만약 값이 유사 배열이라면 Array.from()을 통해 배열로 변환할 수 있다.

```javascript
// 유사배열, 그치만 이터러블은 아니다!
const arrayLike = { length: 2, 0: 'a', 1: 'b' };

for (const x of arrayLike) { // TypeError
    console.log(x);
}

for (const x of Array.from(arrayLike)) { // OK
    console.log(x);
}
```

###17.4 이터레이션 변수들: let 선언 vs var 선언
만약 let으로 이터레이션 변수를 선언한다면, 각각의 이터레이션을 위한 새로운 바인딩(storage space)이 생성될 것이다.
그것은 화살표 함수를 이용해서 나중을 위해 현재의 바인딩을 저장하는 다음의 코드 스니핏과 같이 생겼다. 이 화살표 함수는 elem 을 위한 똑같은 바인딩을 공유하지 않는다는 것을 알게 될 것이다. 이 각각의 함수는 다른 elem을 갖는다.
```javascript
const arr = [];
for (const elem of [0, 1, 2]) {
    arr.push(() => elem); // save `elem` for later
}
console.log(arr.map(f => f())); // [0, 1, 2]

// `elem` only exists inside the loop:
console.log(elem); // ReferenceError: elem is not defined
```

const 선언은 let선언과 똑같은 방식으로 동작한다.(그러나 바인딩은 immutable 하다)

당신이 이터레이션 변수를 var로 선언한다면 이것들이 어떻게 다른지 살펴보는 것은 유익하다. 이제 모든 화살표 함수는 똑같은 elem을 참조한다.
```javascript
const arr = [];
for (var elem of [0, 1, 2]) {
    arr.push(() => elem);
}
console.log(arr.map(f => f())); // [2, 2, 2]

// `elem` exists in the surrounding function:
console.log(elem); // 2
```
루프를 통해 함수를 만들때라면 각 이터레이션에 하나의 바인딩을 갖는것은 매우 유용하다. (예: 이벤트 리스너 추가)
for또는 for-in 루프에서 let을 쓴다면 역시 매 이터레이션에 하나의 바인딩을 갖는다 variables 챕터에서 설명된다.

###17.5 존재하는 변수의 이터레이팅, 객체 프로퍼티와 배열 요소들
지금까지 선언된 이터레이션 변수의 for-of만 살펴봤다. 그러나 몇가지 다른 유형도 있다.

이미 존재하는 변수로 이터레이트도 가능하다:
```javascript
let x;
for (x of ['a', 'b']) {
    console.log(x);
}
```
객체 프로퍼티로 이터레이트하는 것 또한 가능하다:
```javascript
const obj = {};
for (obj.prop of ['a', 'b']) {
    console.log(obj.prop);
}
```
그리고 배열의 요소로 이터레이트 할 수 있다:
```javascript
const arr = [];
for (arr[0] of ['a', 'b']) {
    console.log(arr[0]);
}
```

###17.6 해체 패턴과 사용되는 이터레이팅
해체와 for-of 를 조합하는 것은 특별히 [key, value] 쌍(encoded ad Arrays)인 이터러블에 유용하다. That’s what Maps are:
```javascript
const map = new Map().set(false, 'no').set(true, 'yes');
for (const [k,v] of map) {
    console.log(`key = ${k}, value = ${v}`);
}
// 출력:
// key = false, value = no
// key = true, value = yes
```
Array.prototype.entries() 또한 [key, value] 를 쌍으로 하는 이터러블을 반환한다:
```javascript
const arr = ['a', 'b', 'c'];
for (const [k,v] of arr.entries()) {
    console.log(`key = ${k}, value = ${v}`);
}
// 출력:
// key = 0, value = a
// key = 1, value = b
// key = 2, value = c
```
그러므로, entries() 는 enumerated items를 다루기 위한 다른 방법을 제공해준다. depending on their position:
Therefore, entries() gives you a way to treat enumerated items differently, depending on their position:

```javascript
/** arr.join(', ') 과 같다 */
function toString(arr) {
    let result = '';
    for (const [i,elem] of arr.entries()) {
        if (i > 0) {
            result += ', ';
        }
        result += String(elem);
    }
    return result;
}
```
이 함수는 아래와 같이 사용된다:

> toString(['eeny', 'meeny', 'miny', 'moe'])
'eeny, meeny, miny, moe'
