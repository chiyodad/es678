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

Access both elements and their indices while looping over an Array (the square brackets before of mean that we are using destructuring):
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
for-of goes through the items of iterable and assigns them, one at a time, to the loop variable x, before it executes the body. The scope of x is the loop, it only exists inside it.

break 와 continue 도 사용할 수 있습니다.:
```javascript
for (const x of ['a', '', 'b']) {
    if (x.length === 0) break;
    console.log(x);
}

// 출력:
// a
```

for-of combines the advantages of:

Normal for loops: break/continue; usable in generators
forEach() methods: concise syntax

###17.3 Pitfall: for-of only works with iterable values
The operand of the of clause must be iterable. That means that you need a helper function if you want to iterate over plain objects (see “Plain objects are not iterable”). If a value is Array-like, you can convert it to an Array via Array.from():

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
그것은 화살표 함수를 이용해서 나중을 위해 현재의 바인딩을 저장하는 다음의 코드 스니핏과 같이 생겼다. 이 화살표 함수는 elem 을 위한 똑같은 바인딩을 공유하지 않는다는 것을 알게 될 것이다. they each have a different one.
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


It is instructive to see how things are different if you var-declare the iteration variable. Now all arrow functions refer to the same binding of elem.
```javascript
const arr = [];
for (var elem of [0, 1, 2]) {
    arr.push(() => elem);
}
console.log(arr.map(f => f())); // [2, 2, 2]

// `elem` exists in the surrounding function:
console.log(elem); // 2
```
Having one binding per iteration is very helpful whenever you create functions via a loop (e.g. to add event listeners).

You also get per-iteration bindings in for loops and for-in loops if you use let. Details are explained in the chapter on variables.

###17.5 Iterating with existing variables, object properties and Array elements
So far, we have only seen for-of with a declared iteration variable. But there are several other forms.

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
해체와 for-of 의 조합은 특별히 [key, value] 쌍(encoded ad Arrays)인 이터러블에 유용하다.
Combining for-of with destructuring is especially useful for iterables over [key, value] pairs (encoded as Arrays). That’s what Maps are:
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
