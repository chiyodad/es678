##17. for-of 반복문

###17.1 개요
for-of 는 for-in 과 forEach()를 대체하고 새로운 이터레이션 프로토콜을 지원하는 ES6에 새롭게 추가된 반복문이다. 
이터러블 객체(배열, 문자열, Map, Set 등)의 반복을 위해 사용한다. (이터러블과 이터레이터 챕터를 참고하라)
```javascript
const iterable = ['a', 'b'];
for (const x of iterable) {
    console.log(x);
}

// Output:
// a
// b
```
break and continue work inside for-of loops:
```javascript
for (const x of ['a', '', 'b']) {
    if (x.length === 0) break;
    console.log(x);
}

// Output:
// a
```
Access both elements and their indices while looping over an Array (the square brackets before of mean that we are using destructuring):
```javascript
const arr = ['a', 'b'];
for (const [index, element] of arr.entries()) {
    console.log(`${index}. ${element}`);
}

// Output:
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

// Output:
// false => no
// true => yes
```
###17.2 Introducing the for-of loop
for-of lets you loop over data structures that are iterable: Arrays, strings, Maps, Sets and others. How exactly iterability works is explained in Chap. “Iterables and iterators”. But you don’t have to know the details if you use the for-of loop:
```javascript
const iterable = ['a', 'b'];
for (const x of iterable) {
    console.log(x);
}

// Output:
// a
// b
```
for-of goes through the items of iterable and assigns them, one at a time, to the loop variable x, before it executes the body. The scope of x is the loop, it only exists inside it.

You can use break and continue:
```javascript
for (const x of ['a', '', 'b']) {
    if (x.length === 0) break;
    console.log(x);
}

// Output:
// a
```
for-of combines the advantages of:

Normal for loops: break/continue; usable in generators
forEach() methods: concise syntax

###17.3 Pitfall: for-of only works with iterable values
The operand of the of clause must be iterable. That means that you need a helper function if you want to iterate over plain objects (see “Plain objects are not iterable”). If a value is Array-like, you can convert it to an Array via Array.from():

```javascript
// Array-like, but not iterable!
const arrayLike = { length: 2, 0: 'a', 1: 'b' };

for (const x of arrayLike) { // TypeError
    console.log(x);
}

for (const x of Array.from(arrayLike)) { // OK
    console.log(x);
}
```

###17.4 Iteration variables: let declarations versus var declarations
If you let-declare the iteration variable, a fresh binding (storage space) will be created for each iteration. That can be seen in the following code snippet where we save the current binding of elem for later, via an arrow function. Afterwards, you can see that the arrow functions don’t share the same binding for elem, they each have a different one.
```javascript
const arr = [];
for (const elem of [0, 1, 2]) {
    arr.push(() => elem); // save `elem` for later
}
console.log(arr.map(f => f())); // [0, 1, 2]

// `elem` only exists inside the loop:
console.log(elem); // ReferenceError: elem is not defined
```

A const declaration works the same way as a let declaration (but the bindings are immutable).

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

You can iterate with an existing variable:
```javascript
let x;
for (x of ['a', 'b']) {
    console.log(x);
}
```
You can also iterate with an object property:
```javascript
const obj = {};
for (obj.prop of ['a', 'b']) {
    console.log(obj.prop);
}
```
And you can iterate with an Array element:
```javascript
const arr = [];
for (arr[0] of ['a', 'b']) {
    console.log(arr[0]);
}
```

###17.6 Iterating with a destructuring pattern
Combining for-of with destructuring is especially useful for iterables over [key, value] pairs (encoded as Arrays). That’s what Maps are:
```javascript
const map = new Map().set(false, 'no').set(true, 'yes');
for (const [k,v] of map) {
    console.log(`key = ${k}, value = ${v}`);
}
// Output:
// key = false, value = no
// key = true, value = yes
```
Array.prototype.entries() also returns an iterable over [key, value] pairs:
```javascript
const arr = ['a', 'b', 'c'];
for (const [k,v] of arr.entries()) {
    console.log(`key = ${k}, value = ${v}`);
}
// Output:
// key = 0, value = a
// key = 1, value = b
// key = 2, value = c
```
Therefore, entries() gives you a way to treat enumerated items differently, depending on their position:

```javascript
/** Same as arr.join(', ') */
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
