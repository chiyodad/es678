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

## 4.6 for에서 forEach()로 for-of로
ES5이전에는, 당신은 배열을 아래 처럼 순환했다.::

```javascript
var arr = ['a', 'b', 'c'];
for (var i=0; i<arr.length; i++) {
    var elem = arr[i];
    console.log(elem);
}
```
ES5에서는, 당신은 배열 메소드 forEach 사용에 대한 선택이 있다.:

```javascript
arr.forEach(function (elem) {
    console.log(elem);
});
```
for 루프는 당신이 정지 할 수 있는 이점이 있고, forEach는 간결하다는 이점이 있다.

ES6에서는 for-fo 루프는 두 이점을 결합한다.:

```javascript
const arr = ['a', 'b', 'c'];
for (const elem of arr) {
    console.log(elem);
}
```
만약 당신이 각 배열의 원소의 인덱스나 값을 원한다면 새로운 배열 메소드 entires()와 해체를 통해 for-of는 당신에게 다루게 해 준다.:

```javascript
for (const [index, elem] of arr.entries()) {
    console.log(index+'. '+elem);
}
```
더 자세한 내용: 챕터 "for-of 루프"

## 4.7 파라미터 기본값 다루기
ES5에서 당신은 파라미터의 기본값을 아래 처럼 지정한다.:

```javascript
function foo(x, y) {
    x = x || 0;
    y = y || 0;
    ···
}
```
ES6는 더 좋은 문법을 갖는다.:

```javascript
function foo(x=0, y=0) {
    ···
}
```
ES6에서 추가적인 이득은 파라미터 기본값은 오직 undefined에 의해 유발되며 반면에 ES5이전에서는 어느 거짓인 값(falsy)에 의 해 유발된다.

더 자세한 내용: 섹션 "파라미터 기본값".

## 4.8 기명 파라미터 다루기
자바스크립트에서 파라미터에 이름을 붙이는 흔한 방법은 객체리터럴을 통한 것이다(옵션 객체 패턴으로 불리는).:

```javascript
selectEntries({ start: 0, end: -1 });
```
이 접근은 두가지 이점이 있다: 코드가 더 스스로를 설명하고 임의로 파라미터를 빼기 더 쉽다.

ES5에서 당신은 selectEntries()를 다음 처럼 구현할 수 있다.:

```javascript
function selectEntries(options) {
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    ···
}
```
ES6에서 당신은 파라미터 정의에서 해체를 사용할 수 있고 코드는 더욱 간단해 진다.:

```javascript
function selectEntries({ start=0, end=-1, step=1 }) {
    ···
}
```
### 4.8.1 선택적 파라미터 만들기
ES5에서 파라미터 옵션을 선택적으로 만들기 위해서 당신은 코드의 A줄을 추가 해야 한다.:

```javascript
function selectEntries(options) {
    options = options || {}; // (A)
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    ···
}
```
es6에서는 당신은 파라미터 기본값을 {}로 지정 할 수 있다.:

```javascript
function selectEntries({ start=0, end=-1, step=1 } = {}) {
    ···
}
```
더 자세한 내용은: 섹션 "시뮬레이션 명명 파라미터"

## 4.9 arguments에서 남은 파라미터로
ES5에서 당신은 함수(또는 메소드) 인수를 임의의 수를 받게 한다면 당신은 반드시 특별 변수인 arguments를 사용해야 한다.:

```javascript
function logAllArguments() {
    for (var i=0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}
```
ES6에서 당신은 남은 파라미터(아래 예제의 args) ...연산자를 통해 선언 할 수 있다.:

```javascript
function logAllArguments(...args) {
    for (const arg of args) {
        console.log(arg);
    }
}
```
만약 당신이 뒷 부분에만 흥미가 있다면 남은 파라미터는 심지어 더 좋다:
```javascript
function format(pattern, ...args) {
    ···
}
```
ES5에서 이 케이스를 다루면 꼴사납다:

```javascript
function format() {
    var pattern = arguments[0];
    var args = [].slice.call(arguments, 1);
    ···
}
```
남은 파라미터는 코드를 읽기에 쉽게 만들어 준다: 당신은 단지 그 파라미터가 정의를 보면 함수가 변할 수 있는 수의 파라미터를 갖는지 알 수 있다.

더 자세한 정보: 섹션 "남은 파라미터".

## 4.10 apply()에서 펼침 연산자 (...)
ES5에서 당신은 배열을 apply()을 통해 파라미터로 변환한다. ES6은 이 목적을 위해 펼침 연산자를 갖는다.

### 4.10.1 Math.amx();
ES5 – apply():
```javascript
> Math.max.apply(null, [-1, 5, 11, 3])
11
```
ES6 – 펼침 연산자:
```javascript
> Math.max(...[-1, 5, 11, 3])
11
```
### 4.10.2 Array.prototype.push()
ES5 – apply():

```javascript
var arr1 = ['a', 'b'];
var arr2 = ['c', 'd'];

arr1.push.apply(arr1, arr2);
    // arr1 is now ['a', 'b', 'c', 'd']
```

ES6 – 펼침 연산자:

```javascript
const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

arr1.push(...arr2);
    // arr1 is now ['a', 'b', 'c', 'd']
```
더 자세한 내용: 섹션 "펼침 연산자 (...)".

## 4.11 concat()에서 펼침 연산자 (...)로
펼침 연산자는 그 피연산자의 내용을 배열 원소로 바꿀 수 있다. 그것은 배열 메소드 concat()의 대안이 되는 것을 의미한다..

ES5 – concat():

```javascript
var arr1 = ['a', 'b'];
var arr2 = ['c'];
var arr3 = ['d', 'e'];

console.log(arr1.concat(arr2, arr3));
    // [ 'a', 'b', 'c', 'd', 'e' ]
```

ES6 – spread operator:

```javascript
const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];

console.log([...arr1, ...arr2, ...arr3]);
    // [ 'a', 'b', 'c', 'd', 'e' ]
```
더 자세한 내용: 섹션 "펼침 연산자 (...)".

## 4.12 객체 리터럴에서의 함수표현식에서 메소드 정의로
자바스크립트에서 메소드는 프로퍼티의 값이 함수인 것이다.

ES5 객체 리터럴에서 메소드는 다른 프로퍼티 처럼 생성된다. 이 프로퍼티 값은 함수 표현식을 통해 제공된다.

```javascript
var obj = {
    foo: function () {
        ···
    },
    bar: function () {
        this.foo();
    }, // trailing comma is legal in ES5
}
```
ES6은 메소드 정의를 갖고, 메소드 생성에 대한 특별 문법을 갖는다.:

```javascript
const obj = {
    foo() {
        ···
    },
    bar() {
        this.foo();
    },
}
```
더 자세한 내용: 섹션 "메소드 정의"

## 4.13 생성자로부터 클래스로
ES6 클래스는 대부분 함수 생성자에 대한 문법보다 더 편리 하다.

### 4.13.1 기본 클래스
ES5에서 당신은 생성자 함수를 바로 구현한다.:

```javascript
function Person(name) {
    this.name = name;
}
Person.prototype.describe = function () {
    return 'Person called '+this.name;
};
```
ES6에서 클래스는 생성자 함수에 대한 약간 더 편리한 문법을 제공한다(특히 메소드 정의에 대한 간결한 문법을 주목하라 - function 키워드는 필요치 않다).:
```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
    describe() {
        return 'Person called '+this.name;
    }
}
```

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
