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
`These roles clash: Due to roles 2 and 3, functions always have their own this. But that prevents you from accessing the this of, e.g., a surrounding method from inside a callback (role 1).`

다음의 ES5코드에서 볼 수 있습니다.
`You can see that in the following ES5 code:`
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
`In line C, we’d like to access this.prefix, but can’t do that because the this of the function from line B shadows the this of the method from line A. In strict mode, this is undefined in non-method functions, which is why we get an error if we use Prefixer:`
```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
TypeError: Cannot read property 'prefix' of undefined
```
ECMAScript 5에 이 문제점을 피하는 방법에는 세가지가 있습니다.
`There are three ways to work around this problem in ECMAScript 5.`
### 13.2.1 해결책 1: that = this
명시적인 변수 that에 this를 할당할 수 있습니다. 즉, 아래  A라인과 같습니다. 
`You can assign this to a variable that isn’t shadowed. That’s what’s done in line A, below:`
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
`Now Prefixer works as expected:`
```javascript
> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
[ 'Hi Joe', 'Hi Alex' ]
```
### 13.2.2 해결책 2: this값 지정
일부 Array 메소드는 콜백을 호출 할 때 this 값을 지정하기 위한 추가 인자를 가지고 있습니다.
아래 A라인의 마지막 인자와 같습니다.
`A few Array methods have an extra parameter for specifying the value that this should have when invoking the callback. That’s the last parameter in line A, below.`
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
`You can use the method bind() to convert a function whose this is determined by how it is called (via call(), a function call, a method call, etc.) to a function whose this is always the same fixed value. That’s what we are doing in line A, below.`
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
`Arrow functions are basically solution 3, with a more convenient syntax. With an arrow function, the code looks as follows.`
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
`To fully ES6-ify the code, you’d use a class and a more compact variant of arrow functions:`
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
`In line A we save a few characters by tweaking two parts of the arrow function:`
* 함수 인자가 하나이고 그 인자의 식별자가 하나이면 괄호는 생략 가능합니다.`If there is only one parameter and that parameter is an identifier then the parentheses can be omitted.`
* 화살표 다음에 이어지는 하나의 표현식은 반환되어집니다.`An expression following the arrow leads to that expression being returned.`
코드에서, constructor와 prefixArray 메소드가 오브젝트리터럴에서 동작하는 더 간결한 ES6문법을 새롭게 사용하여 정의된 것 또한 볼수 있습니다.`In the code, you can also see that the methods constructor and prefixArray are defined using new, more compact ES6 syntax that also works in object literals.`

## 13.3 화살표함수 문법
뚱뚱한화살표 => ( 얇은화살표의 반대 -> )는 CoffeeScript와 호환되도록 채용 되었습니다. 애로우 함수와 매우 유사 합니다.`The “fat” arrow => (as opposed to the thin arrow ->) was chosen to be compatible with CoffeeScript, whose fat arrow functions are very similar.`

인자명시:`Specifying parameters:`
```javascript
    () => { ... } // 인자 없음
     x => { ... } // 인자 하나
(x, y) => { ... } // 인자 여러개
```
본문명시:`Specifying a body:`
```javascript
x => { return x * x }  // 블럭
x => x * x  // 위 라인과 같음
```
블럭구문은 보통 함수 본문처럼 동작 합니다. 예를들어 값을 반환하려면 return이 필요합니다. 
하나의 표현식본문, 그 표현식은 언제나 암시적으로 반환되어집니다.
`The statement block behaves like a normal function body. For example, you need return to give back a value. With an expression body, the expression is always implicitly returned.`

Note 표현식본문을 가진 애로우함수가 어느정도 장황함을 줄일 수 있는지 비교:
`Note how much an arrow function with an expression body can reduce verbosity. Compare:`
```javascript
const squares = [1, 2, 3].map(function (x) { return x * x });
const squares = [1, 2, 3].map(x => x * x);
```

### 13.3.1 단일인자 주변 괄호생략
인자 주변에 괄호는 단일 식별자로 이뤄졌을 때만 생략 가능합니다.
`Omitting the parentheses around the parameters is only possible if they consist of a single identifier:`
```javascript
> [1,2,3].map(x => 2 * x)
[ 2, 4, 6 ]
```
그 밖의 경우가 생기면, 비록 단일인자라 하더라도 괄호를 입력해야만 합니다. 예를들어, 단일인자를 분해한다면 괄호가 필요합니다:
`As soon as there is anything else, you have to type the parentheses, even if there is only a single parameter. For example, you need parens if you destructure a single parameter:`
```javascript
> [[1,2], [3,4]].map(([a,b]) => a + b)
[ 3, 7 ]
```
그리고 단일인자가 기본값을 가질 때도 괄호가 필요합니다.(undefined는 기본값을 트리거 합니다!):
`And you need parens if a single parameter has a default value (undefined triggers the default value!):`
```javascript
> [1, undefined, 3].map((x='yes') => x)
[ 1, 'yes', 3 ]
```

## 13.4 Lexical변수들
`13.4 Lexical variables`
### 13.4 변수값의 출처: 정적 vs 동적
`13.4.1 Sources of variable values: static versus dynamic`
변수가 값을 받는 두가지 방법은 다음과 같습니다.
`The following are two ways in which a variable can receive its value.`

첫째, 정적(어휘) : 변수의 값이 프로그램의 구조에 의해 결정됩니다; 변수의 값은 감싸는 스코프로 부터 받습니다. 
예를들어:
`First, statically (lexically): Its value is determined by the structure of the program; it receives its value from a surrounding scope. For example:`
```javascript
const x = 123;

function foo(y) {
    return x; // 값이 정적으로 받아졌다.
}
```
둘째, 동적: 변수의 값은 함수 호출에 의해 받습니다.
예를들어:
`Second, dynamically: It receives its value via a function call. For example:`
```javascript
function bar(arg) {
    return arg; // 값이 동적으로 받아졌다.
}
```
### 13.4.2 화살표함수 lexical에 있는 변수들
`13.4.2 Variables that are lexical in arrow functions`
this의 출처는 다른것과 중요하게 구별되는 화살표 함수의 양상입니다.`The source of this is an important distinguishing aspect of arrow functions:`
* 이전함수는 동적 this를 갖습니다; this의 값은 함수들이 호출되어진 방법에 의해 결정됩니다.`Traditional functions have a dynamic this; its value is determined by how they are called.`
* 화살표함수는 lexical this를 갖습니다; 그것의 값은 감싸는 스코프에 의해 결정됩니다.`Arrow functions have a lexical this; its value is determined by the surrounding scope.`

변수의 값이 lexically로 결정되는 [전체목록](http://exploringjs.com/es6/ch_arrow-functions.html) 입니다.
`The complete list of variables whose values are determined lexically is:`
* arguments
* super
* this
* new.target

## 13.5 문법함정
`13.5 Syntax pitfalls`
가끔 실수를 유발할 수 있는 몇 가지 문법관련 세부사항이 있습니다.
`There are a few syntax-related details that can sometimes trip you up.`

### 13.5.1 화살표함수는 매우 느슨하게 연결됩니다.
`13.5.1 Arrow functions bind very loosely`
문법상, 화살표함수는 매우 느슨하게 연결됩니다. 그 이유는 당신은 모든 표현식을 표현식본문에 뭉쳐서 나타낼 수 있길 원하기 때문입니다.
표현식은 화살표함수보다 더 우선적으로 결합될 것입니다.

```javascript
const f = x => (x % 2) === 0 ? x : 0;
```
그 결과, 만약 또 다른곳에서 표시 한다면 당신은 자주 화살표함수를 괄호 안으로 감싸야 합니다.
예를들어:
`As a consequence, you often have to wrap arrow functions in parentheses if they appear somewhere else. For example:`
```javascript
console.log(typeof () => {}); // SyntaxError
console.log(typeof (() => {})); // OK
```
반면, 표현식본문을 괄호에 넣지 않는 것 처럼 typeof를 사용할 수 있습니다.
`On the flip side, you can use typeof as an expression body without putting it in parens:`
```javascript
const f = x => typeof x;
```
### 13.5.2 화살표함수 인자 뒤에서 개행 금지
`13.5.2 No line break after arrow function parameters`
ES6는 화살표함수의 인자선언과 본문 사이에 개행을 허용하지 않습니다.
`ES6 forbids a line breaks between the parameter definitions and the body of an arrow function:`
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
`The rationale for this restriction is that it keeps the options open w.r.t. to “headless” arrow functions in the future: if there are zero parameters, you’d be able to omit the parentheses.`

### 13.5.3 표현식의 본문으로 문을 사용할 수 없습니다.
`13.5.3 You can’t use statements as expression bodies`
#### 13.5.3.1 표현식 vs 문
`13.5.3.1 Expressions versus statements`
Quick review ( 더 많은 정보는 “자바스크립트를 말하다”를 찾아보세요 ):
`Quick review (consult “Speaking JavaScript” for more information):`

표현식은 값들을 생성합니다(평가됩니다). 예:
`Expressions produce (are evaluated to) values. Examples:`
```javascript
3 + 4
foo(7)
'abc'.length
```
문은 동작을 합니다. 예:
`Statements do things. Examples:`
```javascript
while (true) { ··· }
return 123;
```
대부분의 표현식은 단순히 문안에서 언급함으로써 문처럼 사용할 수 있습니다.
`Most expressions1 can be used as statements, simply by mentioning them in statement positions:`
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
`If an expression is the body of an arrow function, you don’t need braces:`
```javascript
asyncFunc.then(x => console.log(x));
```
하지만 문은 중괄호안에 넣어줘야 합니다.
`However, statements have to be put in braces:`
```javascript
asyncFunc.catch(x => { throw x });
```
### 13.5.4 오브젝트리터럴 리턴하기
`13.5.4 Returning object literals`
만약 표현식본문이 오브젝트 리터럴이길 원한다면 블럭몸체와 더블어 표현식본문을 괄호 안으로 넣어야 합니다.
`Having a block body in addition to an expression body means that if you want the expression body to be an object literal, you have to put it in parentheses.`
이 화살표함수의 본문은 블럭에 레이블 bar와  123 서술 표현식입니다.
`The body of this arrow function is a block with the label bar and the expression statement 123.`
```javascript
const f = x => { bar: 123 }
```
이 화살표 함수의 본문은 하나의 오브젝트리터럴 표현식입니다.
`The body of this arrow function is an expression, an object literal:`
```javascript
const f = x => ({ bar: 123 })
```
## 13.6 화살표함수 즉시실행
`13.6 Immediately-invoked arrow functions`
함수즉시실행 표현식을 기억합니까(IIFEs)? 다음과 같이 보면 ECMAScript 5 에서 블록스코핑과 값 반환 블록을 흉내내기 위해 사용하곤 했다.
`Remember Immediately Invoked Function Expressions (IIFEs)? They look as follows and are used to simulate block-scoping and value-returning blocks in ECMAScript 5:``
```javascript
(function () { // open IIFE
    // inside IIFE
}()); // close IIFE
```
화살표함수 즉시실행을(IIAF) 사용하면 약간의 문자를 줄일 수 있습니다.
`You can save a few characters if you use an Immediately Invoked Arrow Function (IIAF):`
```javascript
(() => {
    return 123
})();
```
IIFEs와 마찬가지로, IIAFs가 두번연속 함수호출 처럼 해석되는(첫번째로는 함수, 두번째로는 인자) 것을 방지하기 위해 IIAFs도 세미콜론으로 끝내야 합니다( 또는 상응하는 조치를 사용 )
`Similarly to IIFEs, you should terminate IIAFs with semicolons (or use an equivalent measure), to avoid two consecutive IIAFs being interpreted as a function call (the first one as the function, the second one as the parameter).`

IIAF에 블럭몸체가 있더라도 반드시 괄호로 감싸야 합니다. 왜냐하면 늦은 연결의 이유로 함수를 (직접적으로) 호출 할 수 없기 때문입니다. Note 괄호는 반드시 화살표함수 주변에 써야 합니다. IIFEs에서는 전제 문을 감싸거나 함수 표현만 감싸거나 어느 한쪽을 선택할 수 있습니다.  
`Even if the IIAF has a block body, you must wrap it in parentheses, because it can’t be (directly) function-called, due to how loosely it binds. Note that the parentheses must be around the arrow function. With IIFEs you have a choice: you can either put the parentheses around the whole statement or just around the function expression.`

앞 절에서 언급한 바와 같이 화살표함수의 느슨한 결합은 여기의 표현식 처럼 본문을 표현하는데 하는데 도움이 됩니다.
`As mentioned in the previous section, arrow functions binding loosely is useful for expression bodies, where you want this expression:`
```javascript
const value = () => foo()
```
이처럼 해석
`to be interpreted as:`
```javascript
const value = () => (foo())
```
이것과는 다름
`and not as:`
```javascript
const value = (() => foo)()
```
[A section in the chapter on callable entities](http://exploringjs.com/es6/ch_callables.html#sec_iifes-in-es6) 
에서 ES6의 IIFE과 IIAF를 이용한 더 많은 정보를 접할 수 있습니다.
`A section in the chapter on callable entities has more information on using IIFEs and IIAFs in ES6.`


## 13.7 화살표함수 vs 보통함수
`13.7 Arrow functions versus normal functions`
화살표함수는 보통 함수와 단지 두가지만 다릅니다.
`An arrow function is different from a normal function in only two ways:`

* 다음구성은 lexical이다 : arguments, super, this, new.target `The following constructs are lexical: arguments, super, this, new.target`
* 생성자로 사용할 수 없다 : 보통함수는 내부메소드인[[Construct]]과 prototype에 의한 new를 지원합니다. 화살표함수는 둘 다 지원하지 않아  new (() => {})할 때 에러가 발생합니다. `It can’t be used as a constructor: Normal functions support new via the internal method [[Construct]] and the property prototype. Arrow functions have neither, which is why new (() => {}) throws an error.`

그 밖에 화살표함수와 일반함수 사이에 크게 주목할만한 차이점은 없습니다.
예를들어 typeof와 instanceof 는 같은 결과입니다.
`Apart from that, there are no observable differences between an arrow function and a normal function. For example, typeof and instanceof produce the same results:`
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
`Consult the chapter on callable entities for more information on when to use arrow functions and when to use traditional functions.`
