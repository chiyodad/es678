# 화살표함수

## 13.1 개요

화살표함수는 두 가지 이점이 있습니다.
첫째, 이전함수 표현식 보다 덜 복잡합니다.

```javascript
const arr = [1, 2, 3];
const squares = arr.map(x => x * x);

// Traditional function expression:
const squares = arr.map(function (x) { return x * x });
```
둘째,  this가 주변(lexical)으로부터 찾아집니다. 따라서 더 이상 bind() 또는 that = this가 필요없습니다.
```javascript
function UiComponent() {
    const button = document.getElementById('myButton');
    button.addEventListener('click', () => {
        console.log('CLICK');
        this.handleClick(); // lexical `this`
    });
}
```
다음의 변수는 모두 화살표함수 lexical에 포함됩니다.
 * arguments
 * super
 * this
 * new.target


## 13.2 메소드가 아닌 이전함수에서 this 사용은 좋지 않다.
자바스크립트에서 이전함수는 다음처럼 사용할 수 있습니다:
 1. 메소드가 아닌 함수
 2. 메소드
 3. 생성자
//
이들의 역할충돌 : 역할2, 3번 때문에 함수는 항상 그들의 this를 갖기 때문에 콜백( 역할1 ) 내부에서 감싸는 메소드의 this에 접근하는 것을 막습니다.

다음의 ES5코드에서 볼 수 있습니다.

```javascript
function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) { // (A)
    'use strict';
    return arr.map(function (x) { // (B)
        // Doesn’t work:
        return this.prefix + x; // (C)
    });
};
```
C라인에서 우리는 this.prefix에 접근해야 하지만 할 수 없는 이유는 B라인 함수의 this가 A라인 메소드의 this를 가리기 때문입니다.
strict mode시 none-method 함수안에서 this는 undefined이며 이것이 우리가 Prefixer를 사용할 때 에러를 받는 이유입니다:

```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
TypeError: Cannot read property 'prefix' of undefined
```
ECMAScript 5에 이 문제점을 피하는 방법에는 세가지가 있습니다.

### 13.2.1 해결책 1: that = this
명시적인 변수 that에 this를 할당할 수 있습니다. 즉, 아래  A라인과 같습니다. 

```javascript
function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    var that = this; // (A)
    return arr.map(function (x) {
        return that.prefix + x;
    });
};
```
예상대로 지금은 Prefixer가 동작합니다:

```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
[ 'Hi Joe', 'Hi Alex' ]
```
### 13.2.2 해결책 2: this값 지정
일부 Array 메소드는 콜백을 호출 할 때 this 값을 지정하기 위한 추가 인자를 가지고 있습니다.
아래 A라인의 마지막 인자와 같습니다.

```javascript
function Prefixer(prefix) {
    this.prefix = prefix;w
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }, this); // (A)
};
```
### 13.2.3 해결책 3: bind(this)
당신은 bind()메소드를 사용해 함수의 this가 호출된 방법에 따라 결정되는 것에서 함수의 this가 언제나 같은 값으로 고정 되도록
변환할 수 있다. 즉, 아래 A라인에서 하는 일과 같습니다.

```javascript
function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }.bind(this)); // (A)
};
```
### 13.2.4 ECMAScript 6 해결책: 화살표함수
화살표함수는 보다 편한 문법으로 해결책 3을 가능하게 합니다. 화살표함수를 사용한 코드를 보시죠.

```javascript
function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map((x) => {
        return this.prefix + x;
    });
};
```
완전히 ES6스러운 코드애서는 클래스와 화살표함수의 좀더 간단한 변형 사용할 수 있습니다:

```javascript
class Prefixer {
    constructor(prefix) {
        this.prefix = prefix;
    }
    prefixArray(arr) {
        return arr.map(x => this.prefix + x); // (A)
    }
}
````
A라인에서 화살표함수의 두 부분을 수정해 약간의 문자를 줄일 수 있습니다.

* 함수 인자가 하나이고 그 인자의 식별자가 하나이면 괄호는 생략 가능합니다.
* 화살표 다음에 이어지는 하나의 표현식은 반환되어집니다.
코드에서, constructor와 prefixArray 메소드가 오브젝트리터럴에서 동작하는 더 간결한 ES6문법을 새롭게 사용하여 정의된 것 또한 볼수 있습니다.

## 13.3 화살표함수 문법
뚱뚱한화살표 => ( 얇은화살표의 반대 -> )는 CoffeeScript와 호환되도록 채용 되었습니다. 애로우 함수와 매우 유사 합니다.

인자명시:
```javascript
    () => { ... } // 인자 없음
     x => { ... } // 인자 하나
(x, y) => { ... } // 인자 여러개
```
본문명시:
```javascript
x => { return x * x }  // 블럭
x => x * x  // 위 라인과 같음
```
블럭구문은 보통 함수 본문처럼 동작 합니다. 예를들어 값을 반환하려면 return이 필요합니다. 
하나의 표현식본문, 그 표현식은 언제나 암시적으로 반환되어집니다.

Note 표현식본문을 가진 애로우함수가 어느정도 장황함을 줄일 수 있는지 비교:

```javascript
const squares = [1, 2, 3].map(function (x) { return x * x });
const squares = [1, 2, 3].map(x => x * x);
```

### 13.3.1 단일인자 주변 괄호생략
인자 주변에 괄호는 단일 식별자로 이뤄졌을 때만 생략 가능합니다.

```javascript
> [1,2,3].map(x => 2 * x)
[ 2, 4, 6 ]
```
그 밖의 경우가 생기면, 비록 단일인자라 하더라도 괄호를 입력해야만 합니다. 예를들어, 단일인자를 분해한다면 괄호가 필요합니다:

```javascript
> [[1,2], [3,4]].map(([a,b]) => a + b)
[ 3, 7 ]
```
그리고 단일인자가 기본값을 가질 때도 괄호가 필요합니다.(undefined는 기본값을 트리거 합니다!):

```javascript
> [1, undefined, 3].map((x='yes') => x)
[ 1, 'yes', 3 ]
```

## 13.4 Lexical변수들

### 13.4 변수값의 출처: 정적 vs 동적

변수가 값을 받는 두가지 방법은 다음과 같습니다.

첫째, 정적(어휘) : 변수의 값이 프로그램의 구조에 의해 결정됩니다; 변수의 값은 감싸는 스코프로 부터 받습니다. 
예를들어:

```javascript
const x = 123;

function foo(y) {
    return x; // 값이 정적으로 받아졌다.
}
```
둘째, 동적: 변수의 값은 함수 호출에 의해 받습니다.
예를들어:

```javascript
function bar(arg) {
    return arg; // 값이 동적으로 받아졌다.
}
```
### 13.4.2 화살표함수 lexical에 있는 변수들

this의 출처는 다른것과 중요하게 구별되는 화살표 함수의 양상입니다.
* 이전함수는 동적 this를 갖습니다; this의 값은 함수들이 호출되어진 방법에 의해 결정됩니다.
* 화살표함수는 lexical this를 갖습니다; 그것의 값은 감싸는 스코프에 의해 결정됩니다.

변수의 값이 lexically로 결정되는 [전체목록](http://exploringjs.com/es6/ch_arrow-functions.html) 입니다.

* arguments
* super
* this
* new.target

## 13.5 문법함정

가끔 실수를 유발할 수 있는 몇 가지 문법관련 세부사항이 있습니다.

### 13.5.1 화살표함수는 매우 느슨하게 연결됩니다.

문법상, 화살표함수는 매우 느슨하게 연결됩니다. 그 이유는 당신은 모든 표현식을 표현식본문에 뭉쳐서 나타낼 수 있길 원하기 때문입니다.
표현식은 화살표함수보다 더 우선적으로 결합될 것입니다.

```javascript
const f = x => (x % 2) === 0 ? x : 0;
```
그 결과, 만약 또 다른곳에서 표시 한다면 당신은 자주 화살표함수를 괄호 안으로 감싸야 합니다.
예를들어:

```javascript
console.log(typeof () => {}); // SyntaxError
console.log(typeof (() => {})); // OK
```
반면, 표현식본문을 괄호에 넣지 않는 것 처럼 typeof를 사용할 수 있습니다.

```javascript
const f = x => typeof x;
```
### 13.5.2 화살표함수 인자 뒤에서 개행 금지

ES6는 화살표함수의 인자선언과 본문 사이에 개행을 허용하지 않습니다.

```javascript
const func1 = a // SyntaxError
=> a*2;
const func2 = a => a*2; // OK

const func3 = (x, y) // SyntaxError
=> {
    return x + y;
};
const func4 = (x, y) => { // OK
    return x + y;
};
const func5 = (x, // OK
y) => {
    return x + y;
};
...
```
이런 규제의 이론적 해석은 미래의 화살표함수 “headless”에 대한 옵션을 열어둔 것입니다: 만약 인자가 없다면 괄호 생략이 가능합니다.

### 13.5.3 표현식의 본문으로 문을 사용할 수 없습니다.

#### 13.5.3.1 표현식 vs 문

Quick review ( 더 많은 정보는 “자바스크립트를 말하다”를 찾아보세요 ):

표현식은 값들을 생성합니다(평가됩니다). 예:

```javascript
3 + 4
foo(7)
'abc'.length
```
문은 동작을 합니다. 예:

```javascript
while (true) { ··· }
return 123;
```
대부분의 표현식은 단순히 문안에서 언급함으로써 문처럼 사용할 수 있습니다.

```javascript
function bar() {
    3 + 4;
    foo(7);
    'abc'.length;
}
```
#### 13.5.3.2 화살표함수의 본문
`13.5.3.2 The bodies of arrow functions`
만약 화살표함수의 본문이 하나의 표현식이라면 중괄호가 필요하지 않습니다.

```javascript
asyncFunc.then(x => console.log(x));
```
하지만 문은 중괄호안에 넣어줘야 합니다.

```javascript
asyncFunc.catch(x => { throw x });
```
### 13.5.4 오브젝트리터럴 리턴하기
`13.5.4 Returning object literals`
만약 표현식본문이 오브젝트 리터럴이길 원한다면 블럭몸체와 더블어 표현식본문을 괄호 안으로 넣어야 합니다.
이 화살표함수의 본문은 블럭에 레이블 bar와  123 서술 표현식입니다.

```javascript
const f = x => { bar: 123 }
```
이 화살표 함수의 본문은 하나의 오브젝트리터럴 표현식입니다.

```javascript
const f = x => ({ bar: 123 })
```
## 13.6 화살표함수 즉시실행
함수즉시실행 표현식을 기억합니까(IIFEs)? 다음과 같이 보면 ECMAScript 5 에서 블록스코핑과 값 반환 블록을 흉내내기 위해 사용하곤 했습니다.
```javascript
(function () { // open IIFE
    // inside IIFE
}()); // close IIFE
```
화살표함수 즉시실행을(IIAF) 사용하면 약간의 문자를 줄일 수 있습니다.
```javascript
(() => {
    return 123
})();
```
IIFEs와 마찬가지로, IIAFs가 두번연속 함수호출 처럼 해석되는(첫번째로는 함수, 두번째로는 인자) 것을 방지하기 위해 IIAFs도 세미콜론으로 끝내야 합니다.( 또는 상응하는 조치를 사용 )

IIAF에 블럭몸체가 있더라도 반드시 괄호로 감싸야 합니다. 왜냐하면 늦은 연결의 이유로 함수를 (직접적으로) 호출 할 수 없기 때문입니다. Note 괄호는 반드시 화살표함수 주변에 써야 합니다. IIFEs에서는 전제 문을 감싸거나 함수 표현만 감싸거나 어느 한쪽을 선택할 수 있습니다.  

앞 절에서 언급한 바와 같이 화살표함수의 느슨한 결합은 여기의 표현식 처럼 본문을 표현하는데 하는데 도움이 됩니다.

```javascript
const value = () => foo()
```
이처럼 해석
```javascript
const value = () => (foo())
```
이것과는 다름
```javascript
const value = (() => foo)()
```
[A section in the chapter on callable entities](http://exploringjs.com/es6/ch_callables.html#sec_iifes-in-es6) 
에서 ES6의 IIFE과 IIAF를 이용한 더 많은 정보를 접할 수 있습니다.

## 13.7 화살표함수 vs 보통함수
화살표함수는 보통 함수와 단지 두가지만 다릅니다.

* 다음구성은 lexical이다 : arguments, super, this, new.target
* 생성자로 사용할 수 없다 : 보통함수는 내부메소드인[[Construct]]과 prototype에 의한 new를 지원합니다. 화살표함수는 둘 다 지원하지 않아  new (() => {})할 때 에러가 발생합니다.

그 밖에 화살표함수와 일반함수 사이에 크게 주목할만한 차이점은 없습니다.
예를들어 typeof와 instanceof 는 같은 결과입니다.

```javascript
> typeof (() => {})
'function'
> () => {} instanceof Function
true

> typeof function () {}
'function'
> function () {} instanceof Function
true
```
화살표함수와 일반함수 사용에 대한 더 많은 정보는 [the chapter on callable entities]( http://exploringjs.com/es6/ch_callables.html#sec_callables-style )를 참고하세요.
