#해체(Destructuring)

ECMAScript6 에서는 객체나 배열(possibly nested)의 값을 추출하기위한 편리한 방법으로 *해체(destructuring)*를 지원한다. 이번 챕터에서는 해체가 어떻게 동작하는지 설명하고 유용한 예제도 함께 제공한다.

##10.1 개요
해체는 데이터 할당을 받는 곳에서(이를테면 할당 연산자의 좌변), 데이터의 일부를 추출하기 위한 패턴 사용을 가능하게 한다.

##10.1.1 객체 해체(Object destructuring)
객체 해체하기:
```javascript
const obj = { first: 'Jane', last: 'Doe' };
const {first: f, last: l} = obj;
// f = 'Jane'; l = 'Doe'

// {prop} is short for {prop: prop}
const {first, last} = obj;
// first = 'Jane'; last = 'Doe'
```    
    
해체는 반환된 값의 처리를 돕는다 : 
```javascript
const obj = { foo: 123 };

const {writable, configurable} = Object.getOwnPropertyDescriptor(obj, 'foo');

console.log(writable, configurable); // true true
```
##10.1.2 배열해체(Array destructuring)

모든 이터러블은 배열 해체가 가능하다.
```javascript
const iterable = ['a', 'b'];
const [x, y] = iterable;
    // x = 'a'; y = 'b'
```
해체는 반환된 값의 처리를 돕는다 : 

```javascript
const [all, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```

##10.1.3 해체는 어디에서 쓰이는가?
해체는 다음과 같은 곳에서 쓰일 수 있다:
```javascript
// 변수 선언:
const [x] = ['a'];
let [x] = ['a'];
var [x] = ['a'];

// 할당:
[x] = ['a'];

// 매개변수 선언:

function f([x]) { ··· }
f(['a']);
```
해체는 for-of 루프에서도 동작한다:

```javascript
const arr1 = ['a', 'b'];
for (const [index, element] of arr1.entries()) {
    console.log(index, element);
}
// 결과:
// 0 a
// 1 b

const arr2 = [
    {name: 'Jane', age: 41},
    {name: 'John', age: 40},
];

for (const {name, age} of arr2) {
    console.log(name, age);
}
// 결과:
// Jane 41
// John 40
```

##10.2 배경지식: 데이터 생성 vs 데이터 추출

해체가 무엇인지 완벽히 이해하기 위해서 먼저 broader context를 알아보자. 자바스크립트는 데이터 생성을 위한 operations을 가진다 :
```javascript
const obj = {};
obj.first = 'Jane';
obj.last = 'Doe';
```
그리고 데이터 추출을 위한 operations도 가진다:
```javascript
const f = obj.first;
const l = obj.last;
```
우리가 constructing에 사용해오던 똑같은 문법임을 주목하라.

constructing 을 위한 더 나은 문법인 객체 리터럴 :
```javascript
const obj = { first: 'Jane', last: 'Doe' };
```
ECMAScript6의 해체는 데이터 추출을 위한 동일한 문법을 가능케한다. 
Destructuring in ECMAScript 6 enables the same syntax for extracting data, where it is called an object pattern:
```javascript
const { first: f, last: l } = obj;
```
객체 리터럴을 통해 여러개의 프로퍼티를 한 번에 만들 수 있듯이, 객체 해체 패턴도 여러래의 프로퍼티를 한 번에 추출 할 수 있다.  


이러한 패턴들을 통하여 배열을 해체하는 것도 가능하다 :
```javascript
const [x, y] = ['a', 'b']; // x = 'a'; y = 'b'
```

##10.3 패턴(Patterns)


The following two parties are involved in destructuring:

Destructuring source: 해체될 데이터. 예를 들면, 해체 할당식의 우변.

Destructuring target: 해체를 위해 사용되는 패턴. 해체 할당식의 좌변.

destructuring target은 다음중 하나이다.:

Assignment target. 예 : 없음.
변수선언과 매개변수 정의에서는, 오직 변수에 대한 참조만 허용된다. 해체 할당에서 여러가지 옵션이 있지만 나중에 설명하겠다.

객체 패턴. 예: { first: «pattern», last: «pattern» }
객체 패턴의 parts는 프로퍼티이고 이 프로퍼티 값은 다시 패턴이다(재귀적으로).

배열 배턴. 예: [ «pattern», «pattern» ]
배열패턴의 parts는 요소이고 이 요소들은 다시 패턴이다(재귀적으로).

말인즉슨, 얼마든지 깊게 패턴을 응용할 수 있다 :
```javascript
const obj = { a: [{ foo: 123, bar: 'abc' }, {}], b: true };
const { a: [{foo: f}] } = obj; // f = 123
```

##10.3.1 Pick what you need
객체를 해체한다면 오직 프로퍼티만이 관심사이다 :
```javascript
const { x: x } = { x: 7, y: 3 }; // x = 7
```
배열을 해체한다면 prefix추출만을 택할 수 있다.
```javascript
const [x,y] = ['a', 'b', 'c']; // x='a'; y='b';
```
##10.4 패턴이 값의 내부에 접근하는 방법?
할당패턴 = 값, how does the pattern access what’s inside someValue?


##10.4.1 객체 패턴은 객체에 값을 강제한다.
객체 패턴은 프로퍼티에 접근하기 전에 오브젝트에 소스 해체를 강제한다. 그 말인 즉슨 it works with primitive values:
```javascript
const {length : len} = 'abc'; // len = 3
const {toString: s} = 123; // s = Number.prototype.toString
```
##10.4.1.1 Failing to object-destructure a value
The coercion to object is not performed via Object(), but via the internal operation ToObject(). Object() never fails:
```javascript
> typeof Object('abc')
'object'
> var obj = {};
> Object(obj) === obj
true
> Object(undefined)
{}
> Object(null)
{}
```
ToObject() throws a TypeError if it encounters undefined or null. Therefore, the following destructurings fail, even before destructuring accesses any properties:
```javascript
const { prop: x } = undefined; // TypeError
const { prop: y } = null; // TypeError
```
위 결과로써, 값이 객체에 강제되는지 여부를 알기위해 빈 객체 패턴{}을 사용 할 수 있음을 알 수 있다.
As a consequence, you can use the empty object pattern {} to check whether a value is coercible to an object. As we have seen, only undefined and null aren’t:
```javascript
({} = [true, false]); // OK, 배열은 객체에 coercible 하다.
({} = 'abc'); // OK, 문자열은 객체에 coercible 하다.

({} = undefined); // TypeError
({} = null); // TypeError
```javascript
자바스크립트에서 문(statements)은 중괄호로 시작되면 안되기 때문에 표현식을 둘러싸고 있는 소괄호가 필요하다.
The parentheses around the expressions are necessary because statements must not begin with curly braces in JavaScript.

##10.4.2 Array patterns work with iterables
배열해체는 소스의 요소(Elements)를 얻기위해 이터레이터를 사용한다. 그러므로 어떤 값이 이터러블이라면 배열해체가 가능하다.
이터러블 값의 예제를 보자. 

문자열은 이터러블이다 : 
```javascript
const [x,...y] = 'abc'; // x='a'; y=['b', 'c']
```

이터레이터는 code points (“Unicode characters”, 21 bits)를 반환하는 것이지 code units(“JavaScript characters”, 16 bits)를 반환하는게 아님을 유념하라. (자세한 내용은 “Chapter 24. Unicode and JavaScript” in “Speaking JavaScript”. 를 참고하라. 예를 들어 :
```javascript
const [x,y,z] = 'a\uD83D\uDCA9c'; // x='a'; y='\uD83D\uDCA9'; z='c'
```

인덱스로 Set 의 요소에 접근 할 수는 없지만, 이터레이터를 이용하면 접근이 가능하다. 그러므로 Set도 배열해체가 가능하다. :
```javascript
const [x,y] = new Set(['a', 'b']); // x='a'; y='b’;
```
Set 이터레이터는 요소가 삽입된 그 위치에서 순서대로 요소(elements)를 리턴한다. 이것이 이전의 해체 결과가 항상 같은 이유이다.

무한수열. 해체는 무한 수열에도 동작한다. 제너레이터 allNaturalNumbers() 는 0,1,2.... 을 yield하는 이터레이터를 반환한다.

```javascript
function* allNaturalNumbers() {
  for (let n = 0; ; n++) {
    yield n;
  }
}
```

무한 수열에서 처음 3개의 요소를 추출하는 해체 코드이다.
```javascript
const [x, y, z] = allNaturalNumbers(); // x=0; y=1; z=2
```

##10.4.2.1 값 배열해체의 실패 Failing to Array-destructure a value
객체를 반환하는 Symbol.iterator가 key인 메소드를 가지고 있다면 이 값은 이터러블이다. 배열해체는 해체하려는 값이 이터러블이 아니면 타입에러를 낸다. :
```javascript
let x;
[x] = [true, false]; // OK, Arrays are iterable
[x] = 'abc'; // OK, strings are iterable
[x] = { * [Symbol.iterator]() { yield 1 } }; // OK, iterable

[x] = {}; // TypeError, empty objecdts are not iterable
[x] = undefined; // TypeError, not iterable
[x] = null; // TypeError, not iterable
```
이 타입에러는 이터러블의 각 요소에 접근하기 이전에도 발생한다. 이 말인 즉슨, 값이 이터러블인지 아닌지 체크하기 위해 빈배열 패턴을 사용 할 수 있다. 
```javascript
[] = {}; // TypeError, empty objects are not iterable
[] = undefined; // TypeError, not iterable
[] = null; // TypeError, not iterable
```

##10.5 매칭되는 부분이 없다면?
Similarly to how JavaScript handles non-existent properties and Array elements, destructuring fails silently if the target mentions a part that doesn’t exist in the source: the interior of the part is matched against undefined. If the interior is a variable that means that the variable is set to undefined:
```javascript
const [x] = []; // x = undefined
const {prop:y} = {}; // y = undefined
```

객체 패턴과 배열 패턴이 undefined로 매치되면 타입에러를 내는것을 기억하라 

##10.5.1 기본값

Default values are a feature of patterns: If a part (an object property or an Array element) has no match in the source, it is matched against:

its default value (if specified)
undefined (otherwise)
That is, providing a default value is optional.

예제를 살펴보자. 다음 해체 구문에서, 인덱스가 0인 요소는 우변과 매치되지 않는다. 그러므로, destructuring continues by matching x against 3, which leads to x being set to 3.
```javascript
const [x=3, y] = []; // x = 3; y = undefined
```
객체 패턴의 기본값 사용도 가능하다:
```javascript
const {foo: x=3, bar: y} = {}; // x = 3; y = undefined
```
##10.5.1.1 undefined 는 기본값을 triggers 한다.
만약 매칭되는 값이 없거나 undefined 인 경우에는 기본값이 사용된다.
Default values are also used if a part does have a match and that match is undefined:
```javascript
const [x=1] = [undefined]; // x = 1
const {prop: y=2} = {prop: undefined}; // y = 2
```
이런 동작을 위한 합리적인 이유는, 파라미터 기본값 섹션인다음 장에서 설명한다. 

##10.5.1.2 Default values are computed on demand
필요한 경우 기본값은 스스로 연산된다. 다시 말하면 이 해체 구문은 :

```javascript
const {prop: y=someFunc()} = someValue;
```
위 코드는 아래와 같다.
```javascript
let y;
if (someValue.prop === undefined) {
    y = someFunc();
} else {
    y = someValue.prop;
}
```

console.log()를 사용해서 이를 관찰 할 수 있다.
```javascript
> function log(x) { console.log(x); return 'YES' }

> const [a=log('hello')] = [];
hello
> a
'YES'

> const [b=log('hello')] = [123];
> b
123
```

두 번째 해체 구문에서 초기값은 triggered 되지 않고 log() 또한 호출되지 않는다.


##10.5.1.3 기본 값은 패턴안의 다른 변수를 참조 할 수 있다.
기본값은 같은 패턴안의 다른 변수를 포함하여 어떤 변수도 참조가능하다 : 
```javascript
const [x=3, y=x] = [];     // x=3; y=3
const [x=3, y=x] = [7];    // x=7; y=7
const [x=3, y=x] = [7, 2]; // x=7; y=2
```


However, order matters: the variables x and y are declared from left to right and produce a ReferenceError if they are accessed before their declaration:
```javascript
const [x=y, y=3] = []; // ReferenceError
```

##10.5.1.4 패턴의 기본값
우리는 변수의 기본값만을 봐왔지만, 기본값은 패턴에도 연관지을 수 있다 :
```javascript
const [{ prop: x } = {}] = [];
```
이게 무엇을 의미하는가? Recall the rule for default values:

소스에서 매칭되는 부분이 없다면 해체는 기본값으로 계속한다.[…]

The element at index 0 has no match, which is why destructuring continues with:
```javascript
const { prop: x } = {}; // x = undefined
```
You can more easily see why things work this way if you replace the pattern { prop: x } with the variable pattern:
```javascript
const [pattern = {}] = [];
```
More complex default values. Let’s further explore default values for patterns. In the following example, we assign a value to x via the default value { prop: 123 }:
```javascript
const [{ prop: x } = { prop: 123 }] = [];
```

Because the Array element at index 0 has no match on the right-hand side, destructuring continues as follows and x is set to 123.
```javascript
const { prop: x } = { prop: 123 };  // x = 123
```

However, x is not assigned a value in this manner if the right-hand side has an element at index 0, because then the default value isn’t triggered.
```javascript
const [{ prop: x } = { prop: 123 }] = [{}];
```
In this case, destructuring continues with:
```javascript
const { prop: x } = {}; // x = undefined
```
Thus, if you want x to be 123 if either the object or the property is missing, you need to specify a default value for x itself:
```javascript
const [{ prop: x=123 } = {}] = [{}];
```
Here, destructuring continues as follows, independently of whether the right-hand side is [{}] or [].
```javascript
const { prop: x=123 } = {}; // x = 123
```

아직 혼란스러운가?
나중에 알고리즘으로써의 관점에서 해체를 살펴보겠다. 이것은 또 다른 인사이트를 줄 것이다.

##10.6 More object destructuring features
##10.6.1 Property value shorthands
Property value shorthands are a feature of object literals: If the value of a property is provided via a variable whose name is the same as the key, you can omit the key. This works for destructuring, too:
```javascript
const { x, y } = { x: 11, y: 8 }; // x = 11; y = 8
```

This declaration is equivalent to:

const { x: x, y: y } = { x: 11, y: 8 };
You can also combine property value shorthands with default values:
```javascript
const { x, y = 1 } = {}; // x = undefined; y = 1
```

##10.6.2 Computed property keys
Computed property keys are another object literal feature that also works for destructuring: You can specify the key of a property via an expression, if you put it in square brackets:
```javascript
const FOO = 'foo';
const { [FOO]: f } = { foo: 123 }; // f = 123
```
Computed property keys allow you to destructure properties whose keys are symbols:

// 심볼키를 가지는 속성의 생성과 해체
const KEY = Symbol();
const obj = { [KEY]: 'abc' };
const { [KEY]: x } = obj; // x = 'abc'

// Array.prototype[Symbol.iterator] 추출
const { [Symbol.iterator]: func } = [];
console.log(typeof func); // function

##10.7 더 많은 배열 해체의 특징
##10.7.1 생략
Elision lets you use the syntax of Array “holes” to skip elements during destructuring:
```javascript
const [,, x, y] = ['a', 'b', 'c', 'd']; // x = 'c'; y = 'd'
```

##10.7.2 나머지 연산자(rest operator) (...)
나머지 연산자는 배열의 각 요소의 추출한 배열가능케 한다. 배열 패턴의 마지막 파라미터로 나머지 연산자를 사용할 수 있다.:
```javascript
const [x, ...y] = ['a', 'b', 'c']; // x='a'; y=['b', 'c']
```
나머지 연산자는 데이터를 추출한다. 펼침 연산자(...) 의 문법도 똑같다. 펼침 연산자는 데이터를 배열 리터럴과 함수 호출에 공헌한다. 이건 다음 챕터에서 설명한다.

만약 연산자가 요소를 찾지 못한다면, 빈배열로 매칭된다. 때문에 undefined나 null은 절대 할당되지 않는다.

```javascript
const [x, y, ...z] = ['a']; // x='a'; y=undefined; z=[]
```
나머지 연산자의 피연산자가 변수일 필요는 없다. 이 역시 패턴을 사용하면 된다. :
```javascript
const [x, ...[y, z]] = ['a', 'b', 'c'];
    // x = 'a'; y = 'b'; z = 'c'
```

The rest operator triggers the following destructuring:

[y, z] = ['b', 'c']
펼침 연산자는  (...) 나머지 연산자와 완전히 똑같이 생겼다. 그러나 나머지 연산자는 함수 호출과 배열 리터럴 안에서만 사용된다.(해체 패턴 안이 아니라)

10.8 변수가 아닌 곳에도 할당가능하다.
해체를 통한 할당이라면, 각 할당 타겟은 객체의 프로퍼티(obj.prop) 참조나 배열의 요소(arr[0]) 참조를 포함하여 일반적인 할당문의 좌변에서 허용되는 모든 것이 될 수 있다. 
```javascript
const obj = {};
const arr = [];

({ foo: obj.prop, bar: arr[0] }) = { foo: 123, bar: true };

console.log(obj); // {prop:123}
console.log(arr); // [true]
```

또한 나머지 연산자(...)를 이용해서 객체 프로퍼티와 배열 요소를 할당 할 수도 있다.

```javascript
const obj = {};
[first, ...obj.prop] = ['a', 'b', 'c'];
    // first = 'a'; obj.prop = ['b', 'c']
```
만약 해체를 이용해서 변수를 선언하거나 파라미터를 정의 한다면, simple identifiers를 사용해야한다. 객체 프로퍼티나 배열의 요소를 참조하면 안된다.


##10.9 해체의 Pitfalls
해체를 사용할 때 2가지 유념해야할 사항이 있다.

중괄호로 선언문을 시작하면 안된다.
You can’t start a statement with a curly brace.

해체문에는 변수 선언이나 변수 할당이 가능하다. 둘 다 동시에 하는 것은 불가능하다.
다음 두 섹션에 자세한 내용이 있다.

##10.9.1 중괄호로 선언문을 시작하지 말아라
코드 블럭이 중괄호로 시작하기 때문에, 선언문은 그렇게 하면 안된다. 이것은 할당문에서 객체 해체를 사용할 때 불편하다 :

{ a, b } = someObject; // SyntaxError
The work-around is to put the complete expression in parentheses:

({ a, b } = someObject); // ok

##10.9.2 이미 존재하는 변수에 선언과 할당을 조합하면 안된다.
Within a destructuring variable declaration, every variable in the source is declared. In the following example, we are trying to declare the variable b and refer to the existing variable f, which doesn’t work.

let f;
···
let { foo: f, bar: b } = someObject;
    // During parsing (before running the code):
    // SyntaxError: Duplicate declaration, f
The fix is to use a destructuring assignment and to declare b beforehand:

let f;
···
let b;
({ foo: f, bar: b }) = someObject;
##10.10 Examples of destructuring
Let’s start with a few smaller examples.

The for-of loop supports destructuring:

const map = new Map().set(false, 'no').set(true, 'yes');
for (const [key, value] of map) {
  console.log(key + ' is ' + value);
}
You can use destructuring to swap values. That is something that engines could optimize, so that no Array would be created.

[a, b] = [b, a];
You can use destructuring to split an Array:
```javascript
const [first, ...rest] = ['a', 'b', 'c'];
    // first = 'a'; rest = ['b', 'c']
```
##10.10.1 Destructuring returned Arrays
Some built-in JavaScript operations return Arrays. Destructuring helps with processing them:
```javascript
const [all, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```
If you are only interested in the groups (and not in the complete match, all), you can use elision to skip the array element at index 0:
```javascript
const [, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```
exec() returns null if the regular expression doesn’t match. Unfortunately, you can’t handle null via default values, which is why you must use the Or operator (||) in this case:
```javascript
const [, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec(someStr) || [];
```
Array.prototype.split() returns an Array. Therefore, destructuring is useful if you are interested in the elements, not the Array:
```javascript
const cells = 'Jane\tDoe\tCTO'
const [firstName, lastName, title] = cells.split('\t');
console.log(firstName, lastName, title);
```

##10.10.2 Destructuring returned objects
Destructuring is also useful for extracting data from objects that are returned by functions or methods. For example, the iterator method next() returns an object with two properties, done and value. The following code logs all elements of Array arr via the iterator iter. Destructuring is used in line A.
```javascript
const arr = ['a', 'b'];
const iter = arr[Symbol.iterator]();
while (true) {
    const {done,value} = iter.next(); // (A)
    if (done) break;
    console.log(value);
}
```

##10.10.3 Array-destructuring iterable values
Array-destructuring works with any iterable value. That is occasionally useful:
```javascript
const [x,y] = new Set().add('a').add('b');
    // x = 'a'; y = 'b'

const [a,b] = 'foo';
    // a = 'f'; b = 'o'
```

##10.10.4 값을 여러번 반환하기

값을 여러번 반환하는 유용함을 알아보기 위해, findElement(a, p)를 구현해보자. 이 함수는 true를 리턴하는 함수 p의 배열에서 첫 번째 요소를 찾는다. 문제는 그 함수가 무엇을 반환해야 하는가 이다. 때때로  Sometimes one is interested in the element itself, sometimes in its index, sometimes in both. The following implementation returns both.
```javascript
function findElement(array, predicate) {
    for (const [index, element] of array.entries()) { // (A)
        if (predicate(element)) {
            return { element, index }; // (B)
        }
    }
    return { element: undefined, index: -1 };
}
```

line A에서, 배열 메소드 entries() 는 [index, element] 쌍으로 가진 이터러블을 반환한다. 한번의 이터레이션 마다 한 쌍을 해체한다. line B에서 객체  { element: element, index: index } 를 반환하기 위해 속성값 shorthands를 사용한다.

findElement()를 사용해봅시다. 다음 예제에서, 몇몇 ECMAScript 6 특징들은 더 간략한 코딩을 가능하게 해준다.: 콜백은 arrow function 으로, 반환값은 속성값의 shorthands 객체 패턴으로 해체된다
```javascript
const arr = [7, 8, 6];
const {element, index} = findElement(arr, x => x % 2 === 0);
    // element = 8, index = 1
```
속성 키를 참조하는 인덱스와 요소 때문에 우리가 언급한 그 순서는 상관이 없다. 
```javascript
const {index, element} = findElement(···);
```
인덱스와 요소 모두 요구되는 케이스를 성공적으로 다뤘다. 그 것들중에 하나에만 관심이 있었다면 어떨까? 이것은 ECMAScript6에 이르러 해결되었다. 우리의 구현은 이제 그게 가능하다. 그리고 문법 과부하는 하나의 반환값 으로 비교된다.

```javascript
const a = [7, 8, 6];

const {element} = findElement(a, x => x % 2 === 0);
    // element = 8

const {index} = findElement(a, x => x % 2 === 0);
    // index = 1
```

매번 우리가 필요한 속성값을 추출한다 

##10.11 해체 알고리즘
이번 섹션은 다른관점에서 해체를 바라볼 것이다.: 재귀적 패턴 매칭 알고리즘

이 다른 관점은 특히 기본값을 이해하는데 도움을 준다. 만약 완벽히 이해되지 않는다면 계속 읽어봐라

마지막으로 다음 2개의 함수선언문의 차이점을 설명하기 위한 알고리즘을 사용 할 것이다.
```javascript
function move({x=0, y=0} = {})         { ··· }
function move({x, y} = { x: 0, y: 0 }) { ··· }
```
##10.11.1 알고리즘
해체 할당은 다음과 같이 생겼다.

«pattern» = «value»
값으로 부터 데이터 추출을 위한 패턴을 사용을 원한다. 이제부터 그 방법에 대한 알고리즘을 설명 할 것이다. 이것은 패턴 매칭에 의한 함수형 프로그래밍으로 알려져 있다.(이하 매칭). 이 알고리즘은 값의 패턴매칭이 되는 해체 할당을 위해 ← 를 연산자로 규정하고 이러한 과정이 진행되며 변수에 할당한다.

«pattern» ← «value»

이 알고리즘은 좌측을 가르키는 연산자를 받는 재귀적 법칙에 의해 명시된다. 이 선언적인 표기법은 종종 사용되지만 알고리즘의 스펙을 더 간결하게 만든다. 각각의 법칙은 두 부분으로 나뉜다.

head는 룰에 의해 어떤 피연산자부가 다뤄질지 명시한다.
body는 다음에 무엇을 할지 명시한다.

오직 해체 할당을 위한 알고리즘만 보여준다. 변수 선언을 해체하는 것과와 파라미터선언을 해체하는 것은 비슷하게 작동한다.

고급 특징들을 다루지는 않는다. (계산된 속성 키, 속성값, 할당자로써의 객체 속성 또는 배열요소). 기본만 다룬다.


##10.11.1.1 Patterns
A pattern is either:

A variable: x
객체 패턴: {«properties»}
배열 패턴: [«elements»]
Each of the following sections describes one of these three cases.

##10.11.1.2 변수
(1) x ← value (including undefined and null)
  x = value
##10.11.1.3 객체패턴
(2a) {«properties»} ← undefined
  throw new TypeError();
(2b) {«properties»} ← null
  throw new TypeError();
(2c) {key: «pattern», «properties»} ← obj
  «pattern» ← obj.key
  {«properties»} ← obj
(2d) {key: «pattern» = default_value, «properties»} ← obj
  const tmp = obj.key;
  if (tmp !== undefined) {
      «pattern» ← tmp
  } else {
      «pattern» ← default_value
  }
  {«properties»} ← obj
(2e) {} ← obj
  // 좌변의 빈 프로퍼티, 아무것도 안함
  
##10.11.1.4 배열 패턴
배열 패턴과 이터러블. 배열해체를 위한 알고리즘은 배열 패턴과 이터러블로 시작한다.

(3a) [«elements»] ← non_iterable
assert(!isIterable(non_iterable))
  throw new TypeError();
(3b) [«elements»] ← iterable
assert(isIterable(iterable))
  const iterator = iterable[Symbol.iterator]();
  «elements» ← iterator
Helper function:
```javascript
function isIterable(value) {
    return (value !== null
        && typeof value === 'object'
        && typeof value[Symbol.iterator] === 'function');
}
```
배열 요소와 이터레이터. The algorithm continues with the elements of the pattern (left-hand side of the arrow) and the iterator that was obtained from the iterable (right-hand side of the arrow).

(3c) «pattern», «elements» ← iterator
  «pattern» ← getNext(iterator) // undefined after last item
  «elements» ← iterator
(3d) «pattern» = default_value, «elements» ← iterator
  const tmp = getNext(iterator);  // undefined after last item
  if (tmp !== undefined) {
      «pattern» ← tmp
  } else {
      «pattern» ← default_value
  }
  «elements» ← iterator
(3e) , «elements» ← iterator (hole, elision)
  getNext(iterator); // skip
  «elements» ← iterator
(3f) ...«pattern» ← iterator (always last part!)
```javascript
  const tmp = [];
  for (const elem of iterator) {
      tmp.push(elem);
  }
  ```
  «pattern» ← tmp
  
(3g) ← iterator
  // No elements left, nothing to do
Helper function:
```javascript
function getNext(iterator) {
    const {done,value} = iterator.next();
    return (done ? undefined : value);
}
```
##10.11.2 알고리즘 적용
다음의 함수선언문은 *이름있는 파라미터(named parameters)*를 가지고 있다. 이 기술은 파라미터 핸들링 챕터에서 때때로 options object 라고 불리거나 설명된다. 이 파라미터는 해체를 사용하고 x와 y가 생략될 수 있는 방법의 기본값을 사용한다. 그러나 아래 코드의 마지막 라인에서 보듯이 객체와 함께있는 파라미터도 생략 가능하다. 이 특징은 함수 선언문의 머리에서  {} 를 통해 사용된다.

```javascript
function move1({x=0, y=0} = {}) {
    return [x, y];
}
move1({x: 3, y: 8}); // [3, 8]
move1({x: 3}); // [3, 0]
move1({}); // [0, 0]
move1(); // [0, 0]
```
But why would you define the parameters as in the previous code snippet? Why not as follows – which is also completely legal ES6 code?
```javascript
function move2({x, y} = { x: 0, y: 0 }) {
    return [x, y];
}
```
move1()이 왜 올바른지 알기위해 두 가지 예제를 위한 함수를 사용해보자. 그 전에 우선, 어떻게 파라미터 전달이 매칭에 의해 설명 될수 있는지를 먼저 보자. 

##10.11.2.1 Background: 매칭을 이용한 파라미터 전달
For function calls, formal parameters (inside function definitions) are matched against actual parameters (inside function calls). As an example, take the following function definition and the following function call.
```javascript
function func(a=0, b=0) { ··· }
func(1, 2);
```
파라미터 a와 b는 다음의 해체와 유사한 과정으로 세팅된다.

[a=0, b=0] ← [1, 2]
10.11.2.2 사용하기  move2()
 move2() 해체가 어떻게 이뤄지는지 보자.

예제 1. move2()의 해체 과정:

[{x, y} = { x: 0, y: 0 }] ← []
The only Array element on the left-hand side does not have a match on the right-hand side, which is why {x,y} is matched against the default value and not against data from the right-hand side (rules 3b, 3d):

{x, y} ← { x: 0, y: 0 }
좌변은 
The left-hand side contains property value shorthands, it is an abbreviation for:

{x: x, y: y} ← { x: 0, y: 0 }
이 해체는 아래 내용처럼 두 개의 할당과 같은 결과로 이끈다.(rule 2c, 1):

x = 0;
y = 0;

그러나 이것은 기본값이 사용되는 유일한 경우이다.

예제 2. 함수 호출 move2({z:3})를 보자. 이것은 다음의 해체 과정을 걸친다.:

[{x, y} = { x: 0, y: 0 }] ← [{z:3}]
좌변의 0번 인덱스에 배열 요소가 있다. 그러므로 기본값은 무시되고 다음으로 넘어간다.

{x, y} ← { z: 3 }
That leads to both x and y being set to undefined, which is not what we want.

10.11.2.3 Using move1()
Let’s try move1().

Example 1: move1()

[{x=0, y=0} = {}] ← []
우변의 0번 인덱스의 배열 요소는 없다. 기본값을 사용하지 않는다 
We don’t have an Array element at index 0 on the right-hand side and use the default value (rule 3d):

{x=0, y=0} ← {}

The left-hand side contains property value shorthands, which means that this destructuring is equivalent to:

{x: x=0, y: y=0} ← {}
프로퍼티 x와 y 는 우변과 매칭되지 않는다. 그러므로 기본값이 사용되고 아래와 같은 다음 해체가 수행된다. (rule 2d):

x ← 0
y ← 0
다음의 할당 결과로 이끈다.(rule 1):

x = 0
y = 0
Example 2: move1({z:3})

[{x=0, y=0} = {}] ← [{z:3}]

배열 패턴의 첫 번째 요소는 우변부와 매칭되고, 이 매칭은 해체를 지속하는데 사용됩니다.
The first element of the Array pattern has a match on the right-hand side and that match is used to continue destructuring (rule 3d):

{x=0, y=0} ← {z:3}

첫 번 째 예제처럼 x와 y 프로퍼티는 우변에 없고 default values 가 사용된다:
x = 0
y = 0


##10.11.3 결론 
예제에서는 기본값이 패턴부(객체의 프로퍼티 또는 배열 요소)의 특징임을 증명합니다. 일치하는 부분이 없거나 undefined 라면 기본값이 사용 됩니다. 말인즉슨, 패턴은 기본값과 일치됩니다.

