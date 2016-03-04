----


11. Parameter handling

Parameter handling has been significantly upgraded in ECMAScript 6. It now supports parameter default values, rest parameters (varargs) and destructuring.
ECMAScript 6에서 parameter handling은 크게 향상되었다. 이제 parameter 기본값, rest parameter, 그리고 destructing을 지원한다.

For this chapter, it is useful to be familiar with destructuring (which is explained in the previous chapter).

앞 장에서 나온 

11.1 Overview
Default parameter values:

function findClosestShape(x=0, y=0) {
    // ...
}
Rest parameters:

function format(pattern, ...params) {
    return params;
}
console.log(format('a', 'b', 'c')); // ['b', 'c']
Named parameters via destructuring:

function selectEntries({ start=0, end=-1, step=1 } = {}) {
    // The object pattern is an abbreviation of:
    // { start: start=0, end: end=-1, step: step=1 }

    // Use the variables `start`, `end` and `step` here
    ···
}

selectEntries({ start: 10, end: 30, step: 2 });
selectEntries({ step: 3 });
selectEntries({});
selectEntries();


11.1.1 Spread operator (...)

Spread operator (...)

In function and constructor calls, the spread operator turns iterable values into arguments:

functions이나 생성자에서 호출시, 이 spread operator는 iterable value를 argument로 변환한다.


> Math.max(-1, 5, 11, 3)
11
> Math.max(...[-1, 5, 11, 3])
11
> Math.max(-1, ...[-1, 5, 11], 3)
11
In Array literals, the spread operator turns iterable values into Array elements:

배열 리터럴에서 spread operator는 iterable 변수들을 배열 element로 변환한다.

> [1, ...[2,3], 4]
[1, 2, 3, 4]
11.2 Parameter handling as destructuring
The ES6 way of handling parameters is equivalent to destructuring the actual parameters via the formal parameters. That is, the following function call:

function func(«FORMAL_PARAMETERS») {
    «CODE»
}
func(«ACTUAL_PARAMETERS»);
is roughly equivalent to:

이는 대강 아래와 같다

{
    let [«FORMAL_PARAMETERS»] = [«ACTUAL_PARAMETERS»];
    {
        «CODE»
    }
}
Example – the following function call:

예를 들어 아래의 function을 호출하면

function logSum(x=0, y=0) {
    console.log(x + y);
}
logSum(7, 8);
becomes:

이렇게 된다.

{
    let [x=0, y=0] = [7, 8];
    {
        console.log(x + y);
    }
}
Let’s look at specific features next.

다음에서는  을 보도록 하자

11.3 Parameter default values
ECMAScript 6 lets you specify default values for parameters:
ECMAScript 6는 parameter를 위해 구체적인 default value를 가능하게 해준다???????

function f(x, y=0) {
  return [x, y];
}
Omitting the second parameter triggers the default value:
두번째 parameter를 생략함으로써 default value가 되도록 한다.

> f(1)
[1, 0]
> f()
[undefined, 0]
Watch out – undefined triggers the default value, too:

undefined 역시 default value를 발생시키는 것을 볼 수 있다.

> f(undefined, undefined)
[undefined, 0]
The default value is computed on demand, only when it is actually needed:

이 default value는 실질적인 필요가 있을때에는 언제든지 산출된다.


> const log = console.log.bind(console);
> function g(x=log('x'), y=log('y')) {return 'DONE'}
> g()
x
y
'DONE'
> g(1)
y
'DONE'
> g(1, 2)
'DONE'
11.3.1 Why does undefined trigger default values?

왜 undefined가 default value를 발생시키는가?

It isn’t immediately obvious why undefined should be interpreted as a missing parameter or a missing part of an object or Array.
The rationale for doing so is that it enables you to delegate the definition of default values. Let’s look at two examples.

왜 undefined가 missing parameter 혹은 object나 array의 missing part처럼 interprete되는지 아주 명확하지는 않다.
이에 대한 이론적인 근거는 default value의 정의를 위임할 수 있다는 것이다. 아래의 두 예제를 보자.


In the first example (source: Rick Waldron’s TC39 meeting notes from 2012-07-24), we don’t have to define a default value in setOptions(), we can delegate that task to setLevel().

첫번째 예제에서, 우리는 setOption()에서 default value를 정의내릴 필요가 없다. 우리는 이 작업을 setLevel()에 위임할 수 있다.

function setLevel(newLevel = 0) {
    light.intensity = newLevel;
}
function setOptions(options) {
    // Missing prop returns undefined => use default
    setLevel(options.dimmerLevel);
    setMotorSpeed(options.speed);
    ···
}
setOptions({speed:5});
In the second example, square() doesn’t have to define a default for x, it can delegate that task to multiply():

두 번째 예제에서 squrare()는 x의 default를 정의할 필요가 없다. 이는 multiply()에 위임하여 처리할 수 있다.

function multiply(x=1, y=1) {
    return x * y;
}
function square(x) {
    return multiply(x, x);
}
Default values further entrench the role of undefined as indicating that something doesn’t exist, versus null indicating emptiness.

null이 빈 값을 지칭하는 것과는 대조적으로. default value는 undefined가 존재하지 않는 어떤 것을 보여주는 역할로 자리잡게 해준다.


11.3.2 Referring to other parameters in default values

default value에서 다른 파라미터 참조하기

Within a parameter default value, you can refer to any variable, including other parameters:

default value parameter에서 당신은 다른 parameter를 포함한 어떤 변수든지 참조할 수 있다.

function foo(x=3, y=x) { ··· }
foo();     // x=3; y=3
foo(7);    // x=7; y=7
foo(7, 2); // x=7; y=2
However, order matters: parameters are declared from left to right and within a default value, you get a ReferenceError if you access a parameter that hasn’t been declared, yet.
??
그러나 order matter는 parameter는 왼쪽에서 오른쪽으로 정의되어야 하는데 default value내에서는 아직 정의되지 않은 parameter에 접근했다는 referenceError를 얻게된다.

11.3.3 Referring to “inner” variables in default values
default value에서 내부에 있는 변수들을 참조하기

Default values exist in their own scope, which is between the “outer” scope surrounding the function and the “inner” scope of the function body. Therefore, you can’t access “inner” variables from the default values:
????
default value는 그들의 고유한 scope에서 존재한다. 이 영역은 function으로 둘러싸인 외부 영역과 functions의 내부 영역 사이에 있다.
따라서 당신은 default value로부터 내부 변수에 접근할 수 없는 것이다.



const x = 'outer';
function foo(a = x) {
    const x = 'inner';
    console.log(a); // outer
}
If there were no outer x in the previous example, the default value x would produce a ReferenceError (if triggered).
위 예시에서 만약에 외부의 x가 없었다면 default value x는 ReferenceError를 발생시킬 것이다.


This restriction is probably most surprising if default values are closures:







function bar(callback = () => QUX) {
    const QUX = 3; // can’t be accessed from default value
    callback();
}
bar(); // ReferenceError
To see why that is the case, consider the following implementation of bar() which is roughly equivalent to the previous one:

function bar(...args) { // (A)
    const [callback = () => QUX] = args; // (B)
    { // (C)
        const QUX = 3; // can’t be accessed from default value
        callback();
    }
}
Within the scope started by the opening curly brace at the end of line A, you can only refer to variables that are declared either in that scope or in a scope surrounding it. Therefore, variables declared in the scope starting in line C are out of reach for the statement in line B.

11.4 Rest parameters
Putting the rest operator (...) in front of the last formal parameter means that it will receive all remaining actual parameters in an Array.

function f(x, ...y) {
    ···
}
f('a', 'b', 'c'); // x = 'a'; y = ['b', 'c']
If there are no remaining parameters, the rest parameter will be set to the empty Array:

f(); // x = undefined; y = []
The spread operator (...) looks exactly like the rest operator, but it is used inside function calls and Array literals (not inside destructuring patterns).

11.4.1 No more arguments!
Rest parameters can completely replace JavaScript’s infamous special variable arguments. They have the advantage of always being Arrays:

// ECMAScript 5: arguments
function logAllArguments() {
    for (var i=0; i < arguments.length; i++) {
        console.log(arguments[i]);
    }
}

// ECMAScript 6: rest parameter
function logAllArguments(...args) {
    for (const arg of args) {
        console.log(arg);
    }
}
11.4.1.1 Combining destructuring and access to the destructured value
One interesting feature of arguments is that you can have normal parameters and an Array of all parameters at the same time:

function foo(x=0, y=0) {
    console.log('Arity: '+arguments.length);
    ···
}
You can avoid arguments in such cases if you combine a rest parameter with Array destructuring. The resulting code is longer, but more explicit:

function foo(...args) {
    let [x=0, y=0] = args;
    console.log('Arity: '+args.length);
    ···
}
The same technique works for named parameters (options objects):

function bar(options = {}) {
    let { namedParam1, namedParam2 } = options;
    ···
    if ('extra' in options) {
        ···
    }
}
11.4.1.2 arguments is iterable
arguments is iterable1 in ECMAScript 6, which means that you can use for-of and the spread operator:

> (function () { return typeof arguments[Symbol.iterator] }())
'function'
> (function () { return Array.isArray([...arguments]) }())
true
11.5 Simulating named parameters
When calling a function (or method) in a programming language, you must map the actual parameters (specified by the caller) to the formal parameters (of a function definition). There are two common ways to do so:

Positional parameters are mapped by position. The first actual parameter is mapped to the first formal parameter, the second actual to the second formal, and so on.
Named parameters use names (labels) to perform the mapping. Names are associated with formal parameters in a function definition and label actual parameters in a function call. It does not matter in which order named parameters appear, as long as they are correctly labeled.
Named parameters have two main benefits: they provide descriptions for arguments in function calls and they work well for optional parameters. I’ll first explain the benefits and then show you how to simulate named parameters in JavaScript via object literals.

11.5.1 Named Parameters as Descriptions
As soon as a function has more than one parameter, you might get confused about what each parameter is used for. For example, let’s say you have a function, selectEntries(), that returns entries from a database. Given the function call:

selectEntries(3, 20, 2);
what do these three numbers mean? Python supports named parameters, and they make it easy to figure out what is going on:

# Python syntax
selectEntries(start=3, end=20, step=2)
11.5.2 Optional Named Parameters
Optional positional parameters work well only if they are omitted at the end. Anywhere else, you have to insert placeholders such as null so that the remaining parameters have correct positions.

With optional named parameters, that is not an issue. You can easily omit any of them. Here are some examples:

# Python syntax
selectEntries(step=2)
selectEntries(end=20, start=3)
selectEntries()
11.5.3 Simulating Named Parameters in JavaScript
JavaScript does not have native support for named parameters like Python and many other languages. But there is a reasonably elegant simulation: name parameters via an object literal, passed as a single actual parameter. When you use this technique, an invocation of selectEntries() looks as follows:

selectEntries({ start: 3, end: 20, step: 2 });
The function receives an object with the properties start, end, and step. You can omit any of them:

selectEntries({ step: 2 });
selectEntries({ end: 20, start: 3 });
selectEntries();
In ECMAScript 5, you’d implement selectEntries() as follows:

function selectEntries(options) {
    options = options || {};
    var start = options.start || 0;
    var end = options.end || -1;
    var step = options.step || 1;
    ···
}
In ECMAScript 6, you can use destructuring, which looks like this:

function selectEntries({ start=0, end=-1, step=1 }) {
    ···
}
If you call selectEntries() with zero arguments, the destructuring fails, because you can’t match an object pattern against undefined. That can be fixed via a default value. In the following code, the object pattern is matched against {} if there isn’t at least one argument.

function selectEntries({ start=0, end=-1, step=1 } = {}) {
    ···
}
You can also combine positional parameters with named parameters. It is customary for the latter to come last:

someFunc(posArg1, { namedArg1: 7, namedArg2: true });
In principle, JavaScript engines could optimize this pattern so that no intermediate object is created, because both the object literals at the call sites and the object patterns in the function definitions are static.

In JavaScript, the pattern for named parameters shown here is sometimes called options or option object (e.g., by the jQuery documentation).

11.6 Examples of destructuring in parameter handling
11.6.1 forEach() and destructuring
You will probably mostly use the for-of loop in ECMAScript 6, but the Array method forEach() also profits from destructuring. Or rather, its callback does.

First example: destructuring the Arrays in an Array.

const items = [ ['foo', 3], ['bar', 9] ];
items.forEach(([word, count]) => {
    console.log(word+' '+count);
});
Second example: destructuring the objects in an Array.

const items = [
    { word:'foo', count:3 },
    { word:'bar', count:9 },
];
items.forEach(({word, count}) => {
    console.log(word+' '+count);
});
11.6.2 Transforming Maps
An ECMAScript 6 Map doesn’t have a method map() (like Arrays). Therefore, one has to:

Convert it to an Array of [key,value] pairs.
map() the Array.
Convert the result back to a Map.
This looks as follows.

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
11.6.3 Handling an Array returned via a Promise
The tool method Promise.all() works as follows:

Input: an Array of Promises.
Output: a Promise that resolves to an Array as soon as the last input Promise is resolved. The Array contains the resolutions of the input Promises.
Destructuring helps with handling the Array that the result of Promise.all() resolves to:

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
function downloadUrl(url) {
    return fetch(url).then(request => request.text());
}
fetch() is a Promise-based version of XMLHttpRequest. It is part of the Fetch standard.

11.7 Coding style tips
This section mentions a few tricks for descriptive parameter definitions. They are clever, but they also have downsides: they add visual clutter and can make your code harder to understand.

11.7.1 Optional parameters
I occasionally use the parameter default value undefined to mark a parameter as optional (unless it already has a default value):

function foo(requiredParam, optionalParam = undefined) {
    ···
}
11.7.2 Required parameters
In ECMAScript 5, you have a few options for ensuring that a required parameter has been provided, which are all quite clumsy:

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
In ECMAScript 6, you can (ab)use default parameter values to achieve more concise code (credit: idea by Allen Wirfs-Brock):

/**
 * Called if a parameter is missing and
 * the default value is evaluated.
 */
function mandatory() {
    throw new Error('Missing parameter');
}
function foo(mustBeProvided = mandatory()) {
    return mustBeProvided;
}
Interaction:

> foo()
Error: Missing parameter
> foo(123)
123
11.7.3 Enforcing a maximum arity
This section presents three approaches to enforcing a maximum arity. The running example is a function f whose maximum arity is 2 – if a caller provides more than 2 parameters, an error should be thrown.

The first approach collects all actual parameters in the formal rest parameter args and checks its length.

function f(...args) {
    if (args.length > 2) {
        throw new Error();
    }
    // Extract the real parameters
    let [x, y] = args;
}
The second approach relies on unwanted actual parameters appearing in the formal rest parameter empty.

function f(x, y, ...empty) {
    if (empty.length > 0) {
        throw new Error();
    }
}
The third approach uses a sentinel value that is gone if there is a third parameter. One caveat is that the default value OK is also triggered if there is a third parameter whose value is undefined.

const OK = Symbol();
function f(x, y, arity=OK) {
    if (arity !== OK) {
        throw new Error();
    }
}
Sadly, each one of these approaches introduces significant visual and conceptual clutter. I’m tempted to recommend checking arguments.length, but I also want arguments to go away.

function f(x, y) {
    if (arguments.length > 2) {
        throw new Error();
    }
}
11.8 The spread operator (...)
The spread operator (...) looks exactly like the rest operator, but is its opposite:

The rest operator collects the remaining items of an iterable value into an Array and is used for rest parameters and destructuring.
The spread operator turns the items of an iterable value into arguments of a function call or into elements of an Array.
11.8.1 Spreading into function and method calls
Math.max() is a good example for demonstrating how the spread operator works in method calls. Math.max(x1, x2, ···) returns the argument whose value is greatest. It accepts an arbitrary number of arguments, but can’t be applied to Arrays. The spread operator fixes that:

> Math.max(-1, 5, 11, 3)
11
> Math.max(...[-1, 5, 11, 3])
11
In contrast to the rest operator, you can use the spread operator anywhere in a sequence of parts:

> Math.max(-1, ...[-1, 5, 11], 3)
11
Another example is JavaScript not having a way to destructively append the elements of one Array to another one. However, Arrays do have the method push(x1, x2, ···), which appends all of its arguments to its receiver. The following code shows how you can use push() to append the elements of arr2 to arr1.

const arr1 = ['a', 'b'];
const arr2 = ['c', 'd'];

arr1.push(...arr2);
// arr1 is now ['a', 'b', 'c', 'd']
11.8.2 Spreading into constructors
In addition to function and method calls, the spread operator also works for constructor calls:

new Date(...[1912, 11, 24]) // Christmas Eve 1912
That is something that is difficult to achieve in ECMAScript 5.

11.8.3 Spreading into Arrays
The spread operator can also be used inside Array literals:

> [1, ...[2,3], 4]
[1, 2, 3, 4]
That gives you a convenient way to concatenate Arrays:

const x = ['a', 'b'];
const y = ['c'];
const z = ['d', 'e'];

const arr = [...x, ...y, ...z]; // ['a', 'b', 'c', 'd', 'e']
One advantage of the spread operator is that its operand can be any iterable value (concat() does not support iteration).

11.8.3.1 Converting iterable or Array-like objects to Arrays
The spread operator lets you convert any iterable value to an Array:

const arr = [...someIterableObject];
Let’s convert a Set to an Array:

const set = new Set([11, -1, 6]);
const arr = [...set]; // [11, -1, 6]
Your own iterable objects can be converted to Arrays in the same manner:

const obj = {
    * [Symbol.iterator]() {
        yield 'a';
        yield 'b';
        yield 'c';
    }
};
const arr = [...obj]; // ['a', 'b', 'c']
Note that, just like the for-of loop, the spread operator only works for iterable values. Most important objects are iterable: Arrays, Maps, Sets and arguments. Most DOM data structures will also eventually be iterable.

Should you ever encounter something that is not iterable, but Array-like (indexed elements plus a property length), you can use Array.from()2 to convert it to an Array:

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
Iterables are explained in another chapter.↩
Explained in the chapter on Arrays.↩
