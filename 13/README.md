#Arrow functions

##13.1 Overview

애로우함수는 두가지 이점이 있습니다.
첫째, 일반함수 표현 보다 덜 복잡합니다.
```javascript
const arr = [1, 2, 3];
const squares = arr.map(x => x * x);

// Traditional function expression:
const squares = arr.map(function (x) { return x * x });
```
둘째,  this를 어휘환경에서 찾습니다. 따라서 더 이상 bind() 또는 that = this가 필요없습니다.
```javascript
function UiComponent() {
    const button = document.getElementById('myButton');
    button.addEventListener('click', () => {
        console.log('CLICK');
        this.handleClick(); // lexical `this`
    });
}
```
다음의 변수는 모두 애로우함수 어휘안에 포함됩니다.
* arguments
* super
* this
* new.target


##13.2 non-method 일반함수는 this 때문에 좋지 않다.
자바스크립트에서 일반함수는 다음처럼 사용할 수 있습니다.
* 1. Non-method함수
* 2. 메소드
* 3. 생성자

이것들의 역할충돌 : 역할2, 3번 때문에 함수는 항상 자신의 this를 갖습니다. 
하지만 콜백안에서는 감싸는 메소드의 this 접근을 방지합니다.( 역할1 )

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
C라인에서 우리는 this.prefix에 접근하려 합니다. 하지만 그럴 수 없습니다. 왜냐하면 B라인 함수의 this가 A라인 메소드의 this를 가리기 때문입니다.
strict mode에서 none-method함수의 this는 undefined이며 이것이 우리가 Prefixer를 사용할 때 에러가 발생하는 이유입니다.
```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
TypeError: Cannot read property 'prefix' of undefined
```
###13.2.1 해결책 1: that = this
this를 명시적으로 변수에 할당할 수 있습니다. 즉, 아래  A라인과 같습니다. 
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
예상대로  지금은 Prefixer가 동작합니다.
```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
[ 'Hi Joe', 'Hi Alex' ]
```
###13.2.2 해결책 2: this값 지정
일부 Array 메소드는 콜백을 호출 할 때 this 값을 지정하기 위한 추가 인자를 가지고 있습니다.
아래 A라인의 마지막 인자와 같습니다.
```javascript
function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }, this); // (A)
};
```
###13.2.3 해결책 3: bind(this)
호출하는 방식에 의해( call의한 함수 호출, 메소드 호출 등등 ) this가 결정되는 함수를 언제나 같은 this값으로 고정된 함수로 변환해주는 bind() 메소드를 사용할수 있습니다.
즉, 아래 A라인에서 하는 일과 같습니다.
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
###13.2.4 ECMAScript 6 해결책: 애로우함수
애로우함수는 보다 편한 문법으로 해결책 3을 가능하게 합니다. 애로우함수를 사용한 코드를 보시죠.
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
완전한 ES6코드, 클래스와 애로우 함수의 좀더 간결한 변화?를 사용하세요.
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
A라인에서 애로우함수 두 부분의 수정으로 약간의 문자를 줄일 수 있습니다.
* 함수 인자가 하나이고 그 인자의 식별자가 하나이면 괄호는 생략 가능합니다.
* 화살표 다음에 이어지는 하나의 식은 반환됩니다.
코드에서, constructor 와 prefixArray 메소드가 object literals에서 동작하는 더 간결한 ES6문법을 새롭게 사용하여 정의된 것 또한 볼수 있습니다.

##13.3 애로우함수 문법
굵은화살표 => ( 얇은 화살표의 반대 -> )는 CoffeeScript와 호환되도록 채용 되었습니다. 애로우 함수와 매우 유사 합니다.

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
블럭구문은 일반함수 본문처럼 동작 합니다. 예를들어 값을 반환하려면 return이 필요합니다. 
하나의 식을 가진 본문, 그 식은 언제나 암시적으로 값을 반환합니다.

Note 식본문을 가진 애로우함수가 어느정도 장황함을 줄일 수 있는지 비교:
```javascript
const squares = [1, 2, 3].map(function (x) { return x * x });
const squares = [1, 2, 3].map(x => x * x);
```

###13.3.1 단일인자 주변 괄호생략
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

##13.4 어휘변수들
###13.4 변수값의 출처: 정적 vs 동적
어떤 변수가 그 값을 받는 두가지 방법은 다음과 같습니다.

첫째, 정적(어휘적) : 이 값은 프로그램의 구조에 의해 결정됩니다. 감싸는 스코프로 부터 그 값을 받습니다. 
예를들어:
```javascript
const x = 123;

function foo(y) {
    return x; // value received statically
}
```
둘째, 동적: 이 값은 함수 호출에 의하여 받습니다.
예를들어:
```javascript
function bar(arg) {
    return arg; // value received dynamically
}
```
###13.4.2 애로우함수 어휘에 있는 변수들
this의 출처는 애로우함수가 일반함수와 구별되는 중요한 부분입니다.
* 일반함수는 동적 this를 갖습니다. 이 값은 함수가 어떻게 호출되느냐에 따라 결정됩니다.
* 애로우함수는 어휘적 this를 갖습니다. 이 값은 감싸는 스코프에 의해 결정됩니다.
어휘로 값이 결정되는 변수의 [전체목록](http://exploringjs.com/es6/ch_arrow-functions.html) 입니다.
* arguments
* super
* this
* new.target

##13.5 문법함정
가끔 실수를 유발하는 몇가지 문법관련 세부사항이 있습니다.

###13.5.1 애로우함수는 매우 느슨하게 연결됩니다.
문법상, 애로우함수는 매우 느슨하게 연결됩니다. 그 이유는 ~~ ( 뭐라는거야 )
```
Syntactically, arrow functions bind very loosely. The reason is that you want every expression that can appear in an expression body to “stick together”; it should bind more tightly than the arrow function:
```
```javascript
const f = x => (x % 2) === 0 ? x : 0;
```
그 결과, 만약 애로우함수가 다른곳에서 표시 한다면 자주 괄호로 감싸야 합니다. 
예를들어:
```javascript
console.log(typeof () => {}); // SyntaxError
console.log(typeof (() => {})); // OK
```
반면, 괄호에 넣지 않은 채 본문 식 처럼 typeof를 사용할 수 있습니다.
```javascript
const f = x => typeof x;
```
###13.5.2 애로우함수 인자 이후에 개행 금지
ES6는 애로우함수의 인자선언과 본문 사이에 개행을 허용하지 않습니다.
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
이런 제한의 근거는 장차 애로우함수의 “headless”에 대해 열려있는 옵션을 유지 한다는 것 입니다.( ?? )
만약 인자가 없다면 당신은 괄호를 생략할 수 있습니다.
```
The rationale for this restriction is that it keeps the options open w.r.t. to “headless” arrow functions in the future: if there are zero parameters, you’d be able to omit the parentheses.
```
###13.5.3 식문 같은 문법을 사용할 수 없습니다.
####13.5.3.1 식 vs 문
Quick review ( 더 많은 정보는 “자바스크립트를 말하다”를 찾아보세요 ):

표현식은 값을 생성합니다(평가됩니다). 예를들어:
```javascript
3 + 4
foo(7)
'abc'.length
```
문은 동작을 합니다. 예를들어:
```javascript
while (true) { ··· }
return 123;
```
대부분의 식은 단순히 문안에서 언급함으로써 문처럼 사용할 수 있습니다.
```javascript
function bar() {
    3 + 4;
    foo(7);
    'abc'.length;
}
```
####13.5.3.2 애로우함수의 본문
만약 애로우함수 본문이 하나의 식이라면 중괄호가 필요하지 않습니다.
```javascript
asyncFunc.then(x => console.log(x));
```
하지만 문은 중괄호를 넣어줘야 합니다.
```javascript
asyncFunc.catch(x => { throw x });
```
###13.5.4 object 리터럴 리턴하기
블럭과 더불어 하나의 본문 표현이 Object 리터럴이라면 당신은 괄호안에 넣어야 합니다.

블럭에 레이블 bar와 서술식 123을 포함한 애로우함수의 본문입니다.
```javascript
const f = x => { bar: 123 }
```

하나의 표현이 object리터럴인 애로우 함수의 본문입니다.
```javascript
const f = x => ({ bar: 123 })
```
##13.6 애로우함수 즉시실행
함수즉시실행 식을 기억합니까(IIFEs)? 그것을 보면 ECMAScript 5에서 블럭은 block-scoping과 value-returning을 simulate하기 위해 사용합니다.
```javascript
(function () { // open IIFE
    // inside IIFE
}()); // close IIFE
```
애로우함수 즉시실행을(IIAF) 사용하면 약간의 문자를 줄일 수 있습니다.
```javascript
(() => {
    return 123
})();
```
IIFEs도 마찬가지로, IIAFs가 두번연속 함수호출 처럼 해석되는(첫번째로는 함수, 두번째로는 인자) 것을 방지하기 위해 IIAFs도 세미콜론으로 끝내야 합니다( 또는 상응하는 조치를 사용 )

IIAF에 블럭이 있더라도 반드시 괄호로 감싸야 합니다. 왜냐하면 늦은 연결의 이유로 함수를 ( 직접적으로 ) 호출 할 수 없기 때문입니다. Note 괄호는 반드시 애로우함수 주변에 써야 합니다. IIFEs에서 어느 한쪽을 선택할 수 있습니다. 전제 문을 감싸던지 함수 표현만 감싸던지

앞 절에서 언급한 바와 같이 애로우함수의 느슨한 결합은 이 식으로 본문을 표현하는데 하는데 도움이 됩니다.
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


##13.7 애로우함수 vs 일반함수
애로우 함수는 일반함수와 두가지만 다릅니다.

* 다음은 어휘구조이다 : arguments, super, this, new.target
* 생성자로 사용할 수 없다 : 일반함수는 new에 의한 내부메소드인[[Construct]]과 prototype를 지원합니다. 애로우함수는 둘 다 지원하지 않습니다. new (() => {})할 때 에러가 발생합니다. 

그 밖에 애로우함수와 일반함수 사이에 크게 주목할만한 차이점은 없습니다.
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
애로우함수와 일반함수 사용에 대한 더 많은 정보는 [the chapter on callable entities]( http://exploringjs.com/es6/ch_callables.html#sec_callables-style )를 참고하세요.
