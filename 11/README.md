----


# 11. Parameter handling
인자 다루기

ECMAScript 6에서 인자 다루기는 크게 향상되었다. 이제 인자 기본값, 나머지인자(varargs : 가변인자?), 그리고 해체를 지원한다.

이 챕터에서는, 앞 장에서 나온 해체와 친숙하다면 굉장히 유용할 것이다.

##11.1 개요

기본 인자 값:

```javascript
 function findClosestShape(x=0, y=0) {
     // ...
 }
 ```

나머지 인자 :

```javascript
function format(pattern, ...params) {
    return params;
}
console.log(format('a', 'b', 'c')); // ['b', 'c']
```

해체에 의한 기명인자:

```javascript
function selectEntries({ start=0, end=-1, step=1 } = {}) {
    
    //객체 패턴의 약자 :
    // { start: start=0, end: end=-1, step: step=1 }

    //start, end, step을 여기서 쓰라..
    ···
}

selectEntries({ start: 10, end: 30, step: 2 });
selectEntries({ step: 3 });
selectEntries({});
selectEntries();
```

### 11.1.1 펼치기 연산자 (...)

함수나 생성자에서 호출시, 펼치기 연산자는 이터러블 값을 인자로 전환시킨다:

```javascript
> Math.max(-1, 5, 11, 3)
11
> Math.max(...[-1, 5, 11, 3])
11
> Math.max(-1, ...[-1, 5, 11], 3)
11
```

배열 리터럴에서 펼치기 연산자는 이터러블 값을 배열 요소로 변환한다.
```javascript
> [1, ...[2,3], 4]
[1, 2, 3, 4]
```
11.2 해체로 인자 다루기

ES6에서 인자 다룰 때 인자를 해체한 것과 선언된 인자를 동등하게 취급한다:

아래 함수 호출 예를 보자:

```javascript
function func(«FORMAL_PARAMETERS») {
    «CODE»
}
func(«ACTUAL_PARAMETERS»);
```

이는 대강 아래와 같다

```javascript
{
    let [«FORMAL_PARAMETERS»] = [«ACTUAL_PARAMETERS»];
    {
        «CODE»
    }
}
```

예를 들어 아래의 함수를 호출하면

```javascript
function logSum(x=0, y=0) {
    console.log(x + y);
}
logSum(7, 8);
```

아래와 같이 될 것이다.
```javascript
{
    let [x=0, y=0] = [7, 8];
    {
        console.log(x + y);
    }
}
```

이제 좀더 상세히 살펴보도록 하겠다.


## 11.3 인자의 기본 값
ECMAScript 6는 인자의 기본값을 특정할 수 있게 해준다.

```javascript
function f(x, y=0) {
  return [x, y];
}
```

두번째 인자를 생략함으로써 기본값이 발동되도록 한다.
```javascript
f(1)
> [1, 0]
f()
> [undefined, 0]
```

주의 - undefined 역시 기본값을 발동시킨다.

```javascript
f(undefined, undefined)
> [undefined, 0]
```

이 기본값은 실제 필요로 할때 요청에 따라 계산된다.

```javascript
const log = console.log.bind(console);
function g(x=log('x'), y=log('y')) {return 'DONE'}
> g()
x
y
'DONE'
> g(1)
y
'DONE'
> g(1, 2)
'DONE'
```

### 11.3.1 왜 undefined가 기본값 발동시키는가?

왜 undefined가 없는 인자 혹은 객체나 배열의 없는 부분처럼 처리되는지 아주 명확하지는 않다.
이에 대한 이론적인 근거는 기본값의 정의를 위임할 수 있다는 것이다. 아래의 두 예제를 보자.

첫번째 예제에서, 우리는 setOption()에서 기본값을 정의내릴 필요가 없다. 우리는 이 작업을 setLevel()에 위임할 수 있다.
```javascript
function setLevel(newLevel = 0) {
   light.intensity = newLevel;
}
function setOptions(options) {
   // 없는 속성은 undefined를 반환한다 --> 기본값 사용
    setLevel(options.dimmerLevel);
    setMotorSpeed(options.speed);
   ···
}
setOptions({speed:5});
```

두 번째 예제에서 squrare()는 x의 기본을 정의할 필요가 없다. 이는 multiply()에 위임하여 처리할 수 있다.
```javascript
function multiply(x=1, y=1) {
   return x * y;
}
function square(x) {
   return multiply(x, x);
}
```
기본값은 더 나아가 null이 빈 값을 가리키는 것으로, undefined는 존재하지 않는 것을 가리키는 역할로 자리잡게 한다.

### 11.3.2 Referring to other parameters in default values

기본값에 다른 인자 참조

Within a parameter default value, you can refer to any variable, including other parameters:

인자 기본값 내에서, 다른 인자를 포함안 어떤 변수든 참조할 수 있다.

```javascript
function foo(x=3, y=x) { ··· }
> foo();     // x=3; y=3
> foo(7);    // x=7; y=7
> foo(7, 2); // x=7; y=2
```

However, order matters: parameters are declared from left to right and within a default value, you get a ReferenceError if you access a parameter that hasn’t been declared, yet.

순서의 문제 : 인자는 왼쪽에서 오른쪽으로 정의되고 기본값 내에 있어, 아직 정의되지 않은 인자에 접근하게 되면 참조에러(ReferenceError)를 얻게 된다.


### 11.3.3 Referring to “inner” variables in default values
기본값에서 "내부의" 변수를 참조하기

Default values exist in their own scope, which is between the “outer” scope surrounding the function and the “inner” scope of the function body. Therefore, you can’t access “inner” variables from the default values:

기본값은 함수로 둘러쌓인 "외부" 영역 사이나 함수 바디의 "내부" 영역과 같은 고유한 영역에 존재한다. 고로 기본값으로부터 "내부"의
변수에 접근할 수 없다.


```javascript
const x = 'outer';
function foo(a = x) {
    const x = 'inner';
    console.log(a); // outer
}
```

If there were no outer x in the previous example, the default value x would produce a ReferenceError (if triggered).
This restriction is probably most surprising if default values are closures:

위 예제에 외부의 x가 없다면 기본값 x는 참조에러(ReferenceError)를 발생시킬 것이다. (만약 발생시킨다면)
만약 기본 값들이 닫혀있다면 이런 제약은 아마 아주 놀랍게도 :

```javascript
function bar(callback = () => QUX) {
   const QUX = 3; // can’t be accessed from default value //기본값으로부터 접근할 수 없다.
    callback();
}
bar(); // ReferenceError
```
To see why that is the case, consider the following implementation of bar() which is roughly equivalent to the previous one:

왜 저게 그렇게 되는지 보려면, 저것과 유사한 bar()의 구현으로 생각해볼 수 있음 : 
```javascript
function bar(...args) { // (A)
    const [callback = () => QUX] = args; // (B)
     { // (C)
   const QUX = 3; // can’t be accessed from default value // 기본값으로부터 접근할 수 없다.
      callback();
    }
}
```

Within the scope started by the opening curly brace at the end of line A, you can only refer to variables that are declared either in that scope or in a scope surrounding it. 
Therefore, variables declared in the scope starting in line C are out of reach for the statement in line B.

라인 A 끝쪽의 중괄호로 시작되는 영역에서 같은 영역 내에 있거나 그것으로 둘러쌓인 영역에 있는 변수만을 참조할 수 있다.

line A의 끝쪽에 중괄호로 시작된 scope 내에서, scope내부나, } 로 둘러쌓인 
따라서 lineC에서 시작되는 scope에 정의된 변수들은 lineB의 정의식에서는 접근할 수 없다.


## 11.4 Rest parameters
나머지 연산자
Putting the rest operator (...) in front of the last formal parameter means that it will receive all remaining actual parameters in an Array.

나머지 연산자(...)를 마지막 정규 인자 바로 앞에 둔다는 것은 배열의 남은 실제 인자를 모두 받아들이겠다는 것이다.
```javascript
function f(x, ...y) {
    ···
}
> f('a', 'b', 'c'); // x = 'a'; y = ['b', 'c']
```

If there are no remaining parameters, the rest parameter will be set to the empty Array:

만약 남아있는 인자가 없으면, 나머지 인자는 빈 배열이 될 것이다.
```javascript
> f(); // x = undefined; y = []
```
The spread operator (...) looks exactly like the rest operator, but it is used inside function calls and Array literals (not inside destructuring patterns).

펼침 연산자는 나머지 연산자와 매우 유사하지만, 이는 내부 함수 호출과 배열 리터럴에서만 쓰인다.(해체 패턴의 내부는 아님)


### 11.4.1 No more arguments!
더이상의 인자는 없다!

Rest parameters can completely replace JavaScript’s infamous special variable arguments. They have the advantage of always being Arrays:

나머지 인자는 완벽하게 javascript의 악명높은 가변인자(varargs)??를 대체할 수 있다. 언제나 배열로 존재한다는 장점을 가지고 있다.

// ECMAScript 5: arguments
ECMAScript 5의 argument
```javascript
 function logAllArguments() {
   for (var i=0; i < arguments.length; i++) {
      console.log(arguments[i]);
  }
}

// ECMAScript 6: rest parameter
ECMAScript6의 나머지 인자
function logAllArguments(...args) {
   for (const arg of args) {
         console.log(arg);
    }
}
```
#### 11.4.1.1 Combining destructuring and access to the destructured value
해체 결합과 해체 값에 접근

One interesting feature of arguments is that you can have normal parameters and an Array of all parameters at the same time:
변수의 한 가지 재미있는 면은 동시에 일반적인 parameter와 모든 parameter의 배열을 동시에 가질 수 있다는 점이다.

```javascript
function foo(x=0, y=0) {
    console.log('Arity: '+arguments.length);
    ···
}
```
You can avoid arguments in such cases if you combine a rest parameter with Array destructuring. The resulting code is longer, but more explicit:

나머지 인자와 배열 해체를 결합하는 경우 변수들을 피할 수 있다.  코드는 길어지지만 보다 명확하다.

```javascript
function foo(...args) {
    let [x=0, y=0] = args;
    console.log('Arity: '+args.length);
    ···
}
```
The same technique works for named parameters (options objects):

같은 기술이 명명 인자에서 동작함:

```javascript
function bar(options = {}) {
  let { namedParam1, namedParam2 } = options;
  ···
  if ('extra' in options) {
      ···
  }
}
```

#### 11.4.1.2 arguments is iterable
변수들은 이터러블하다.

arguments is iterable1 in ECMAScript 6, which means that you can use for-of and the spread operator:
ECMAScript 6에서는 변수는 이터러블하다. 이는 for-of와 펼침 연산자를 이용할 수 있음을 의미한다.

```javascript
> (function () { return typeof arguments[Symbol.iterator] }())
'function'
> (function () { return Array.isArray([...arguments]) }())
true
```

## 11.5 Simulating named parameters
명명 인자 시뮬레이팅

When calling a function (or method) in a programming language, you must map the actual parameters (specified by the caller) to the formal parameters (of a function definition). There are two common ways to do so:

Positional parameters are mapped by position. The first actual parameter is mapped to the first formal parameter, the second actual to the second formal, and so on.
Named parameters use names (labels) to perform the mapping. Names are associated with formal parameters in a function definition and label actual parameters in a function call.
It does not matter in which order named parameters appear, as long as they are correctly labeled.

함수(혹은 method)를 개발언어에서 호출할 때, 실제 인자-실제 인자(호출시에 정의되는거) 를 formal parameter-형식인자(function이 정의의)와 mapping해주어야 한다.(짝지어줘..)
두 가지 일반적인 방법이 있는데:

위치인자? 는 위치로 지정된다. 첫 번째 실제 인자는 첫번째 형식parameter와 짝지어지고, 두번째는 두번째 .. 계속 그렇게 됨.
명명된 parameter는 mapping을 위해 이름(label)을 이용한다. name은 선언된 함수의 formal parameter, 호출된 함수의 실제 인자 label과 연관된다.

이는 스펠링이 맞던지 명명된 파라미터가 나타난? 순서랑은 관계가 없다.


Named parameters have two main benefits: they provide descriptions for arguments in function calls and they work well for optional parameters. 

명명 인자는 두 가지 이점이 있는데: 함수 호출된 변수의 정의를 해주고 옵션인자에서 잘 동작한다. ?????????????????


I’ll first explain the benefits and then show you how to simulate named parameters in JavaScript via object literals.
장점을 보여주고 javascript에서 명명인자가 어떻게 쓰이는지 객체 리터럴을 이용해서 보여줄게.

### 11.5.1 Named Parameters as Descriptions
정의된 명명 인자

As soon as a function has more than one parameter, you might get confused about what each parameter is used for. 
For example, let’s say you have a function, selectEntries(), that returns entries from a database. Given the function call:

함수가 하나 이상의 인자를 가짐에 따라, 각 인자가 어디에 이용되는지 혼란스러울 수 있다.
예를 들어 데이터베이스로부터 entry를 반환하는 selectEntries()라는 함수 가지고 있다고 치면, function 호출:

```javascript
selectEntries(3, 20, 2);
```

what do these three numbers mean? Python supports named parameters, and they make it easy to figure out what is going on:

이 3 숫자들이 의미하는게 무엇일까? 파이썬은 네임드 인자를 지원하고, 또 무슨일이 벌어지는지 쉽게 알수 있게 해준다.

# Python syntax
# 파이썬 문법
```javascript
selectEntries(start=3, end=20, step=2)
```

### 11.5.2 Optional Named Parameters
선택적 네임드 파라미터

Optional positional parameters work well only if they are omitted at the end. Anywhere else, you have to insert placeholders such as null so that the remaining parameters have correct positions.

With optional named parameters, that is not an issue. You can easily omit any of them. Here are some examples:

선택적 위치의 네임드 파라미터는 마지막에 생략되었을 때는 아주 잘 동작한다. 다른 곳에서, 당신은 파라미터들이 올바른 위치를 유지할 수 있도록 null과 같은 자리표시 힌트를 넣어주어야 한다.

선택적 네임드 파라미터에서는 이것은 문제되지 않는다. 당신은 그 중 어떤것이라도 생략해도 된다. 여기 예제가 있는데:

# Python syntax
파이썬 문법
```javascript
selectEntries(step=2)
selectEntries(end=20, start=3)
selectEntries()
```

### 11.5.3 Simulating Named Parameters in JavaScript
자바스크립트에서 네임드 파라미터 써보기

JavaScript does not have native support for named parameters like Python and many other languages. 
But there is a reasonably elegant simulation: name parameters via an object literal, passed as a single actual parameter. When you use this technique, an invocation of selectEntries() looks as follows:

자바스크립트는 파이썬이나 다른 많은 언어처럼 네임드 파라미터를 근본적으로 지원하지는 않는다.
하지만 꽤 멋진 시뮬레이션이 있는데 : 
오브젝트 리터럴을 통한 네임드 파라미터는 하나의 실제 파라미터로 통한다. 니가 이 기술을 이용하면, selectEntries()의 호출은 아래와 같겠다:

```javascript
selectEntries({ start: 3, end: 20, step: 2 });
```

The function receives an object with the properties start, end, and step. You can omit any of them:
이 function은 start, end, step의 property를 가진 object를 받는다. 너는 이중에 암거나 생략할 수 있다:

```javascript
selectEntries({ step: 2 });
selectEntries({ end: 20, start: 3 });
selectEntries();
```

In ECMAScript 5, you’d implement selectEntries() as follows:
ECMAScript5에서는 selectEntries()를 아래처럼 구현했어야 했다:

```javascript
function selectEntries(options) {
    options = options || {};
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    ···
}
```
In ECMAScript 6, you can use destructuring, which looks like this:

ECMAScript6에서는 destructing을 이용할 수 있고, 아래와 같다:
```javascript
function selectEntries({ start=0, end=-1, step=1 }) {
    ···
}
```
If you call selectEntries() with zero arguments, the destructuring fails, because you can’t match an object pattern against undefined. That can be fixed via a default value. In the following code, the object pattern is matched against {} if there isn’t at least one argument.

만약 당신이 인자 없이 selectEntries()를 호출하면 destructuring은 실패한다, 왜냐면 undefined에 대해 오브젝트 패턴을 매치할 수 없기 때문이다.
이는 default value를 통해 고정할 수 있다. 다음의 코드에서,만약 단 하나의 인자도 없다면 오브젝트 패턴은 {}랑 매치된다. 

```javascript
function selectEntries({ start=0, end=-1, step=1 } = {}) {
    ···
}
```

You can also combine positional parameters with named parameters. It is customary for the latter to come last:

또한 네임드 파라미터와 positional 파라미터를 결합시킬 수 있다. 보통은 뒤에 온다????

```javascript
someFunc(posArg1, { namedArg1: 7, namedArg2: true });
```

In principle, JavaScript engines could optimize this pattern so that no intermediate object is created, because both the object literals at the call sites and the object patterns in the function definitions are static.
In JavaScript, the pattern for named parameters shown here is sometimes called options or option object (e.g., by the jQuery documentation).

원칙적으로, 자바스크립트 엔진은 이 패턴에 최적화 되어 어떠한 중간 객체도 생성하지 않는다. 왜냐면 호출하는 곳에서의 두 객체 리터럴과 함수 정의시의 객체 패턴은 정적이기 때문이다. 

자바스크립트에서, 여기서 보여주는 네임드 파라미터의 패턴은 옵션 혹은 옵션 객체라고 불리워진다.(제이쿼리 도큐먼트 예시)

##11.6 Examples of destructuring in parameter handling
파라미터 핸들링할때 destructuring 예시

###11.6.1 forEach() and destructuring
forEach()와 destructuring

You will probably mostly use the for-of loop in ECMAScript 6, but the Array method forEach() also profits from destructuring. Or rather, its callback does.
ECMAScript 6에서 아마 for-of loop문을 가장 빈번하게 쓸텐데, destructuring에서의 배열 메쏘드 forEach()  역시 쓸만하다. 특히 콜백.

First example: destructuring the Arrays in an Array.

첫 예제: 배열에서 배열의 destructuring
```javascript
const items = [ ['foo', 3], ['bar', 9] ];
items.forEach(([word, count]) => {
    console.log(word+' '+count);
});
```
Second example: destructuring the objects in an Array.
두번째 예제 : 배열에서 객체의 destructuring
```javascript
const items = [
    { word:'foo', count:3 },
    { word:'bar', count:9 },
];
items.forEach(({word, count}) => {
    console.log(word+' '+count);
});
```

### 11.6.2 Transforming Maps
맵의 변형

An ECMAScript 6 Map doesn’t have a method map() (like Arrays). Therefore, one has to:

Convert it to an Array of [key,value] pairs.
map() the Array.
Convert the result back to a Map.
This looks as follows.

ECMAScript 6의 맵은 (배열처럼) map() 이라는 메쏘드를 가지지 않는다. 그래서 할수 있는건 :
[key,value] 쌍의 배열로 전환 한다.
배열을 map()하고,
결과를 다시 Map으로 전환한다.
이를 아래에서 볼 수 있다.
```javascript
const map0 = new Map([
    [1, 'a'],
    [2, 'b'],
    [3, 'c'],
]);

const map1 = new Map( // step 3
    [...map0] // step 1
    .map(([k, v]) => [k*2, '_'+v]) // step 2
);

// Resulting Map: {2 -> '_a', 4 -> '_b', 6 -> '_c'}
```

### 11.6.3 Handling an Array returned via a Promise
Promise 를 통해 리턴되는 배열 핸들링

The tool method Promise.all() works as follows:

Input: an Array of Promises.
Output: a Promise that resolves to an Array as soon as the last input Promise is resolved. The Array contains the resolutions of the input Promises.
Destructuring helps with handling the Array that the result of Promise.all() resolves to:

Promise.all() 이라는 메쏘드는 아래와 같이 작동한다:

Input: Promises의 배열
Output : 배열에 들어간 프라미스의 마지막 프라미스가 들어가자 마자. 배열은 집어넣은 프라미스들을 지니게 됨.
Destructuring은 Promise.all() 의 결과로 만들어진 배열 핸들링을 하게 해준다 :
```javascript
const urls = [
    'http://example.com/foo.html',
    'http://example.com/bar.html',
    'http://example.com/baz.html',
];

Promise.all(urls.map(downloadUrl))
.then(([fooStr, barStr, bazStr]) => {
    ···
});

// This function returns a Promise that resolves to
// a string (the text)

//이 함수는 String으로 된 프라미스를 리턴함.

function downloadUrl(url) {
    return fetch(url).then(request => request.text());
}
```
fetch() is a Promise-based version of XMLHttpRequest. It is part of the Fetch standard.

fetch()는 XMLHttpRequest의 Promise 베이스 버전이다. Fetch 표준의 한 부분이다

## 11.7 Coding style tips
코딩 팁

This section mentions a few tricks for descriptive parameter definitions. 
They are clever, but they also have downsides: they add visual clutter and can make your code harder to understand.

이 섹션에서는 파라미터 정의의 기술적인 몇가지 트릭을 언급할것이다.
이는 괜찮긴한데, 단점도 있다: 시각적으로 혼란을 가중하고 니 코드를 이해하는데 더 어렵게 할 수 있다.

11.7.1 Optional parameters
옵션 파라미터

I occasionally use the parameter default value undefined to mark a parameter as optional (unless it already has a default value):

때때로 기본값 파라미터 undefined를 마크하기 위해 이용하기도 한다. (이미 default value를 가지지 않았을 때에만) :
```javascript
function foo(requiredParam, optionalParam = undefined) {
    ···
}
```

### 11.7.2 Required parameters
필수 파라미터

In ECMAScript 5, you have a few options for ensuring that a required parameter has been provided, which are all quite clumsy:

ECMAScript5 , 필수 파라미터가 제공되어짐을 보장하기 위해 몇가지 옵션을 가진다 . 이건 정말 모양빠진다:

```javascript
function foo(mustBeProvided) {
    if (arguments.length < 1) {
        throw new Error();
    }
    if (! (0 in arguments)) {
        throw new Error();
    }
    if (mustBeProvided === undefined) {
        throw new Error();
    }
    ···
}
```
In ECMAScript 6, you can (ab)use default parameter values to achieve more concise code (credit: idea by Allen Wirfs-Brock):

ECMAScript 6에서 좀더 간결한 코드를 위해 default파라미터를 이용(남용)할 수 있다. (Allen Wirfs-Broc이사람 아이디어임):

```javascript
/**
 * Called if a parameter is missing and
 * the default value is evaluated.
 * 파라미터가 없을때 호출하고 DEFAULT value가 평가됨.
 */
function mandatory() {
    throw new Error('Missing parameter');
}
function foo(mustBeProvided = mandatory()) {
    return mustBeProvided;
}
```
Interaction:
상호작용:
```javascript
> foo()
Error: Missing parameter
> foo(123)
123
```

### 11.7.3 Enforcing a maximum arity
최대의 인자 실행하기??

This section presents three approaches to enforcing a maximum arity. 
The running example is a function f whose maximum arity is 2 – if a caller provides more than 2 parameters, an error should be thrown.

The first approach collects all actual parameters in the formal rest parameter args and checks its length.

이 섹션에서는 최대 인자 실행을 위한 3가지 접근을 보여줄것이다. 
실행되는 예제는 최대 인자수가 2인 함수 f가 - 2 이상의 파라미터를 주면서 호출하면 에러를 던지게 될 것이다.

처음에는 정규의 rest 파라미터 args의 모든 실제 파라미터를 수집하고, 그것의 길이를 체크한다..

```javascript
function f(...args) {
    if (args.length > 2) {
        throw new Error();
    }
    // Extract the real parameters
    실제 파라미터를 산출한다.
    let [x, y] = args;
}
```
The second approach relies on unwanted actual parameters appearing in the formal rest parameter empty.

두번째는 원하지 않는 실제 파라미터가 rest 파라미터인 empty에서 나타났을 때
```javascript
function f(x, y, ...empty) {
    if (empty.length > 0) {
        throw new Error();
    }
}
```
The third approach uses a sentinel value that is gone if there is a third parameter. One caveat is that the default value OK is also triggered if there is a third parameter whose value is undefined.

세번째로는 3번째 인자로 보초값(플래그같은거?)를 이용한다. 3번째 파라미터가 undefined이면, OK라는 default value 역시 실행됨을 경고한다.
```javascript
const OK = Symbol();
function f(x, y, arity=OK) {
    if (arity !== OK) {
        throw new Error();
    }
}
```
Sadly, each one of these approaches introduces significant visual and conceptual clutter. I’m tempted to recommend checking arguments.length, but I also want arguments to go away.

안타깝게도, 각각의 방법들은 명확하게 시각적으로나 이론상으로 정신없다. 그래서 인자의 갯수를 체크하는걸 권하고 싶지만, 인자들이 가버렸으면 좋겠다????????
```javascript
function f(x, y) {
    if (arguments.length > 2) {
        throw new Error();
    }
}
```

## 11.8 The spread operator (...)
스프래드 연산자

The spread operator (...) looks exactly like the rest operator, but is its opposite:
The rest operator collects the remaining items of an iterable value into an Array and is used for rest parameters and destructuring.
The spread operator turns the items of an iterable value into arguments of a function call or into elements of an Array.

스프래드 연산자는 rest 연산자랑 완전 똑같아보이지만 완전 반대다:
rest연산는 배열의 반복 가능한 값의 나머지 항목을 수집하고 rest parameter와 destructing에 이용한다.
반복가능한 값의 항목들을 함수 호출 인자나, 배열의 요소로 전환시킨다.

### 11.8.1 Spreading into function and method calls
함수와 매쏘드 호출에서의 스프레딩~~

Math.max() is a good example for demonstrating how the spread operator works in method calls. 
Math.max(x1, x2, ···) returns the argument whose value is greatest. 
It accepts an arbitrary number of arguments, but can’t be applied to Arrays. The spread operator fixes that:

Math.max()는 메쏘드 호출시에 스프래드 연산자가 어떻게 작동하는지를 보여줄 아주 좋은 예제이다.
Math.max(x1, x2, ···)는 가장 높은 값의 인자를 반환한다. 
임의의 숫자 인자들을 받지만, 배열에는 적용할 수 없다. 스프래드 연산자로 그걸 수정해보면 :
```javascript
> Math.max(-1, 5, 11, 3)
11
> Math.max(...[-1, 5, 11, 3])
11
```
In contrast to the rest operator, you can use the spread operator anywhere in a sequence of parts:

rest연산자와는 대조적으로, 아무 순서에서나 스프래드 연산자를 이용할 수 있다.
```javascript
> Math.max(-1, ...[-1, 5, 11], 3)
11
```
Another example is JavaScript not having a way to destructively append the elements of one Array to another one. 

However, Arrays do have the method push(x1, x2, ···), which appends all of its arguments to its receiver. The following code shows how you can use push() to append the elements of arr2 to arr1.

또 다른 예시로, 자바스크립트는 한 배열이 다른 배열의 요소로 파괴?되어 붙을 수 있는 방법을 가지고 있지 않다.

하지만 Arrays는 push(x1, x2, ···)라는 메쏘드를 가지고 있는데, 이는 받는쪽에 모든 인자들을 붙일 수 있다. 아래의 코드는 arr2를 arr1에 어떻게 push()할 수 있는지 보여준다.
```javascript
const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

arr1.push(...arr2);
// arr1 is now ['a', 'b', 'c', 'd']
//arr1은 이제 ['a', 'b', 'c', 'd'] 임.
```

### 11.8.2 Spreading into constructors
생성자에 스프래딩..

In addition to function and method calls, the spread operator also works for constructor calls:
이외에도 함수나 메쏘드 호출에서, 생성자 호출에도 스프래드 인자는 동작한다.
```javascript
new Date(...[1912, 11, 24]) // Christmas Eve 1912
```
That is something that is difficult to achieve in ECMAScript 5.
이는 ECMAScript 5 로 하기에는 정말 어려운 대단한거임.


### 11.8.3 Spreading into Arrays
Arrays에 스프래딩

The spread operator can also be used inside Array literals:
Arrays 리터럴 안에도 스프래드 연산자를 쓸 수 있다.
```javascript
> [1, ...[2,3], 4]
[1, 2, 3, 4]
```
That gives you a convenient way to concatenate Arrays:
Arrays를 연결하는데 아주 편리함을 준다.
```javascript
const x = ['a', 'b'];
const y = ['c'];
const z = ['d', 'e'];

const arr = [...x, ...y, ...z]; // ['a', 'b', 'c', 'd', 'e']
```

One advantage of the spread operator is that its operand can be any iterable value (concat() does not support iteration).
스프래드 연산자의 한가지 이점은 그것의 피 연산자는 그 어떤 iterable 값도 될 수 있따는 것(concat() 은 iteration을 지원하지 않음).

#### 11.8.3.1 Converting iterable or Array-like objects to Arrays
iterable이나 배열스러운 객체를 Arrays로 전환.

The spread operator lets you convert any iterable value to an Array:
스프래드 연산자는 아무 iterable 값을 배열로 전환할 수 있게 해준다.
```javascript
const arr = [...someIterableObject];
```
Let’s convert a Set to an Array:
Set을 Array로 전환하자.
```javascript
const set = new Set([11, -1, 6]);
const arr = [...set]; // [11, -1, 6]
```

Your own iterable objects can be converted to Arrays in the same manner:
iterable 객체는 같은 방법으로 Arrays로 전환할 수 있다.
```javascript
const obj = {
    * [Symbol.iterator]() {
        yield 'a';
        yield 'b';
        yield 'c';
    }
};
const arr = [...obj]; // ['a', 'b', 'c']
```
Note that, just like the for-of loop, the spread operator only works for iterable values. 
Most important objects are iterable: Arrays, Maps, Sets and arguments. Most DOM data structures will also eventually be iterable.
Should you ever encounter something that is not iterable, but Array-like (indexed elements plus a property length), you can use Array.from()2 to convert it to an Array:


주목할 것은, for-of loop같이, 스프래드 연산자는 오로지 iterable 값에서만 동작한다.
가장 중요한 객체는 iterable : Arrays, Maps, Sets 그리고 arguments. 다. 대부분의 DOM 데이터 구조 역시 결과적으로는 이터러블 하다.

만약 iterable하지 않지만 배열스러운(length 속성을 가지고 순서를 가진 요소들을 가진) 무엇인가랑 맞딱뜨리게 된다면, Array.from()을 이용해서 Array로 변환시킬 수 있다.
```javascript
const arrayLike = {
    '0': 'a',
    '1': 'b',
    '2': 'c',
    length: 3
};

// ECMAScript 5:
var arr1 = [].slice.call(arrayLike); // ['a', 'b', 'c']

// ECMAScript 6:
const arr2 = Array.from(arrayLike); // ['a', 'b', 'c']

// TypeError: Cannot spread non-iterable value
const arr3 = [...arrayLike];
```
Iterables are explained in another chapter.

Iterables은 다른 챕터에서 설명될 것이다.

Explained in the chapter on Arrays.

Arrays에서 설명될 것이다.

