#Arrow functions

##13.1 Overview

애로우함수는 두가지 장점이 있습니다.
첫째, 일반적인 함수 보다 간략합니다.
```javascript
const arr = [1, 2, 3];
const squares = arr.map(x => x * x);

// Traditional function expression:
const squares = arr.map(function (x) { return x * x });
```
둘째,  this를 lexical 환경에서 찾습니다. 따라서 더 이상 bind() 또는 that = this가 필요없습니다.
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


##13.2 none-method 일반 함수가 나쁜건 this 때문이다
자바스크립트에서 일반함수는 다음과 같이 사용할 수 있습니다.
* 1. Non-method함수
* 2. 메소드
* 3. 생성자

역할충돌 : 역할2, 3번 함수는 항상 this를 갖는 반면 함수 내부의 콜백( 역할1 )에선  this를 접근할 수 없습니다.
다음과 같은 ES5코드에서 확인할 수 있습니다.
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
C라인에서 우리는 this.prefix를 접근하고 싶지만 A라인 메소드의 this는 B라인의 함수에서 가려지기 때문에 그럴 수 없습니다.
우리가 Prefixer를 사용할 때 에러가 나는 이유는 strict mode에서 none-method함수의 this는 undefined이기 때문입니다.
```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
TypeError: Cannot read property 'prefix' of undefined
```
###13.2.1 Solution 1: that = this
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
예상대로  Prefixer가 동작합니다.
```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
[ 'Hi Joe', 'Hi Alex' ]
```
###13.2.2 Solution 2: this값 지정
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
###13.2.3 Solution 3: bind(this)
함수의 this는 함수가 호출되는 방식에 따라 결정되는데 ( call의한 함수 호출, 메소드 호출 등등 ) bind() 메소드를 사용하면 함수의 this를 항상 같은 값으로 유지할 수 있습니다. 다음 A라인과 같습니다.
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
###13.2.4 ECMAScript 6 solution: arrow functions
애로우 함수는 보다 편한 문법으로 Solution 3을 가능하게 합니다. 애로우 함수를 사용한 코드를 보시죠.
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
완전한 ES6코드, 클래스와 애로우 함수의 좀더 간결한 변형?을 사용하세요.
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
A라인에서 애로우 함수의 두 부분을 수정하면 약간의 문자를 줄일 수 있습니다.
* 만약 함수인자가 하나라면 괄호를 생략할 수 있습니다.
* 애로우 다음에 이어지는 식은 반환됩니다.
In the code, you can also see that the methods constructor and prefixArray are defined using new, more compact ES6 syntax that also works in object literals.

##13.3 애로우함수 문법
굵은화살표 => ( 얇은 화살표의 반대 -> )는 CoffeeScript와 호환되며 애로우 함수와 매우 유사 합니다.

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
블럭구문은 일반 함수의 본문처럼 동작 합니다. 예를들어 값을 반환하려면 return이 필요합니다. 
식 본문은( 한 줄짜리 식? ) 표현은 암시적으로 값을 반환합니다.

Note how much an arrow function with an expression body can reduce verbosity. Compare:
```javascript
const squares = [1, 2, 3].map(function (x) { return x * x });
const squares = [1, 2, 3].map(x => x * x);
```

###13.3.1 단일인자 괄호생략
단일 인자로 이뤄진 함수는 괄호 생략이 가능합니다.
```javascript
> [1,2,3].map(x => 2 * x)
[ 2, 4, 6 ]
```
인자가 하나더라도 ~~~ 면 괄호를 입력해야 합니다. 예를들어, 
As soon as there is anything else, you have to type the parentheses, even if there is only a single parameter. For example, you need parens if you destructure a single parameter:
```javascript
> [[1,2], [3,4]].map(([a,b]) => a + b)
[ 3, 7 ]
```
그리고 단일인자가 기본값을 가질 때도 괄호가 필요합니다.(undefined triggers the default value!):
```javascript
> [1, undefined, 3].map((x='yes') => x)
[ 1, 'yes', 3 ]
```

##13.4 어휘변수 
###13.4 변수값의 근원: 정적 vs 동적
변수가 값을 받을 수 있는 방법에는 두 가지가 있습니다.

첫째, 정적( lexically ) : 프로그램의 구조에 따라 그 값이 결정됩니다. 주변 스코프로 부터 그 값을 받습니다.
예:
```javascript
const x = 123;

function foo(y) {
    return x; // value received statically
}
```
둘째, 동적: 함수 호출에 의해 값을 받습니다.
예:

```javascript
function bar(arg) {
    return arg; // value received dynamically
}
```
###13.4.2 애로우함수에 있는 어휘 변수들
애로우함수가 일반함수와 구별되는 중요한 부분입니다.

* 일반함수는 동적 this를 갖습니다. 이 값은 호출되는 방식에 의해 결정됩니다.
* 애로우함수는 어휘적 this를 갖습니다. 이 값은 주변 스코프에 의해 결정됩니다.
The [complete list](http://exploringjs.com/es6/ch_arrow-functions.html) of variables whose values are determined lexically is:

* arguments
* super
* this
* new.target

##13.5 문법함정
가끔 실수를 유발하는 몇가지 문법적 세부사항이 있습니다.

###13.5.1 애로우함수는 매우 느슨하게 결합합니다.
Syntactically, arrow functions bind very loosely. The reason is that you want every expression that can appear in an expression body to “stick together”; it should bind more tightly than the arrow function:
```javascript
const f = x => (x % 2) === 0 ? x : 0;
```
결과적으로, (만약 다른곳에 표시한다면?? ) 종종 애로우함수를 괄호로 감싸야 합니다. if they appear somewhere else
예:
```javascript
console.log(typeof () => {}); // SyntaxError
console.log(typeof (() => {})); // OK
```
반대로, 
On the flip side, you can use typeof as an expression body without putting it in parens:
```javascript
const f = x => typeof x;
```
###13.5.2 애로우함수 인자 이후에 개행 금지
ES6는 애로우함수에서 인자선언과 본문 사이에 개행을 허용하지 않습니다.
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
이렇게 제한하는 이유는 향후 애로우함수
The rationale for this restriction is that it keeps the options open w.r.t. to “headless” arrow functions in the future: 

만약 인자가 없다면 당신은 괄호 생략이 가능합니다.if there are zero parameters, you’d be able to omit the parentheses.

###13.5.3 식문 같은 문법을 사용할 수 없습니다.
####13.5.3.1 식 vs 문
Quick review ( 더 많은 정보는 “자바스크립트를 말하다”를 찾아보세요 ):

표현식은 값으로 평가됩니다. 예:
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
대부분의 식은 문 위치에서 언급함으로서 간단히 문처럼 사용할 수 있습니다.
```javascript
function bar() {
    3 + 4;
    foo(7);
    'abc'.length;
}
```
####13.5.3.2 애로우함수의 본문
만약 애로우함수 본문이 식이라면 중괄호가 필요하지 않습니다.
```javascript
asyncFunc.then(x => console.log(x));
```
하지만 문은 중괄호를 넣어줘야 합니다.
```javascript
asyncFunc.catch(x => { throw x });
```
###13.5.4 Returning object literals
Having a block body in addition to an expression body means that  만약 식을 Object 리터럴로 사용하려면 괄호를 넣어야 합니다.

애로우 함수의 본문 블럭에는 label bar 그리고 123표현문이 있습니다.
```javascript
const f = x => { bar: 123 }
```
이 애로우함수의 본몬은 객체 리터럴 식입니다.
```javascript
const f = x => ({ bar: 123 })
```
##13.6 애로우함수 즉시실행
함수즉시실행 식을 기억합니까(IIFEs)? ECMAScript 5에선 block-scoping과 value-returning을 이용해 다음과 같이 가장합니다
```javascript
(function () { // open IIFE
    // inside IIFE
}()); // close IIFE
```
애로우함수 즉시실행을 사용하면 약간의 문자를 줄일 수 있습니다.
```javascript
(() => {
    return 123
})();
```
IIFEs도 마찬가지로, 두번 연속 호출로 해석되는 것을 피하기 위해(첫번째로는 함수, 두번째로는 인자) 세미콜론으로 IIAFs를 종료해야 합니다( 또는 상응하는 조치를 사용 ).

IIAF에 블럭이 있더라도 반드시 괄호로 감싸야 합니다. 왜냐하면 늦은 결합의 이유로 ( 직접적으로 ) 함수를 호출 할 수 없기 때문입니다. Note 괄호는 반드시 애로우함수 주변에 써야 합니다. IIFEs에서 어느 한쪽을 선택할 수 있습니다. 전제 문을 감싸던지 함수 표현만 감싸던지

/*
Even if the IIAF has a block body, you must wrap it in parentheses, because it can’t be (directly) function-called, due to how loosely it binds. Note that the parentheses must be around the arrow function. With IIFEs you have a choice: you can either put the parentheses around the whole statement or just around the function expression.
*/

앞 절에서 언급한 바와 같이 애로우함수의 느슨한 결합은 여기서 표현할 식본문에 유용합니다.( 뭔소린지 모르겠다.. )

/*
As mentioned in the previous section, arrow functions binding loosely is useful for expression bodies, where you want this expression:
*/
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
애로우 함수는 일반함수와 두가지가 다릅니다.

* 다음 구조는 어휘이다 : arguments, super, this, new.target
* 생성자로 사용할 수 없다 : 일반 함수는 내부함수[[Construct]]를 이용한 new와 prototype 속성을 지원합니다. 애로우함수는 둘다 지원하지 않습니다. new (() => {})할 때 에러가 발생합니다. 

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
