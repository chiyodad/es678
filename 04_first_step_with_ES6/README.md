# 4. ECMAScript 6의 첫 단계

이 챕터는 당신이 ECMAScript6 첫 단계를 오르는 것을 돕는다. 쉽게 적용하고, ES5를 통해 이 기능을 설명할 ES6 기능 리스트를 만든다.

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
func()가 undefined 반환 하는것은 놀랍다. 만약 무슨일이 일어났는지 더 면밀하게 반영하기 위해 이 코드를 재작성 한다면 이유를 알 수 있다.

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

ES6 – 펼침 연산자:

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

### 4.13.2 파생 클래스
ES5에서 하위 클래스를 만드는 것은 복잡하고 그 중에서 특히 부모 생성자와 부모 프로퍼티를 참조 하는 것은 복잡하다. 여기 Person에 하위 생성자 Employee를 생성하는 인정받은 방법이 있다.:

```javascript
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
```
ES6는 extends 절을 통한 서브클래스 생성을 내장으로 지원한다.:

```javascript
class Employee extends Person {
    constructor(name, title) {
        super(name);
        this.title = title;
    }
    describe() {
        return super.describe() + ' (' + this.title + ')';
    }
}
```
더 자세한 내용: 챕터 "클래스".

## 4.14 커스톰 에러 생성자로 부터 에러의 하위 클래스로
ES5에서 익셉션, 에러에 대한 내장 생성자를 하위 클래스를 만드는것은 불가능 했다(챕터 말하는 자바스크립트의 내장객체의 하위 클래스에서 이유를 설명한다.). 아래 코드는 생성자 MyError에게 스택 트래이스처럼 중요한 기능을 주는 해결 방법을 보여준다.:

```javascript
function MyError() {
    // Use Error as a function
    var superInstance = Error.apply(null, arguments);
    copyOwnPropertiesFrom(this, superInstance);
}
MyError.prototype = Object.create(Error.prototype);
MyError.prototype.constructor = MyError;
```

ES6에서 모든 내장 생성자는 서브클래스를 만들수 있고, 이것은 아래의 코드가 ES5코드를 흉내 내는 것을 할 수 있는 이유이다.:

```javascript
class MyError extends Error {
}
```
더 자세한 정보: 섹션 "내장 생성자의 서브클래스 만들기"

## 4.15 객체에서 맵으로
이 언어에서 문자열로 아무 값을 얻는 맵(자료 구조)처럼 객체 생성을 사용하는것은 자바스크립트에서 임시 방법이 였다. 이것의 더 안전한 방법은 프로퍼티가 null인 객체 생성을 통한 것 이다. 이 때 당신은 심지어 문자열 '__proto__' 가 없는지 확인해야 한다. 왜냐하면 그 프로퍼티 키는 많은 자바스크립트 엔진에서 특별한 기능을 유발하기 때문이다.

아래 ES5 코드는 맵과 같은 객체 dict을 사용한 함수 countWords를 포함한다.:

```javascript
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
```

ES6에서 당신은 내장 자료 구조 Mad을 사용할 수 있고 escapeKey는 없어도 된다. 단점은 맵안에서 값을 증가시키는 것은 덜 편리하다.

```javascript
const map = new Map();
function countWords(word) {
    const count = map.get(word) || 0;
    map.set(word, count + 1);
}
```
맵에 대한 다른 장점은 키로 아무 문자열 뿐 아니라 아무 값이나 사용할 수 있다.

더 자세한 내용: :

자바스크립트 말하기에서 섹션 "사전 패턴: 프로토타입 없는 객체는 더 나은 맵 이다." 
챕터 "맵과 셋"

## 4.16 새로운 문자열 메소드
ECMAScript 6 표준 라이브러리는 몇개의 스트링에 관한 새로운 메소드를 제공한다.

indexOf로 부터 startsWidth:

```javascript
if (str.indexOf('x') === 0) {} // ES5
if (str.startsWith('x')) {} // ES6
```

indexOf로 부터 endsWith:

```javascript
function endsWith(str, suffix) { // ES5
  var index = str.indexOf(suffix);
  return index >= 0
    && index === str.length-suffix.length;
}
str.endsWith(suffix); // ES6
```

indexOf로 부터 includes:

```javascript
if (str.indexOf('x') >= 0) {} // ES5
if (str.includes('x')) {} // ES6
```
join으로 부터 repeat (문자열을 반복의 이전 방법 더 핵(hack)이다.):

```javascript
new Array(3+1).join('#') // ES5
'#'.repeat(3) // ES6
```

더 자세한 정보: 챕터 "새로운 문자열 기능"

## 4.17 새로운 배열 메소드
ES6에서 몇 가지 새로운 배열 메소드가 있다.

### 4.17.1 Array.prototype.indexOf에서 Array.prototype.findIndex
후자는 NaN을 발견하는데 사용 할 수 있지만 전자는 발견하지 못한다.:

```javascript
const arr = ['a', NaN];

console.log(arr.indexOf(NaN)); // -1
console.log(arr.findIndex(x => Number.isNaN(x))); // 1
```

여담으로 Number.isNaN()은 NaN을 확인하는 안전한 방법을 제공한다. (왜냐하면 이것은 비 숫자를 숫자로 강제로 변환하지 않기 때문이다.):

```javascript
> isNaN('abc')
true
> Number.isNaN('abc')
false
```

### 4.17.2 Array.prototype.slice()에서 Array.from()으로
ES5에서 후자 메소드는 유사 배열을 배열로 변경하는데 사용하였는다. ES6에서는 Array.from()을 사용한다.:

```javascript
var arr1 = Array.prototype.slice.call(arguments); // ES5
const arr2 = Array.from(arguments); // ES6
```

만약 값이 이터러블이면 또한 펼침 연산자를 사용하여 이것은 배열로 만들 수 있다.:

```javascript
const arr1 = [...'abc'];
    // ['a', 'b', 'c']
const arr2 = [...new Set(['b', 'b', 'a', 'b'])];
    // ['b', 'a']
```

### 4.17.3 apply()에서 Array.ptototype.fill()로
이전은 값으로 채워진 임의 수의 배열 크기를 갖는 배열 생성에 대한 꼼수(hack)를 할 수 있게 하였다. 후자는 이것에 대한(이것은 모든 존재하는 원소를 덥어쓰고 만약 이것이 element undefinded같은 구멍을 다룬다.) 명백한 방법을 제공한다.

```javascript
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
```
더 자세한 정보: 챕터 "새로운 배열 기능"

## 4.18 CommonJS 모듈에서 ES6모듈로
심지어 ES5에서 모듈 시스템은 AMD 문법 또는 CommonJS문법 기반으로 대부분 "the revealing module pattern"같은 수기로 변경하는 방법을 가졌다.

ES6는 모듈을 내장해서 지원한다. 유감스럽게도 네이티브로 제공하는 자바스크립트 엔진은 없었다. 그러나 브라우져파이, 웹팩 또는 jspm 같은 툴은 당신이 모듈 생성에 대한 ES6 문법을 사용하게 해 주고 당신의 쓴 코드를 미래지향적으로 만들어 준다.

### Multiple export

CommonJS에서 당신은 어려 개체를 익스포트는 다음과 같다.:

```javascript
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
```

그 대신에 당신은 객체로 전체 모듈을 임포트할 수 있고 그것을 통해 square나 diag에 접근할 수 있다.

```javascript
//------ main2.js ------
var lib = require('lib');
console.log(lib.square(11)); // 121
console.log(lib.diag(4, 3)); // 5
```

ES6에서 다중 익스포트는 기명된 익스포트로 불리고 이것 처럼 다룬다.:

```javascript
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
```

객체 처럼 모듈 임포트하는것에 대한 문법은 아래 (줄 A) 처럼 보인다.:

```javascript
//------ main2.js ------
import * as lib from 'lib'; // (A)
console.log(lib.square(11)); // 121
console.log(lib.diag(4, 3)); // 5
```

### 4.18.2 단일 익스포트
Node.js는 CommonJS를 확장하였고 module.exports를 통해 당신을 모듈로 부터 단일 값으로 익스포트한다.:

```javascript
//------ myFunc.js ------
module.exports = function () { ··· };

//------ main1.js ------
var myFunc = require('myFunc');
myFunc();
```
ES6에서 exprot default를 통해 같은 것을 한다.:

```javascript
//------ myFunc.js ------
export default function () { ··· } // no semicolon!

//------ main1.js ------
import myFunc from 'myFunc';
myFunc();
```
더 자세한 정보: 챕터 "모듈".

## 4.19 다음에는 무엇을 할까
이제 당신은 ES6에 대해 처음으로 맛을 보았고, 당신은 챕터를 검색하여 탐험을 계속 할 수 있다. 각 챕터는 기능이나 관련된 기능의 집합을 다루고 개요와 함께 시작된다.
