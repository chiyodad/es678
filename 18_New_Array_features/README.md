#18. New Array features

##18.1 개요
새로운 배열 메서드 :
- `Array.from(arrayLike, mapFunc?, thisArg?)`
- `Array.of(...items)`

새로운 Array.prototype 메서드 :
- Iterating:
  - `Array.prototype.entries()`
  - `Array.prototype.keys()`
  - `Array.prototype.values()`

- 요소 검색:
  - `Array.prototype.find(predicate, thisArg?)`
  - `Array.prototype.findIndex(predicate, thisArg?)`

- `Array.prototype.copyWithin(target, start, end=this.length)`
- `Array.prototype.fill(value, start=0, end=this.length)`


##18.2 새로운 배열 메서드들

###18.2.1 Array.from(arrayLike, mapFunc?, thisArg?)
`Array.from()`의 기본 기능은 다음의 두 경우에 대해 새로운 배열을 반환해주는 역할이다.
- [유사배열객체](http://speakingjs.com/es5/ch18.html#_pitfall_array_like_objects) : length 프로퍼티와 인덱스 처리된 엘리먼트를 지닌 객체. `document.getElementsByClassName()` 등을 통해 얻을 수 있는 DOM 객체가 그 예에 해당한다.
- [이터러블객체](http://exploringjs.com/es6/ch_iteration.html#ch_iteration) : 매 호출시마다 한 개의 엘리먼트를 인출할 수 있는 객체. 문자열과 배열 및 Map, Set 등이 이터러블객체에 해당한다.

다음은 유사배열객체를 배열로 컨버팅하는 예제이다.
```js
const arrayLike = {
  length: 2,
  0: 'a',
  1: 'b'
};
// for-of는 오직 이터러블객체에 대해서만 동작한다.
for (const x of arrayLike) { // TypeError
    console.log(x);
}

const arr = Array.from(arrayLike);
for (const x of arr) { // OK, iterable
    console.log(x);
}
// a
// b
```


####18.2.1.1 Array.from()으로 매핑하기
`Array.from()`의 두 번째 매개변수로 매핑함수를 입력하면 `map()`을 대체할 수 있다.
```js
const spans = document.querySelectorAll('span.name');
const names1 = Array.prototype.map.call(spans, s => s.textContent);
const names2 = Array.from(spans, s => s.textContent);
```
위 예제에서 `document.querySelectorAll()`의 결과 반환된 값은 배열이 아닌 유사배열객체이므로, `map()` 메서드를 바로 적용할 수 없다. 따라서 기존까지는 forEach()메서드 등을 적용하기 위해 유사배열객체를 배열로 전환하거나, call, apply 등을 활용하곤 했다. 이제는 Array.from() 메서드를 통해 이와 같은 중간 과정을 생략할 수 있게 되었다.

####18.2.1.2 Array의 서브클래스에서 호출하는 from()
`Array.from()`의 또다른 사용법은 유사배열객체나 이터러블객체를 Array의 서브클래스의 인스턴스로 설정하는 것이다. 예를 들어 Array에 서브클래스 'MyArray'를 생성하고 어떤 객체를 'MyArray'의 인스턴스로 설정하고자 한다면, 단순히 `MyArray.from()`을 사용하면 된다. 이 방식이 동작하는 이유는 ES6에서 생성자 사이의 상속관계 때문이다(상위 생성자는 하위 생성자의 프로토타입이다).
```js
class MyArray extends Array {
    ···
}
const instanceOfMyArray = MyArray.from(anIterable);
```

또한 from 메서드를 이용하면 map 메서드와 달리 반환되는 배열의 생성자를 지정할 수 있다.
```js
// from() – receiver를 통해 결과값의 생성자를 결정한다.
// 아래의 경우 반환된 배열은 'MyArray'의 인스턴스이다.
const instanceOfMyArray = MyArray.from([1, 2, 3], x => x * x);

// map(): 결과는 언제나 Array의 인스턴스이다.
const instanceOfArray   = [1, 2, 3].map(x => x * x);
```

어떤 패턴들은 배열의 non-static한 내장메서드들을 설정할 수 있게 한다(slice, filter, map 등에 대해 그렇다). 더 자세한 내용은 'Classes' 챕터의 'The species pattern' 섹션에서 확인할 수 있다.


###18.2.2 Array.of(...items)
`Array.of(item_0, item_1, ···)`는 item_0, item_1,···을 인스턴스로 하는 배열을 생성한다.

####18.2.2.1 Array의 서브클래스에서 호출하는 of()
Array 생성자함수는 숫자형 값 하나만 넘길 경우 제대로 동작하지 않는 문제(정수를 넘기면 배열의 length값을 할당하고, 소수를 넘기면 오류를 발생시킴)가 있기 때문에, 배열을 생성할 때는 늘 배열 리터럴로 선언할 필요가 있다.
```js
new Array(3, 11, 8) // [ 3, 11, 8 ]
new Array(3)        // [ , ,  ,]
new Array(3.1)      // RangeError: Invalid array length
```

그렇다면 Array의 서브클래스의 인스턴스에 값을 할당하려면 어떻게 해야 할까? 바로 이 때 Array.of() 메서드가 도움이 된다(Array의 하위 생성자는 Array의 모든 메서드를 상속받는다는 것을 기억하자).

```js
class MyArray extends Array {
    ···
}
console.log(MyArray.of(3, 11, 8) instanceof MyArray); // true
console.log(MyArray.of(3).length === 1); // true
```


##18.3 New Array.prototype methods

###18.3.1 Iterating over Arrays
배열을 이터레이트하는 데에 도움을 주는 메서드들은 다음과 같다 :
- `Array.prototype.entries()`
- `Array.prototype.keys()`
- `Array.prototype.values()`

위 각 메서드들의 결과는 '배열이 아닌' 일련의 값이다. 이들은 이터레이터를 통해 값을 하나씩 반환한다. 예제를 살펴보자. 이터레이터의 컨텐츠를 배열에 담기 위해 Array.from() 메서드를 이용했다.
```js
Array.from(['a', 'b'].keys())
[ 0, 1 ]
> Array.from(['a', 'b'].values())
[ 'a', 'b' ]
> Array.from(['a', 'b'].entries())
[ [ 0, 'a' ],
  [ 1, 'b' ] ]
```

펼침연산자`...`를 이용할 수도 있다.
```js
[...['a', 'b'].keys()]    // [ 0, 1 ]
```


####18.3.1.1 [index, element]쌍에 대한 이터레이션
`entries()` 메서드를 이용하면 for-of 루프 및 destructuring시 편리하게 [index, element] 쌍을 이터레이트할 수 있다.
```js
for (const [index, element] of ['a', 'b'].entries()) {
    console.log(index, element);
}
```

###18.3.2 배열 요소 검색
`Array.prototype.find(predicate, thisArg?)`
콜백(predicate)함수에서 true를 반환하는 요소 중 첫번째 요소를 반환한다. 만약 true인 요소가 없다면 undefined를 반환한다.
```js
[6, -5, 8].find(x => x < 0)    // -5
[6, 5, 8].find(x => x < 0)     // undefined
```

`Array.prototype.findIndex(predicate, thisArg?)`
콜백(predicate)함수에서 true를 반환하는 요소 중 첫번째 요소의 인덱스값을 반환한다. 만약 true인 요소가 없다면 -1을 반환한다.
```js
[6, -5, 8].findIndex(x => x < 0)    // 1
[6, 5, 8].findIndex(x => x < 0)     // -1
```

콜백함수에 지정 가능한 매개변수 및 순서는 다음과 같다.
```js
predicate(element[, index[, array]])
```


####18.3.2.1 findIndex()로 NaN 찾기
`Array.prototype.indexOf()`는 '==='를 통해 검색하기 때문에 NaN을 검색하지 못하는 한계가 있다는 점은 잘 알려진 사실이다.
```js
[NaN].indexOf(NaN)    // -1
```

`Object.is()`와 함께 `findIndex()`를 이용하면 문제없이 잘 동작한다.
```js
[NaN].findIndex(y => Object.is(NaN, y))    // 0
```

elemIs()라는 헬퍼함수를 만들어 보다 일반적인 접근방식을 적용할 수도 있다.
```js
function elemIs(x) { return Object.is.bind(Object, x) }
[NaN].findIndex(elemIs(NaN))    // 0
```


###18.3.3 Array.prototype.copyWithin()

문법은 다음과 같다.
`Array.prototype.copyWithin(target[index], start[index], end[this.length])`
> return : this

배열 내 요소 중 start(인덱스)부터 end(개수)만큼의 요소들을 복사하여 target 인덱스부터 차례로 치환한다. 범위를 초과할 경우에는 가능한 영역(끝)까지만 치환한다.

```js
const arr = [0, 1, 2, 3, 4, 5, 6, 7];
arr.copyWithin(3, 1, 4)    // [ 0, 1, 2, 1, 2, 3, 6, 7]
```


###18.3.4 Array.prototype.fill()

문법은 다음과 같다.
`Array.prototype.fill(value[any], start[index = 0], end[this.length])`
> return : this

배열의 각 요소들을 지정한 값으로 채운다.
```js
const arr = ['a', 'b', 'c'];
arr.fill(7)    // [ 7, 7, 7 ]
arr            // [ 7, 7, 7 ]
```

시작지점과 끝지점을 지정하여 값을 채울 영역을 제한할 수 있다.
```js
['a', 'b', 'c', 'd'].fill(7, 1, 3)    // [ 'a', 7, 7, 'd']
```

##18.4 배열 내의 빈칸
배열 내의 빈칸은 요소가 없음을 의미한다. 예를들어 다음 배열은 인덱스 1의 값이 비어있다. 인덱스 2는 비어있지 않다!
```js
const arr = ['a', , undefined, 'b']
'use strict'
0 in arr    // true
1 in arr    // false
2 in arr    // true
3 in arr    // true
arr[1]      // undefined
```

> ES6는 빈칸을 '존재하지 않는다'고 가정한다(기존 문법과의 호환성이 보장하는 한에서 그러하다). 따라서 빈칸이 성능에 부정적인 영향을 미치지는 않을지를 두고 고민하지 않아도 된다.


###18.4.1 ES6는 빈칸을 undefined 처럼 다룬다.
```js
Array.from(['a', ,'b'])                    // [ 'a', undefined, 'b' ]
[ ,'a'].findIndex(x => x === undefined)    // 0
[...[ ,'a'].entries()]                     // [ [ 0, undefined ], [ 1, 'a' ] ]
```


###18.4.2 빈칸과 배열 조작
####18.4.2.1 Iteration
Array.prototype[Symbol.iterator]에 의해 생성된 이터레이터는 매 빈칸을 마치 undefined인 값이 있는 것처럼 다룬다.
```js
var arr = [, 'a'];
var iter = arr[Symbol.iterator]();
iter.next()    // { value: undefined, done: false }
iter.next()    // { value: 'a', done: false }
```

spread operator(`...`)나 `for-of` 역시 이터레이션 프로토콜을 기반으로 하고 있으므로 빈칸에 대해 마찬가지로 취급한다.

```js
[...[, 'a']]    // [ undefined, 'a' ]

for (const x of [, 'a']) {
  console.log(x);
}
// undefined
// a
```

_대부분의 Array prototype method(filter() 등)은 이터레이션 프로토콜을 사용하지 않음에 주의할 것!_



####18.4.2.2 Array.from()
첫 번째 인자로 이터러블 객체를 지정할 경우 Array.from() 메서드는 이터레이션을 이용하여 이 인자를 배열로 전환한다. 이는 spread operator와 동일한 동작이다.
```js
Array.from([, 'a'])    // [ undefined, 'a' ]
```

Array.from() 메서드는 유사배열객체를 배열로 전환할 수도 있다. 이 때 빈칸은 undefined로 자동 지정된다.
```js
Array.from({1: 'a', length: 2})    // [ undefined, 'a' ]
```


두번째 인자에 함수를 지정하면 Array.prototype.map()과 유사한 동작을 한다.
다만, 빈칸을 undefined로 지정한다는 점이 map()과 다르다.
```js
Array.from([ ,'a'], x => x)        // [ undefined, 'a' ]
Array.from([ ,'a'], (x,i) => i)    // [ 0, 1 ]

[ ,'a'].map(x => x)        // [ , 'a' ]
[ ,'a'].map((x,i) => i)    // [ , 1 ]
```


####18.4.2.3 Array.prototype methods
Array.prototype 메서드는 ES5에서 이미 약간의 변화가 있었다. 예를 들면
- forEach(), filter(), every(), some() : 빈칸을 무시한다.
- map() : 빈칸을 건너뛰면서 그대로 유지한다.
- join(), toString() : 빈칸을 undefined로 취급하고, null과 undefined를 빈문자열로 해석한다.

ES6에서는 여기에 새로운 동작방식이 추가되었다.
- copyWithin()은 빈칸을 복사할 경우 빈 칸을 생성해낸다(즉 필요한 경우엔 요소를 삭제하여 빈칸을 만든다)
- entries(), keys(), values(), find(), findIndex() : 빈칸을 undefined로 취급한다.
- fill() :  빈칸인지 여부를 신경쓰지 않는다.

아래는 Array.prototype 메서드들이 빈칸을 취급하는 형태를 설명한 표이다.

메서드 | 빈칸인식 | input | result |
:---: | :---: | --- | ---
concat | 빈칸 | `['a',,'b'].concat(['c',,'d'])` | `['a',,'b','c',,'d']`
copyWithin *ES6 | 빈칸 | `[,'a','b',,].copyWithin(2,0)` | `[,'a',,'a']`
entries *ES6 | 요소 | `[...[,'a'].entries()]` | `[[0,undefined], [1,'a']]`
every | 무시 | `[,'a'].every(x => x==='a')` | `true`
fill *ES6 | 무관 | `new Array(3).fill('a')` | `['a','a','a']`
filter  | 무시 | `['a',,'b'].filter(x => true)` | `['a','b']`
find *ES6 | 요소 | `[,'a'].find(x => true)` | `undefined`
findIndex *ES6 | 요소 | `[,'a'].findIndex(x => true)` | `0`
forEach | 무시 | `[,'a'].forEach((x,i) => log(i));` | `1`
indexOf | 무시 | `[,'a'].indexOf(undefined)` | `-1`
join | 요소 | `[,'a',undefined,null].join('#')` | `'#a##'`
keys *ES6 | 요소 | `[...[,'a'].keys()]` | `[0,1]`
lastIndexOf | 무시 | `[,'a'].lastIndexOf(undefined)` | `-1`
map | 빈칸 | `[,'a'].map(x => 1)` | `[,1]`
pop | 요소 | `['a',,].pop()` | `undefined`
push | 빈칸 | `new Array(1).push('a')` | `2`
reduce | 무시 | `['#',,undefined].reduce((x,y)=>x+y)` | `'#undefined'`
reduceRight | 무시 | `['#',,undefined].reduceRight((x,y)=>x+y)` | `'undefined#'`
reverse | 빈칸 | `['a',,'b'].reverse()` | `['b',,'a']`
shift | 요소 | `[,'a'].shift()` | `undefined`
slice | 빈칸 | `[,'a'].slice(0,1)` | `[,]`
some  | 무시 | `[,'a'].some(x => x !== 'a')` | `false`
sort  | 빈칸 | `[,undefined,'a'].sort()` | `['a',undefined,,]`
splice | 빈칸 | `['a',,].splice(1,1)` | `[,]`
toString | 요소 | `[,'a',undefined,null].toString()` | `',a,,'`
unshift | 빈칸 | `[,'a'].unshift('b')` | `3`
values *ES6 | 요소 | `[...[,'a'].values()]` | `[undefined,'a']`

Notes:
- ES6 메서드는 메서드명 우측에 '*ES6'라고 표기했다.
- JS는 배열의 마지막 부분에 여러개의 빈칸이 중복되어 있을 경우 맨 마지막 빈칸을 무시한다. => `['a',,].length → 2`

###18.4.3 값이 채워진 배열 생성하기
빈칸을 undefined인 요소로 취급하는 ES6의 새로운 명령어들은 값이 채워진 배열을 생성하는 데에 도움을 준다.

####18.4.3.1 고정값으로 채우기
`Array.prototype.fill()` 메서드는 빈칸을 포함한 배열의 모든 요소들을 같은 값으로 채워준다.
```js
new Array(3).fill(7)    // [ 7, 7, 7 ]
```
`new Array(3)`은 세 개의 빈칸으로 이루어진 배열을 만들고, 다시 `fill(7)`에 의해 빈칸들을 모두 7로 채운 것이다.

####18.4.3.2 오름차순으로 채우기
`Array.prototype.keys()`는 빈칸에 대해서도 키값(인덱스)를 반환해준다. 이 반환값은 이터러블하므로, spread operator를 이용하면 0부터 시작하는 오름차순의 숫자들로 구성된 배열을 만들 수 있다.
```js
[...new Array(3).keys()]    // [ 0, 1, 2 ]
```

####18.4.3.3 계산된 값으로 채우기
`Array.from()`의 두번째 파라미터에 지정한 매핑함수 역시 빈칸을 인지한다. 따라서 Array.from()을 통해 보다 세련되게 값들을 채울 수 있다.
```js
Array.from(new Array(5), (x,i) => i*2)    // [ 0, 2, 4, 6, 8 ]
```

####18.4.3.4 undefined로 채우기
undefined인 값으로 채워진 배열을 얻고자 할 때는 spread operator에 의한 iteration이 빈칸을 undefined로 전환한다는 사실을 기억하자.
```js
[...new Array(3)]    // [ undefined, undefined, undefined ]
```

###18.4.4 빈 칸 제거하기
ES5 메서드인 `filter()`를 이용하면 빈 칸을 제거할 수 있다.
```js
['a',,'c'].filter(() => true)    // [ 'a', 'c' ]
```

> ES6의 이터레이션은 빈 칸을 undefined요소로 전환한다.
```js
[...['a',,'c']]    //[ 'a', undefined, 'c' ]
```

##18.5 concat 메서드에 의해 펼쳐질지 여부에 대한 설정(Symbol.isConcatSpreadable)

`Symbol.isConcatSpreadable`에 true 혹은 false를 지정함으로써 `Array.prototype.concat()` 메서드가 인자로 지정한 객체들을 어떻게 취급하는지를 설정할 수 있다.

###18.5.1 배열의 기본값 : spreading(true)
기본적으로 Array.prototype.concat()은 배열에 대해 펼친 상태로 결과에 반영한다.
```js
const arr1 = ['c', 'd'];
['a', 'b'].concat(arr1, 'e');   // ['a', 'b', 'c', 'd', 'e']
```

`Symbol.isConcatSpreadable`을 false로 지정하면 배열에 대해 펼쳐지지 않은 상태로 결합동작을 수행할 수 있다.
```js
const arr2 = ['c', 'd'];
arr2[Symbol.isConcatSpreadable] = false;
['a', 'b'].concat(arr2, 'e');    // ['a', 'b', ['c','d'], 'e']
```

###18.5.2 유사배열객체에 대한 기본값 : non-spreading(false)
유사배열객체에 대해서는 기본적으로 펼치지 않은 채로 결합을 수행한다.
```js
const arrayLike = {length: 2, 0: 'c', 1: 'd'};
console.log(['a', 'b'].concat(arrayLike, 'e'));
  // ['a', 'b', arrayLike, 'e']

console.log(Array.prototype.concat.call(arrayLike, ['e','f'], 'g'));
  // [arrayLike, 'e', 'f', 'g']
```

`Symbol.isConcatSpreadable`을 true로 지정하면 펼친 상태로 결합을 수행할 수 있다.
```js
arrayLike[Symbol.isConcatSpreadable] = true;
console.log(['a', 'b'].concat(arrayLike, 'e'));
  // ['a', 'b', 'c', 'd', 'e']
console.log(Array.prototype.concat.call(arrayLike, ['e','f'], 'g'));
  // ['c', 'd', 'e', 'f', 'g']
```

_배열 및 유사배열객체 이외의 객체에 대해서는 무조건 non-spreading(false)로 동작한다._

###18.5.3 배열 탐지
`concat()` 메서드는 `Array.isArray()`와 동일한 알고리즘을 사용하여 파라미터가 배열인지 여부를 결정한다. 이 알고리즘은 `Array.prototype`이 프로토타입 체인에 속해있는지 여부와 관계 없이 잘 동작한다. ES5 및 그 이전 시절에는 이와 같이 동작하게 하기 위해 Array의 서브클래스에 핵을 사용했던 점에 비추어보면, 이는 매우 의미있는 변화이다(본 책의 __proto__ 섹션을 보자).
> That is important, because, in ES5 and earlier, hacks were used to subclass Array and those must continue to work (see the section on __proto__ in this book).

```js
const arr = [];
Array.isArray(arr);    // true

Object.setPrototypeOf(arr, null);
Array.isArray(arr)     // true
```


###18.5.4 Symbol.isConcatSpreadable in the standard library
ES6 표준 라이브러리의 어떤 객체도 Symbol.isConcatSpreadable을 키로 하는 프로퍼티를 갖고 있지 않다. 따라서 이 작동구조는 순수하게 browser API와 user code에서만 존재하는 것이다.

결론 :
- Array의 서브클래스는 기본적으로 펼쳐진다 (이들의 인스턴스가 배열이기 때문에)
- Array의 서브클래스는 Symbol.isConcatSpreadable 프로퍼티를 false로 지정함으로써 그의 인스턴스가 concat 동작시에 펼쳐지는 것을 방지할 수 있다.
- 유사배열객체는 Symbol.isConcatSpreadable 프로퍼티가 true일 경우에만 concat() 동작시에 펼쳐지게 된다.
- Typed Arrays는 펼쳐지지 않으며, concat() 메서드를 갖고 있지 않다.

> Symbol.isConcatSpreadable in the ES6 spec
> Array.prototype.concat(), you can see that spreading requires an object to be Array-like (property length plus indexed elements).
> Whether or not to spread an object is determined via the spec operation IsConcatSpreadable(). The last step is the default (equivalent to Array.isArray()) and the property [Symbol.isConcatSpreadable] is retrieved via a normal Get() operation, meaning that it doesn’t matter whether it is own or inherited.

##18.6 배열 원소의 수량적 한계
ES6의 배열은 ES5의 규칙을 그대로 따른다.

Array length l 프로퍼티는 0 ≤ l ≤ 2^32−1 의 범위 내에서만 존재 가능하다.
Array 원소 i는 0 ≤ i < 2^32−1 의 범위 내에서만 존재 가능하다.
반면 Typed Array의 원소는 더 넓은 범위 : 0 ≤ i < 2^53−1 에서 존재 가능하다 (2^53−1 은 Javascript가 안전(정확)하게 표현할 수 있는 가장 큰 정수이다). 

> 다음의 굵은 글씨로 표시된부분은 오타가 아닐까 합니다.
> However, Typed Arrays have a larger range of indices: **0 ≤ i < 2^32−1** (2^53−1 is the largest integer that JavaScript’s floating point numbers can safely represent). That’s why generic Array methods such as push() and unshift() allow a larger range of indices. Range checks appropriate for Arrays are performed elsewhere, whenever length is set.