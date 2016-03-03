13. Arrow functions

13.1 Overview

화살표함수는 두가지 장점이 있습니다.

첫째, 일반적인 함수 보다 간략합니다.

const arr = [1, 2, 3];
const squares = arr.map(x => x * x);

// Traditional function expression:
const squares = arr.map(function (x) { return x * x });


둘째,  this를 lexical 환경에서 찾습니다. 따라서 당신은 더 이상 bind() 또는 that = this가 필요없습니다.

function UiComponent() {
    const button = document.getElementById('myButton');
    button.addEventListener('click', () => {
        console.log('CLICK');
        this.handleClick(); // lexical `this`
    });
}

다음의 변수는 모두 화살표 함수 어휘안에 포함됩니다.

* arguments
* super
* this
* new.target


13.2 none-method 일반 함수가 나쁜건 this 때문이다

자바스크립트에서 일반함수는 다음과 같이 사용할 수 있습니다.

1. Non-method 함수
2. 메소드
3. 생성자


역할충돌 : 역할2, 3번 함수는 항상 this를 갖는 반면 함수 내부의 콜백( 역할1 )에선  this를 접근할 수 없습니다.

다음과 같은 ES5코드에서 확인할 수 있습니다.

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


C라인에서 우리는 this.prefix를 접근하고 싶지만 A라인 메소드의 this는 B라인의 함수에서 가려지기 때문에 그럴 수 없습니다.
우리가 Prefixer를 사용할 때 에러가 나는 이유는 strict mode에서 none-method함수의 this는 undefined이기 때문입니다.

> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
TypeError: Cannot read property 'prefix' of undefined


13.2.1 Solution 1: that = this
당신은 this를 명시적으로 변수에 할당할 수 있습니다. 즉, 아래  A라인과 같습니다. 

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    var that = this; // (A)
    return arr.map(function (x) {
        return that.prefix + x;
    });
};

예상대로  Prefixer가 동작합니다.

> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
[ 'Hi Joe', 'Hi Alex' ]


13.2.2 Solution 2: this값 지정

일부 Array 메소드는 콜백을 호출 할 때 this 값을 지정하기 위한 추가 인자를 가지고 있습니다.
아래 A라인의 마지막 인자와 같습니다.

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }, this); // (A)
};


13.2.3 Solution 3: bind(this)

You can use the method bind() to convert a function whose this is determined by how it is called (via call(), a function call, a method call, etc.) to a function whose this is always the same fixed value. That’s what we are doing in line A, below.

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }.bind(this)); // (A)
};


13.2.4 ECMAScript 6 solution: arrow functions



