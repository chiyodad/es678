# 4. ECMAScript 6의 첫 단계

이 챕터는 당신이 ECMAScript6을 첫 단계를 돕는다. 이것은 쉽게 받아들일 수 있고, ES5를 통한 이 기능을 설명할 ES6 기능 리스트를 만든다.

ES6 코드를 실행하는 방법에 대한 자세한 내용 (모던 자바스크립트 엔진에서, 바벨 등을 통한 ES6으로 부터 ES5로 컴파일링), 이 책을 참고 해라 "Setting up ES6"(이 책을 온라인에서 읽는것은 공짜이다.)

# 4.1 var에서 let/const로
ES5에서 당신은 변수를 var를 통해 선언했다. 이러한 변수는 함수 스코프이고, 그들의 스코프는 함수 가장 안쪽 둘러쌓인 함수 이다. var의  행동은 때때로 혼란스럽다. 예제를 보면:

```javascript
var x = 3;
function func(randomize) {
    if (randomize) {
        var x = Math.random(); // (A) scope: whole function
        return x;
    }
    return x; // accesses the x from line A
}
func(false); // undefined
```
func()는 undefined를 반환은 놀라움을 준다. 당신이 만약 무슨일이 일어났는지 더 면밀하게 반영하기 위해 이 코드를 재작성 한다면 당신은 이유를 볼 수 있다.

```javascript
var x = 3;
function func(randomize) {
    var x;
    if (randomize) {
        x = Math.random();
        return x;
    }
    return x;
}
func(false); // undefined
```
ES6에서 당신은 추가적으로 변수를 let이나 const를 통해서 선언할 수 있다. 이 같은 변수들은 블록 스코프를 갖고, 그들의 스코프는 가장 안쪽에 둘러쌓인 블럭이다. let은 대충 var의 블럭스코프 버전이다. const는 마치 let처럼 동작하지만 값이 변할 수 없는 변수를 생성 한다.

let과 const는 더욱 엄격하게 행동하고, 더 많은 익셉션을 던진다 (예: 당신이 그 선언되기 전에 변수에 접근하면). 블럭 스코프는 코드 조각의 효과가  더욱 지역적으로 유지되도록 돕는다 (이 논증을 위한 다음 섹션을 보아라). 그리고 이것은 함수 스코프보다 더 주요 방법이고 이것은 자바스크립트와 다른 프로그래밍 언어 사이의 이동을 용이하게 한다.

만약 첫 버전에서 당신이 var를 let으로 바꾸면, 당신은 다른 동작을 볼수 있다.:
```javascript
let x = 3;
function func(randomize) {
    if (randomize) {
        let x = Math.random();
        return x;
    }
    return x;
}
func(false); // 3
```

이것은 당신이 존재하는 코드에서 var를 let이나 const로 맹목적으로 변경 할 수 없다는 것을 의미한다. 당신인 리팩토링동안 주의를 기울여야 한다.

내 조언은:

* const를 선호해라. 당신은 변경되지 않는 모든 변수들은 const을 사용하라.
* 반면에 let은 값이 변하는 변수를 위해 사용해라.
* var를 피해라.
 
더 자세한 정보: 챕터 "변수와 스코프".

## 4.2 IIFEs에서 Block으로
ES5에서 만약 당신이 블럭에서 tmp의 제한된 스코프를 원한다면, 당신은 IIFE(즉시 호출된 함수 표현식)으로 불리는 패턴을 사용해야 한다.

```javascript
(function () {  // open IIFE
    var tmp = ···;
    ···
}());  // close IIFE

console.log(tmp); // ReferenceError
```
ECMAScript 6에서, 당신은 블럭과 let선언(또는 const 선언)을 간단하게 사용할 수 있다.
```javascript
{  // open block
    let tmp = ···;
    ···
}  // close block

console.log(tmp); // ReferenceError
```
더 자세한 정보: 섹션 "ES6에서 IIFEs를 피해라"

## 4.3 문자열 결함에서 템플레이트 리터럴로
ES6에서, 자바스크립트는 문자열 보간과 멀티라인 문자열을 위한 리터럴을 마침내 얻었다.

### 4.3.1 문자열 보간
ES5에서 당신이 그 값과 문자열 조각을 결합을 통해 값을 문자열에 넣었다.

```javascript
function printCoord(x, y) {
    console.log('('+x+', '+y+')');
}
```
ES6에서 당신은 템플릿 리터럴을 통해 문자 보간을 사용할 수 있다:
```javascript
function printCoord(x, y) {
    console.log(`(${x}, ${y})`);
}
```
### 4.3.2 멀티라인 문자열
템플릿 리터럴은 멀티라인 문자열을 표현하는데 도움을 준다.

예를 들면, 이것은 ES5에서 당신이 하나를 표현하기 위해 해야하는 것 이다:

```javascript
var HTML5_SKELETON =
    '<!doctype html>\n' +
    '<html>\n' +
    '<head>\n' +
    '    <meta charset="UTF-8">\n' +
    '    <title></title>\n' +
    '</head>\n' +
    '<body>\n' +
    '</body>\n' +
    '</html>\n';
```
만약 당신이 백슬래쉬를 통해 뉴라인을 이스케이프 한다면, 이것은 좀더 좋게 보여진다(그러나 당신은 여전히 명시적으로 뉴라인을 추가해야 한다.):

```javascript
var HTML5_SKELETON = '\
    <!doctype html>\n\
    <html>\n\
    <head>\n\
        <meta charset="UTF-8">\n\
        <title></title>\n\
    </head>\n\
    <body>\n\
    </body>\n\
    </html>';
```
ES6 템플릿 리터럴은 여러줄에 걸쳐 있을 수 있다.:

```javascript
const HTML5_SKELETON = `
    <!doctype html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title></title>
    </head>
    <body>
    </body>
    </html>`;
```
(이 예제는 얼마나 많은 공백이 포함하고 있는지 차이가 나지만 이 경우에는 별 문제가 없다.)

더 자세한 정보: 챕터 "템플릿 리터럴과 태그드 템플릿".

## 4.4 함수표현법에서 애로우 함수로
현재 ES5 코드에서, 당신이 함수 표현식을 사용할 때 this를 주의해야 한다. 다음 예제에서 나는 헬퍼 변수 _this(A줄)를 생성하여 UiComponent의 this를 B줄에서 접근 할수 있다.
```javascript
function UiComponent() {
    var _this = this; // (A)
    var button = document.getElementById('myButton');
    button.addEventListener('click', function () {
        console.log('CLICK');
        _this.handleClick(); // (B)
    });
}
UiComponent.prototype.handleClick = function () {
    ···
};
```
ES6에서 당신은 애로우 함수를 사용하여 this(A줄)를 덮지 않을 수 있다.:
```javascript
function UiComponent() {
    var button = document.getElementById('myButton');
    button.addEventListener('click', () => {
        console.log('CLICK');
        this.handleClick(); // (A)
    });
}
```
(ES6에서 당신은 또한 생성자 함수 대신에 클래스를 사용하는 옵션이 있다. 이것 나중에 살펴보자)

애로우 함수는 단지 표현식의 값을 반환하는 짧은 콜백에 특히 편리하다.

ES5에서, 이런 콜백은 비교적 장황하다:

```javascript
var arr = [1, 2, 3];
var squares = arr.map(function (x) { return x * x });
```
ES6에서, 애로우 함수는 더욱더 간결하다.:

```javascript
const arr = [1, 2, 3];
const squares = arr.map(x => x * x);
```

파라미터를 정의시, 만약 파라미터가 단 하나라면, 당신은 심지어 괄호를 뺄 수 있다. 따라서: (x) => x * x 와 x => x * x 는 둘다 허용된다.

더 자세한 내용은: 챕터 "애로우 함수".

## 4.5 다중 리턴 값 다루기
어떤 함수나 메소드는 다중 값을 배열이나 객체를 통해 반환한다. ES5에서 당신은 다중 값을 원할 때 항상 중간 값 생성이 필요하다. ES6에서는 해체(destructuring)를 통해 중간값 변수를 피할 수 있다.

### 4.5.1 배열을 통한 다중 리턴 값
exec()는 유사배열을 통해 캡쳐된 그룹을 반환한다. ES5에서 당신은 심지어 그룹중에서 원하는 값이 있을때, 중간 변수 (아래 예제의 matchObj)가 필요하다.:
```javascript
var matchObj =
    /^(\d\d\d\d)-(\d\d)-(\d\d)$/
    .exec('2999-12-31');
var year = matchObj[1];
var month = matchObj[2];
var day = matchObj[3];
```
ES6에서는 해체는 이 코드를 단순하게 해준다.:
```javascript
const [, year, month, day] =
    /^(\d\d\d\d)-(\d\d)-(\d\d)$/
    .exec('2999-12-31');
```
배열 패턴의 시작부분의 빈 슬롯은 인덱스가 0인 요소를 건너뛴다.

### 객체를 통한 다중 값 반환
메소드인 Object.getOwnPropertyDescriptor()는 프로퍼티 디스크립터를 반환한다. 디스크립터 객체는 그 프로퍼티의 다양한 값을 갖는다.

ES5에서 심지어 당신이 이 객체 중 프로퍼티에 흥미가 있으면 여전히 중간 변수(아래의 예제 propDesc)가 필요하다.:

```javascript
var obj = { foo: 123 };

var propDesc = Object.getOwnPropertyDescriptor(obj, 'foo');
var writable = propDesc.writable;
var configurable = propDesc.configurable;

console.log(writable, configurable); // true true
```
ES6에서 당신은 해채를 사용할 수 있다.:

```javascript
const obj = { foo: 123 };

const {writable, configurable} =
    Object.getOwnPropertyDescriptor(obj, 'foo');

console.log(writable, configurable); // true true
```
{writable, configurable} 는 아래의 축약이다.:
```javascript
{ writable: writable, configurable: configurable }
```
더 자세한 정보는: 챕터 "해체".

4.6 From for to forEach() to for-of
Prior to ES5, you iterated over Arrays as follows:

var arr = ['a', 'b', 'c'];
for (var i=0; i<arr.length; i++) {
    var elem = arr[i];
    console.log(elem);
}
In ES5, you have the option of using the Array method forEach():

arr.forEach(function (elem) {
    console.log(elem);
});
A for loop has the advantage that you can break from it, forEach() has the advantage of conciseness.

In ES6, the for-of loop combines both advantages:

const arr = ['a', 'b', 'c'];
for (const elem of arr) {
    console.log(elem);
}
If you want both index and value of each array element, for-of has got you covered, too, via the new Array method entries() and destructuring:

for (const [index, elem] of arr.entries()) {
    console.log(index+'. '+elem);
}
More information: Chap. “The for-of loop”.

4.7 Handling parameter default values
In ES5, you specify default values for parameters like this:

function foo(x, y) {
    x = x || 0;
    y = y || 0;
    ···
}
ES6 has nicer syntax:

function foo(x=0, y=0) {
    ···
}
An added benefit is that in ES6, a parameter default value is only triggered by undefined, while it is triggered by any falsy value in the previous ES5 code.

More information: section “Parameter default values”.

4.8 Handling named parameters
A common way of naming parameters in JavaScript is via object literals (the so-called options object pattern):

selectEntries({ start: 0, end: -1 });
Two advantages of this approach are: Code becomes more self-descriptive and it is easier to omit arbitrary parameters.

In ES5, you can implement selectEntries() as follows:

function selectEntries(options) {
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    ···
}
In ES6, you can use destructuring in parameter definitions and the code becomes simpler:

function selectEntries({ start=0, end=-1, step=1 }) {
    ···
}
4.8.1 Making the parameter optional
To make the parameter options optional in ES5, you’d add line A to the code:

function selectEntries(options) {
    options = options || {}; // (A)
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    ···
}
In ES6 you can specify {} as a parameter default value:

function selectEntries({ start=0, end=-1, step=1 } = {}) {
    ···
}
More information: section “Simulating named parameters”.

4.9 From arguments to rest parameters
In ES5, if you want a function (or method) to accept an arbitrary number of arguments, you must use the special variable arguments:

function logAllArguments() {
    for (var i=0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}
In ES6, you can declare a rest parameter (args in the example below) via the ... operator:

function logAllArguments(...args) {
    for (const arg of args) {
        console.log(arg);
    }
}
Rest parameters are even nicer if you are only interested in trailing parameters:

function format(pattern, ...args) {
    ···
}
Handling this case in ES5 is clumsy:

function format() {
    var pattern = arguments[0];
    var args = [].slice.call(arguments, 1);
    ···
}
Rest parameters make code easier to read: You can tell that a function has a variable number of parameters just by looking at its parameter definitions.

More information: section “Rest parameters”.

4.10 From apply() to the spread operator (...)
In ES5, you turn arrays into parameters via apply(). ES6 has the spread operator for this purpose.

4.10.1 Math.max()
ES5 – apply():

> Math.max.apply(null, [-1, 5, 11, 3])
11
ES6 – spread operator:

> Math.max(...[-1, 5, 11, 3])
11
4.10.2 Array.prototype.push()
ES5 – apply():

var arr1 = ['a', 'b'];
var arr2 = ['c', 'd'];

arr1.push.apply(arr1, arr2);
    // arr1 is now ['a', 'b', 'c', 'd']
ES6 – spread operator:

const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

arr1.push(...arr2);
    // arr1 is now ['a', 'b', 'c', 'd']
More information: section “The spread operator (...)”.

4.11 From concat() to the spread operator (...)
The spread operator can also turn the contents of its operand into Array elements. That means that it becomes an alternative to the Array method concat().

ES5 – concat():

var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];

console.log(arr1.concat(arr2, arr3));
    // [ 'a', 'b', 'c', 'd', 'e' ]
ES6 – spread operator:

const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];

console.log([...arr1, ...arr2, ...arr3]);
    // [ 'a', 'b', 'c', 'd', 'e' ]
More information: section “The spread operator (...)”.

4.12 From function expressions in object literals to method definitions
In JavaScript, methods are properties whose values are functions.

In ES5 object literals, methods are created like other properties. The property values are provided via function expressions.

var obj = {
    foo: function () {
        ···
    },
    bar: function () {
        this.foo();
    }, // trailing comma is legal in ES5
}
ES6 has method definitions, special syntax for creating methods:

const obj = {
    foo() {
        ···
    },
    bar() {
        this.foo();
    },
}
More information: section “Method definitions”.

4.13 From constructors to classes
ES6 classes are mostly just more convenient syntax for constructor functions.

4.13.1 Base classes
In ES5, you implement constructor functions directly:

function Person(name) {
    this.name = name;
}
Person.prototype.describe = function () {
    return 'Person called '+this.name;
};
In ES6, classes provide slightly more convenient syntax for constructor functions (note especially the compact syntax for method definitions – no keyword function needed):

class Person {
    constructor(name) {
        this.name = name;
    }
    describe() {
        return 'Person called '+this.name;
    }
}
4.13.2 Derived classes
Subclassing is complicated in ES5, especially referring to super-constructors and super-properties. This is the canonical way of creating a sub-constructor of Person, Employee:

function Employee(name, title) {
    Person.call(this, name); // super(name)
    this.title = title;
}
Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;
Employee.prototype.describe = function () {
    return Person.prototype.describe.call(this) // super.describe()
           + ' (' + this.title + ')';
};
ES6 has built-in support for subclassing, via the extends clause:

class Employee extends Person {
    constructor(name, title) {
        super(name);
        this.title = title;
    }
    describe() {
        return super.describe() + ' (' + this.title + ')';
    }
}
More information: chapter “Classes”.

4.14 From custom error constructors to subclasses of Error
In ES5, it is impossible to subclass the built-in constructor for exceptions, Error (the chapter “Subclassing Built-ins” in “Speaking JavaScript” explains why). The following code shows a work-around that gives the constructor MyError important features such as a stack trace:

function MyError() {
    // Use Error as a function
    var superInstance = Error.apply(null, arguments);
    copyOwnPropertiesFrom(this, superInstance);
}
MyError.prototype = Object.create(Error.prototype);
MyError.prototype.constructor = MyError;
In ES6, all built-in constructors can be subclassed, which is why the following code achieves what the ES5 code can only simulate:

class MyError extends Error {
}
More information: section “Subclassing built-in constructors”.

4.15 From objects to Maps
Using the language construct object as a map from strings to arbitrary values (a data structure) has always been a makeshift solution in JavaScript. The safest way to do so is by creating an object whose prototype is null. Then you still have to ensure that no key is ever the string '__proto__', because that property key triggers special functionality in many JavaScript engines.

The following ES5 code contains the function countWords that uses the object dict as a map:

var dict = Object.create(null);
function countWords(word) {
    var escapedWord = escapeKey(word);
    if (escapedWord in dict) {
        dict[escapedWord]++;
    } else {
        dict[escapedWord] = 1;
    }
}
function escapeKey(key) {
    if (key.indexOf('__proto__') === 0) {
        return key+'%';
    } else {
        return key;
    }
}
In ES6, you can use the built-in data structure Map and don’t have to escape keys. As a downside, incrementing values inside Maps is less convenient.

const map = new Map();
function countWords(word) {
    const count = map.get(word) || 0;
    map.set(word, count + 1);
}
Another benefit of Maps is that you can use arbitrary values as keys, not just strings.

More information:

Section “The dict Pattern: Objects Without Prototypes Are Better Maps” in “Speaking JavaScript”
Chapter “Maps and Sets”
4.16 New string methods
The ECMAScript 6 standard library provides several new methods for strings.

From indexOf to startsWith:

if (str.indexOf('x') === 0) {} // ES5
if (str.startsWith('x')) {} // ES6
From indexOf to endsWith:

function endsWith(str, suffix) { // ES5
  var index = str.indexOf(suffix);
  return index >= 0
    && index === str.length-suffix.length;
}
str.endsWith(suffix); // ES6
From indexOf to includes:

if (str.indexOf('x') >= 0) {} // ES5
if (str.includes('x')) {} // ES6
From join to repeat (the former way of repeating a string is more of a hack):

new Array(3+1).join('#') // ES5
'#'.repeat(3) // ES6
More information: Chapter “New string features”

4.17 New Array methods
There are also several new Array methods in ES6.

4.17.1 From Array.prototype.indexOf to Array.prototype.findIndex
The latter can be used to find NaN, which the former can’t detect:

const arr = ['a', NaN];

console.log(arr.indexOf(NaN)); // -1
console.log(arr.findIndex(x => Number.isNaN(x))); // 1
As an aside, Number.isNaN() provides a safe way to detect NaN (because it doesn’t coerce non-numbers to numbers):

> isNaN('abc')
true
> Number.isNaN('abc')
false
4.17.2 From Array.prototype.slice() to Array.from()
In ES5, the latter method was used to convert Array-like objects to Arrays. In ES6, you have Array.from():

var arr1 = Array.prototype.slice.call(arguments); // ES5
const arr2 = Array.from(arguments); // ES6
If a value is iterable, you can also use the spread operator (...) to convert it to an Array:

const arr1 = [...'abc'];
    // ['a', 'b', 'c']
const arr2 = [...new Set(['b', 'b', 'a', 'b'])];
    // ['b', 'a']
4.17.3 From apply() to Array.prototype.fill()
The former enables a hack for creating an Array of arbitrary length that is filled with values. The latter provides a cleaner way for doing so (it overwrites all existing elements and treats each hole as if it were the element undefined).

// ES5: same as Array(undefined, undefined)
var arr1 = Array.apply(null, new Array(2));
    // [undefined, undefined]
const arr2 = new Array(2).fill(undefined);
    // [undefined, undefined]

var arr3 = Array.apply(null, new Array(2))
    .map(function (x) { return 'x' });
    // ['x', 'x']
const arr3 = new Array(2).fill('x');
    // ['x', 'x']
More information: Chapter “New Array features”

4.18 From CommonJS modules to ES6 modules
Even in ES5, module systems based on either AMD syntax or CommonJS syntax have mostly replaced hand-written solutions such as the revealing module pattern.

ES6 has built-in support for modules. Alas, no JavaScript engine supports them natively, yet. But tools such as browserify, webpack or jspm let you use ES6 syntax to create modules, making the code you write future-proof.

4.18.1 Multiple exports
In CommonJS, you export multiple entities as follows:

//------ lib.js ------
var sqrt = Math.sqrt;
function square(x) {
    return x * x;
}
function diag(x, y) {
    return sqrt(square(x) + square(y));
}
module.exports = {
    sqrt: sqrt,
    square: square,
    diag: diag,
};

//------ main1.js ------
var square = require('lib').square;
var diag = require('lib').diag;

console.log(square(11)); // 121
console.log(diag(4, 3)); // 5
Alternatively, you can import the whole module as an object and access square and diag via it:

//------ main2.js ------
var lib = require('lib');
console.log(lib.square(11)); // 121
console.log(lib.diag(4, 3)); // 5
In ES6, multiple exports are called named exports and handled like this:

//------ lib.js ------
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export function diag(x, y) {
    return sqrt(square(x) + square(y));
}

//------ main1.js ------
import { square, diag } from 'lib';
console.log(square(11)); // 121
console.log(diag(4, 3)); // 5
The syntax for importing modules as objects looks as follows (line A):

//------ main2.js ------
import * as lib from 'lib'; // (A)
console.log(lib.square(11)); // 121
console.log(lib.diag(4, 3)); // 5
4.18.2 Single exports
Node.js extends CommonJS and lets you export single values from modules, via module.exports:

//------ myFunc.js ------
module.exports = function () { ··· };

//------ main1.js ------
var myFunc = require('myFunc');
myFunc();
In ES6, the same thing is done via export default:

//------ myFunc.js ------
export default function () { ··· } // no semicolon!

//------ main1.js ------
import myFunc from 'myFunc';
myFunc();
More information: chapter “Modules”.

4.19 What to do next
Now that you got a first taste of ES6, you can continue your exploration by browsing the chapters: Each chapter covers a feature or a set of related features and starts with an overview.
