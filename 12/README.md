# 12. ECMAScript6 호출 할수 있는 개체들
이 챕터는 너가가 ES6안에서 호출 할 수 있는 개체(function calls, method calls, 등)를 어떻게 적절하게 사용하는 방법에 충고를 준다.

이 챕터의 섹션: 
* An overview of callable entities in ES6
* Ways of calling in ES6
* Recommendations for using callable entities
* ES6 callable entities in detail
* Dispatched and direct method calls in ES5 and ES6
* The name property of functions
* FAQ: callable entities

## 12.1 ES6안에서 호출 할 수 있는 개체의 개요
ES5에서는, 싱글 생성자, 전통적이 함수의 세가지 역할:
진짜(메소드가 아닌) 함수 
메소드
생성자

ES6안에서 더 특화되었다. 이 세개의 의무들은 아래처럼 처리되어있다. (클래스 정의는 클래스 생성을 위한 두개의 생성자중 하나 입니다.- 클래스 선언 또는 클래스 표현식)

* 진짜(메스드가 아닌) 함수:
  * 애로우 함수 (오직 표현식형태만 갖는다.)
  * 전통적인 함수 (함수 표현식이나 함수 선언식을 통해 생성된다.)
  * 제너레이터 함수 (제너레이터 함수 표현식이나 제너레이터 함수 선언식을 통해 생성된다.)
* 메소드:
  * 메소드들 (오브젝트 안에서 메소드 정의나 클래스 정의에 의해 생성)
  * 제너레이터 메소드 (오브젝트 리터럴안에 메소드 정의나 클래스 정의에 의해 생성)
* 생성자:
  * 클래스 (클래스 정의를 통해 생성)

나는 아래와 같이 구분합니다.
* 개체: 예 전통적인 함수
* 개체를 생성하는 신택스: 예 함수표현식, 함수 선언

비록 그들의 동작이 다르나(뒤에 설명하겠음), 모든 개체 함수다. 예를 들면:
```
> typeof (() => {}) // arrow function
'function'
> typeof function* () {} // generator function
'function'
> typeof class {} // class
'function'
```
## 12.2 ES6에서 호출 방법
어떤 호출은 아무때나 발생된다. 그 외는 제한된 특정 지역에서만 호출 된다.
### 12.2.1 아무때나 발생되는 호출
ES6안에의 아무때나 호출 되는 3가지 종류:
* 함수 호출: func(3, 1)
* 메서드 호출: obj.method('abc')
* 생성자 호출: new Constr(8)
모든 함수 호출들, 대부분의 ES6 코드는 모듈에 포함되고, 그 모듈 몸통은 암시적으로 stric mode이라는 것을 기억하는것이 중요하다.

### 12.2.2 super에 의한 호출은 특별한 지역으로 제한되어 있다.
호출의 두가지 종류는 super키워드를 통해 만들어 진다; 그것들의 사용은 특정지역으로 제한된다.
*Super-메소드 호출: super.method('abc')
오직 객체 리터럴 이나 파생 클래스 정의 안에서의 메소드 정의에서만 가능하다.
*Super-생성자 호출: super(8)
오직 파생 클래스 정의 안의 특별 메소드인 constructor()에서만 가능하다.

### 12.2.3 비 메소드 함수 대 메소드
미 메소드 함수와 메소드의 차이는 ECMAScript 6 안에서 더욱더 명백하다. 지금 둘다 그들만의 할 수 있는 특별한 개체가 있다.
* 애로우 함수는 비 메소드 함수를 만든다. 이것은 this(과 다른 변수들)을 둘러쌓인 스코프(어휘적 this)를 통해 가져온다. 
* 메소드 정의는 메소드를 만든다. 이것은 super, 부모 프로퍼티의 인용 그리고 부모 메소드 호출을 지원을 제공합니다.

## 12.3 호출 개체 사용을 위한 추천
이 섹션은 호출 개체 사용에 대한 팁을 준다: 언제 어느 개체 사용이 최선인가에 대하여: 등.

### 12.3.1 콜백으로 애로우 함수를 선호
콜백으로써, 애로우 함수는 전통적인 함수에 비해 두 가지 이점을 갖는다.:
* this는 어휘적그러므로 사용하기 안전하다.
* 이 신택스는 더 간결하다. 많은 고차 함수나 메서드가 사용된  함수형 프로그래밍에서 특히 중요하다. (파라메터가 함수인 함수와 메서드)

여러줄에 걸친 콜백함수때문에 나 역시 전통적인 함수 표현식을 수용하는 것을 발견할 수 있다. 그러나 너는 this를 조심해야 한다.

#### 12.3.1.1 문제: 함축적인 파라미터로써 this
아아, 어떤 자바스크립트 API는 this를 콜백 함수의 함축적인 인자로 사용된다. 그것은 너가 애로우 함수를 사용하는것을 막는다. 예를 들면: 이 B라인의 this는 A라인 안의 함수의 함축적 인자 이다.

```
beforeEach(function () { // (A)
    this.addMatchers({ // (B)
        toBeInRange: function (start, end) {  
            ···
        }  
    });  
});
```
이 패턴은 명백하지 않고, 애로우 함수 사용을 막는다.

#### 12.3.1.2 해결 1: API 변경
이것은 고치기 쉬운 방법이나 API변경이 요구 된다:

```
beforeEach(api => {
    api.addMatchers({
        toBeInRange(start, end) {
            ···
        }
    });
});
```
우리는 API를 this의 함축적 파라미터에서 명백한 파라미더 API로 변경했다. 나는 이런 종류의 명확성을 좋아한다.

#### 12.3.1.3 해결 2: this를 다른 방법으로 접근
어떤 API는 this를 얻는 다른 방법이 있다. 아래코드에서 사용된 this를 예를 들면

```
var $button = $('#myButton');
$button.on('click', function () {
    this.classList.toggle('clicked');
});
```
그러나 이벤트의 target은 event.target을 통해 접근 할 수 있어야 한다:

```
var $button = $('#myButton');
$button.on('click', event => {
    event.target.classList.toggle('clicked');
});
```
12.3.2 Prefer function declarations as stand-alone functions
As stand-alone functions (versus callbacks), I prefer function declarations:

function foo(arg1, arg2) {
    ···
}
The benefits are:

Subjectively, I find they look nicer. In this case, the verbose keyword function is an advantage – you want the construct to stand out.
They look like generator function declarations, leading to more visual consistency of the code.
There is one caveat: Normally, you don’t need this in stand-alone functions. If you use it, you want to access the this of the surrounding scope (e.g. a method which contains the stand-alone function). Alas, function declarations don’t let you do that – they have their own this, which shadows the this of the surrounding scope. Therefore, you may want to let a linter warn you about this in function declarations.

Another option for stand-alone functions is assigning arrow functions to variables. Problems with this are avoided, because it is lexical.

const foo = (arg1, arg2) => {
    ···
};
12.3.3 Prefer method definitions for methods
Method definitions are the only way to create methods that use super. They are the obvious choice in object literals and classes (where they are the only way to define methods), but what about adding a method to an existing object? For example:

MyClass.prototype.foo = function (arg1, arg2) {
    ···
};
The following is a quick way to do the same thing in ES6 (caveat: Object.assing() doesn’t move methods with super properly).

Object.assign(MyClass.prototype, {
    foo(arg1, arg2) {
        ···
    }
});
For more information and caveats, consult the section on Object.assign().

12.3.4 Methods versus callbacks
There is a subtle difference between an object with methods and an object with callbacks.

12.3.4.1 An object whose properties are methods
The this of a method is the receiver of the method call (e.g. obj if the method call is obj.m(···)).

For example, you can use the WHATWG streams API as follows:

const surroundingObject = {
    surroundingMethod() {
        const obj = {
            data: 'abc',
            start(controller) {
                ···
                console.log(this.data); // abc (*)
                this.pull(); // (**)
                ···
            },
            pull() {
                ···
            },
            cancel() {
                ···
            },
        };
        const stream = new ReadableStream(obj);
    },
};
That is, obj is an object whose properties start, pull and cancel are methods. Accordingly, these methods can use this to access object-local state (line *) and to call each other (line **).

12.3.4.2 An object whose properties are callbacks
The this of an arrow function is the this of the surrounding scope (lexical this). Arrow functions make great callbacks, because that is the behavior you normally want for callbacks (real, non-method functions). A callback shouldn’t have its own this that shadows the this of the surrounding scope.

If the properties start, pull and cancel are arrow functions then they pick up the this of surroundingMethod() (their surrounding scope):

const surroundingObject = {
    surroundingData: 'xyz',
    surroundingMethod() {
        const obj = {
            start: controller => {
                ···
                console.log(this.surroundingData); // xyz (*)
                ···
            },

            pull: () => {
                ···
            },

            cancel: () => {
                ···
            },
        };
        const stream = new ReadableStream(obj);
    },
};
const stream = new ReadableStream();
If the output in line * surprises you then consider the following code:

const obj = {
    foo: 123,
    bar() {
        const f = () => console.log(this.foo); // 123
        const o = {
            p: () => console.log(this.foo), // 123
        };
    },
}
Inside method bar(), f and o.p work the same, because both arrow functions have the same surrounding lexical scope, bar(). The latter arrow function being surrounded by an object literal does not change that.

12.3.5 Avoid IIFEs in ES6
This section gives tips for avoiding IIFEs in ES6.

12.3.5.1 Replace an IIFE with a block
In ES5, you had to use an IIFE if you wanted to keep a variable local:

(function () {  // open IIFE
    var tmp = ···;
    ···
}());  // close IIFE

console.log(tmp); // ReferenceError
In ECMAScript 6, you can simply use a block and a let or const declaration:

{  // open block
    let tmp = ···;
    ···
}  // close block

console.log(tmp); // ReferenceError
12.3.5.2 Replace an IIFE with a module
In ECMAScript 5 code that doesn’t use modules via libraries (such as RequireJS, browserify or webpack), the revealing module pattern is popular, and based on an IIFE. Its advantage is that it clearly separates between what is public and what is private:

var my_module = (function () {
    // Module-private variable:
    var countInvocations = 0;

    function myFunc(x) {
        countInvocations++;
        ···
    }

    // Exported by module:
    return {
        myFunc: myFunc
    };
}());
This module pattern produces a global variable and is used as follows:

my_module.myFunc(33);
In ECMAScript 6, modules are built in, which is why the barrier to adopting them is low:

// my_module.js

// Module-private variable:
let countInvocations = 0;

export function myFunc(x) {
    countInvocations++;
    ···
}
This module does not produce a global variable and is used as follows:

import { myFunc } from 'my_module.js';

myFunc(33);
12.3.5.3 Immediately-invoked arrow functions
There is one use case where you still need an immediately-invoked function in ES6: Sometimes you only can produce a result via a sequence of statements, not via a single expression. If you want to inline those statements, you have to immediately invoke a function. In ES6, you can use immediately-invoked arrow functions if you want to:

const SENTENCE = 'How are you?';
const REVERSED_SENTENCE = (() => {
    // Iteration over the string gives us code points
    // (better for reversal than characters)
    const arr = [...SENTENCE];
    arr.reverse();
    return arr.join('');
})();
Note that you must parenthesize as shown (the parens are around the arrow function, not around the complete function call). Details are explained in the chapter on arrow functions.

12.3.6 Use classes as constructors
In ES5, constructor functions where the mainstream way of creating factories for objects (but there were also many other techniques, some arguably more elegant). In ES6, classes are the mainstream way of implementing constructor functions. Several frameworks support them as alternatives to their custom inheritance APIs.

12.4 ES6 callable entities in detail
This section starts with a cheat sheet, before describing each ES6 callable entity in detail.

12.4.1 Cheat sheet: callable entities
12.4.1.1 The behavior and structure of callable entities
Value:

 	Func decl/Func expr	Arrow	Class	Method
Function-callable	✔	✔	×	✔
Constructor-callable	✔	×	✔	×
Prototype	F.p	F.p	SC	F.p
Property prototype	✔	×	✔	×
Whole construct:

 	Func decl	Func expr	Arrow	Class	Method
Hoisted	✔	 	 	×	 
Creates window prop. (1)	✔	 	 	×	 
Inner name (2)	×	✔	 	✔	×
Body:

 	Func decl	Func expr	Arrow	Class (3)	Method
this	✔	✔	lex	✔	✔
new.target	✔	✔	lex	✔	✔
super.prop	×	×	lex	✔	✔
super()	×	×	×	✔	×
Abbreviations in cells:

✔ exists, allowed
× does not exist, not allowed
Empty cell: not applicable, not relevant
lex: lexical, inherited from surrounding lexical scope
F.p: Function.prototype
SC: superclass for derived classes, Function.prototype for base classes. The details are explained in the chapter on classes.
Notes:

(1) The rules for what declarations create properties for the global object are explained in the chapter on variables and scoping.
(2) The inner names of named function expressions and classes are explained in the chapter on classes.
(3) This column is about the body of the class constructor.
What about generator functions and methods? Those work like their non-generator counterparts, with two exceptions:

Generator functions and methods have the prototype (GeneratorFunction).prototype ((GeneratorFunction) is an internal object, see diagram in Sect. “Inheritance within the iteration API (including generators)”).
You can’t constructor-call generator functions.
12.4.1.2 The rules for this
 	FC strict	FC sloppy	MC	new
Traditional function	undefined	window	receiver	instance
Generator function	undefined	window	receiver	TypeError
Method	undefined	window	receiver	TypeError
Generator method	undefined	window	receiver	TypeError
Arrow function	lexical	lexical	lexical	TypeError
Class	TypeError	TypeError	TypeError	SC protocol
Abbreviations in column titles:

FC: function call
MC: method call
Abbreviations in cells:

lexical: inherited from surrounding lexical scope
SC protocol: subclassing protocol (new instance in base class, received from superclass in derived class)
12.4.2 Traditional functions
These are the functions that you know from ES5. There are two ways to create them:

Function expression:
  const foo = function (x) { ··· };
Function declaration:
  function foo(x) { ··· }
Rules for this:

Function calls: this is undefined in strict mode and the global object in sloppy mode.
Method calls: this is the receiver of the method call (or the first argument of call/apply).
Constructor calls: this is the newly created instance.
12.4.3 Generator functions
Generator functions are explained in the chapter on generators. Their syntax is similar to traditional functions, but they have an extra asterisk:

Generator function expression:
  const foo = function* (x) { ··· };
Generator function declaration:
  function* foo(x) { ··· }
The rules for this are as follows. Note that it never refers to the generator object.

Function/method calls: this is handled like it is with traditional functions. The results of such calls are generator objects.
Constructor calls: You can’t constructor-call generator functions. A TypeError is thrown if you do.
12.4.4 Method definitions
Method definitions can appear inside object literals:

const obj = {
    add(x, y) {
        return x + y;
    }, // comma is required
    sub(x, y) {
        return x - y;
    }, // comma is optional
};
And inside class definitions:

class AddSub {
    add(x, y) {
        return x + y;
    } // no comma
    sub(x, y) {
        return x - y;
    } // no comma
}
As you can see, you must separate method definitions in an object literal with commas, but there are no separators between them in a class definition. The former is necessary to keep the syntax consistent, especially with regard to getters and setters.

Method definitions are the only place where you can use super to refer to super-properties. Only method definitions that use super produce functions that have the property [[HomeObject]], which is required for that feature (details are explained in the chapter on classes).

Rules:

Function calls: If you extract a method and call it as a function, it behaves like a traditional function.
Method calls: work as with traditional functions, but additionally allow you to use super.
Constructor calls: produce a TypeError.
Inside class definitions, methods whose name is constructor are special, as explained later.

12.4.5 Generator method definitions
Generator methods are explained in the chapter on generators. Their syntax is similar to method definitions, but they have an extra asterisk:

const obj = {
    * generatorMethod(···) {
        ···
    },
};
class MyClass {
    * generatorMethod(···) {
        ···
    }
}
Rules:

Calling a generator method returns a generator object.
You can use this and super as you would in normal method definitions.
12.4.6 Arrow functions
Arrow functions are explained in their own chapter:

const squares = [1,2,3].map(x => x * x);
The following variables are lexical inside an arrow function (picked up from the surrounding scope):

arguments
super
this
new.target
Rules:

Function calls: lexical this etc.
Method calls: You can use arrow functions as methods, but their this continues to be lexical and does not refer to the receiver of a method call.
Constructor calls: produce a TypeError.
12.4.7 Classes
Classes are explained in their own chapter.

// Base class: no `extends`
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `(${this.x}, ${this.y})`;
    }
}

// This class is derived from `Point`
class ColorPoint extends Point {
    constructor(x, y, color) {
        super(x, y);
        this.color = color;
    }
    toString() {
        return super.toString() + ' in ' + this.color;
    }
}
The Method constructor is special, because it “becomes” the class. That is, classes are very similar to constructor functions:

> Point.prototype.constructor === Point
true
Rules:

Function/method calls: Classes can’t be called as functions or methods (why is explained in the chapter on classes).
Constructor calls: follow a protocol that supports subclassing. In a base class, an instance is created and this refers to it. A derived class receives its instance from its superclass, which is why it needs to call super before it can access this.
12.5 Dispatched and direct method calls in ES5 and ES6
There are two ways to call methods in JavaScript:

Via dispatch, e.g. obj.someMethod(arg0, arg1)
Directly, e.g. someFunc.call(thisValue, arg0, arg1)
This section explains how these two work and why you will rarely call methods directly in ECMAScript 6. Before we get started, I’ll refresh your knowledge w.r.t. to prototype chains.

12.5.1 Background: prototype chains
Remember that each object in JavaScript is actually a chain of one or more objects. The first object inherits properties from the later objects. For example, the prototype chain of an Array ['a', 'b'] looks as follows:

The instance, holding the elements 'a' and 'b'
Array.prototype, the properties provided by the Array constructor
Object.prototype, the properties provided by the Object constructor
null (the end of the chain, so not really a member of it)
You can examine the chain via Object.getPrototypeOf():

> var arr = ['a', 'b'];
> var p = Object.getPrototypeOf;

> p(arr) === Array.prototype
true
> p(p(arr)) === Object.prototype
true
> p(p(p(arr)))
null
Properties in “earlier” objects override properties in “later” objects. For example, Array.prototype provides an Array-specific version of the toString() method, overriding Object.prototype.toString().

> var arr = ['a', 'b'];
> Object.getOwnPropertyNames(Array.prototype)
[ 'toString', 'join', 'pop', ··· ]
> arr.toString()
'a,b'
12.5.2 Dispatched method calls
If you look at the method call arr.toString() you can see that it actually performs two steps:

Dispatch: In the prototype chain of arr, retrieve the value of the first property whose name is toString.
Call: Call the value and set the implicit parameter this to the receiver arr of the method invocation.
You can make the two steps explicit by using the call() method of functions:

> var func = arr.toString; // dispatch
> func.call(arr) // direct call, providing a value for `this`
'a,b'
12.5.3 Direct method calls
There are two ways to make direct method calls in JavaScript:

Function.prototype.call(thisValue, arg0?, arg1?, ···)
Function.prototype.apply(thisValue, argArray)
Both method call and method apply are invoked on functions. They are different from normal function calls in that you specify a value for this. call provides the arguments of the method call via individual parameters, apply provides them via an Array.

With a dispatched method call, the receiver plays two roles: It is used to find the method and it is an implicit parameter. A problem with the first role is that a method must be in the prototype chain of an object if you want to invoke it. With a direct method call, the method can come from anywhere. That allows you to borrow a method from another object. For example, you can borrow Object.prototype.toString and thus apply the original, un-overridden implementation of toString to an Array arr:

> const arr = ['a','b','c'];
> Object.prototype.toString.call(arr)
'[object Array]'
The Array version of toString() produces a different result:

> arr.toString() // dispatched
'a,b,c'
> Array.prototype.toString.call(arr); // direct
'a,b,c'
Methods that work with a variety of objects (not just with instances of “their” constructor) are called generic. Speaking JavaScript has a list of all methods that are generic. The list includes most Array methods and all methods of Object.prototype (which have to work with all objects and are thus implicitly generic).

12.5.4 Use cases for direct method calls
This section covers use cases for direct method calls. Each time, I’ll first describe the use case in ES5 and then how it changes with ES6 (where you’ll rarely need direct method calls).

12.5.4.1 ES5: Provide parameters to a method via an Array
Some functions accept multiple values, but only one value per parameter. What if you want to pass the values via an Array?

For example, push() lets you destructively append several values to an Array:

> var arr = ['a', 'b'];
> arr.push('c', 'd')
4
> arr
[ 'a', 'b', 'c', 'd' ]
But you can’t destructively append a whole Array. You can work around that limitation by using apply():

> var arr = ['a', 'b'];
> Array.prototype.push.apply(arr, ['c', 'd'])
4
> arr
[ 'a', 'b', 'c', 'd' ]
Similarly, Math.max() and Math.min() only work for single values:

> Math.max(-1, 7, 2)
7
With apply(), you can use them for Arrays:

> Math.max.apply(null, [-1, 7, 2])
7
12.5.4.2 ES6: The spread operator (...) mostly replaces apply()
Making a direct method call via apply() only because you want to turn an Array into arguments is clumsy, which is why ECMAScript 6 has the spread operator (...) for this. It provides this functionality even in dispatched method calls.

> Math.max(...[-1, 7, 2])
7
Another example:

> const arr = ['a', 'b'];
> arr.push(...['c', 'd'])
4
> arr
[ 'a', 'b', 'c', 'd' ]
As a bonus, spread also works with the new operator:

> new Date(...[2011, 11, 24])
Sat Dec 24 2011 00:00:00 GMT+0100 (CET)
Note that apply() can’t be used with new – the above feat can only be achieved via a complicated work-around in ECMAScript 5.

12.5.4.3 ES5: Convert an Array-like object to an Array
Some objects in JavaScript are Array-like, they are almost Arrays, but don’t have any of the Array methods. Let’s look at two examples.

First, the special variable arguments of functions is Array-like. It has a length and indexed access to elements.

> var args = function () { return arguments }('a', 'b');
> args.length
2
> args[0]
'a'
But arguments isn’t an instance of Array and does not have the method forEach().

> args instanceof Array
false
> args.forEach
undefined
Second, the DOM method document.querySelectorAll() returns an instance of NodeList.

> document.querySelectorAll('a[href]') instanceof NodeList
true
> document.querySelectorAll('a[href]').forEach // no Array methods!
undefined
Thus, for many complex operations, you need to convert Array-like objects to Arrays first. That is achieved via Array.prototype.slice(). This method copies the elements of its receiver into a new Array:

> var arr = ['a', 'b'];
> arr.slice()
[ 'a', 'b' ]
> arr.slice() === arr
false
If you call slice() directly, you can convert a NodeList to an Array:

var domLinks = document.querySelectorAll('a[href]');
var links = Array.prototype.slice.call(domLinks);
links.forEach(function (link) {
    console.log(link);
});
And you can convert arguments to an Array:

function format(pattern) {
    // params start at arguments[1], skipping `pattern`
    var params = Array.prototype.slice.call(arguments, 1);
    return params;
}
console.log(format('a', 'b', 'c')); // ['b', 'c']
12.5.4.4 ES6: Array-like objects are less burdensome
On one hand, ECMAScript 6 has Array.from(), a simpler way of converting Array-like objects to Arrays:

const domLinks = document.querySelectorAll('a[href]');
const links = Array.from(domLinks);
links.forEach(function (link) {
    console.log(link);
});
On the other hand, you won’t need the Array-like arguments, because ECMAScript 6 has rest parameters (declared via a triple dot):

function format(pattern, ...params) {
    return params;
}
console.log(format('a', 'b', 'c')); // ['b', 'c']
12.5.4.5 ES5: Using hasOwnProperty() safely
obj.hasOwnProperty('prop') tells you whether obj has the own (non-inherited) property prop.

> var obj = { prop: 123 };

> obj.hasOwnProperty('prop')
true

> 'toString' in obj // inherited
true
> obj.hasOwnProperty('toString') // own
false
However, calling hasOwnProperty via dispatch can cease to work properly if Object.prototype.hasOwnProperty is overridden.

> var obj1 = { hasOwnProperty: 123 };
> obj1.hasOwnProperty('toString')
TypeError: Property 'hasOwnProperty' is not a function
hasOwnProperty may also be unavailable via dispatch if Object.prototype is not in the prototype chain of an object.

> var obj2 = Object.create(null);
> obj2.hasOwnProperty('toString')
TypeError: Object has no method 'hasOwnProperty'
In both cases, the solution is to make a direct call to hasOwnProperty:

> var obj1 = { hasOwnProperty: 123 };
> Object.prototype.hasOwnProperty.call(obj1, 'hasOwnProperty')
true

> var obj2 = Object.create(null);
> Object.prototype.hasOwnProperty.call(obj2, 'toString')
false
12.5.4.6 ES6: Less need for hasOwnProperty()
hasOwnProperty() is mostly used to implement Maps via objects. Thankfully, ECMAScript 6 has a built-in Map data structure, which means that you’ll need hasOwnProperty() less.

12.5.4.7 ES5: Avoiding intermediate objects
Applying an Array method such as join() to a string normally involves two steps:

var str = 'abc';
var arr = str.split(''); // step 1
var joined = arr.join('-'); // step 2
console.log(joined); // a-b-c
Strings are Array-like and can become the this value of generic Array methods. Therefore, a direct call lets you avoid step 1:

var str = 'abc';
var joined = Array.prototype.join.call(str, '-');
Similarly, you can apply map() to a string either after you split it or via a direct method call:

> function toUpper(x) { return x.toUpperCase() }
> 'abc'.split('').map(toUpper)
[ 'A', 'B', 'C' ]

> Array.prototype.map.call('abc', toUpper)
[ 'A', 'B', 'C' ]
Note that the direct calls may be more efficient, but they are also much less elegant. Be sure that they are really worth it!

12.5.4.8 ES6: Avoiding intermediate objects
Array.from() can convert and map in a single step, if you provide it with a callback as the second argument.

> Array.from('abc', ch => ch.toUpperCase())
[ 'A', 'B', 'C' ]
As a reminder, the two step solution is:

> 'abc'.split('').map(function (x) { return x.toUpperCase() })
[ 'A', 'B', 'C' ]
12.5.5 Abbreviations for Object.prototype and Array.prototype
You can access the methods of Object.prototype via an empty object literal (whose prototype is Object.prototype). For example, the following two direct method calls are equivalent:

Object.prototype.hasOwnProperty.call(obj, 'propKey')
{}.hasOwnProperty.call(obj, 'propKey')
The same trick works for Array.prototype:

Array.prototype.slice.call(arguments)
[].slice.call(arguments)
This pattern has become quite popular. It does not reflect the intention of the author as clearly as the longer version, but it’s much less verbose. Speed-wise, there isn’t much of a difference between the two versions.

12.6 The name property of functions
The name property of a function contains its name:

> function foo() {}
> foo.name
'foo'
This property is useful for debugging (its value shows up in stack traces) and some metaprogramming tasks (picking a function by name etc.).

Prior to ECMAScript 6, this property was already supported by most engines. With ES6, it becomes part of the language standard and is frequently filled in automatically.

12.6.1 Constructs that provide names for functions
The following sections describe how name is set up automatically for various programming constructs.

12.6.1.1 Variable declarations and assignments
Functions pick up names if they are created via variable declarations:

let func1 = function () {};
console.log(func1.name); // func1

const func2 = function () {};
console.log(func2.name); // func2

var func3 = function () {};
console.log(func3.name); // func3
But even with a normal assignment, name is set up properly:

let func4;
func4 = function () {};
console.log(func4.name); // func4

var func5;
func5 = function () {};
console.log(func5.name); // func5
With regard to names, arrow functions are like anonymous function expressions:

const func = () => {};
console.log(func.name); // func
From now on, whenever you see an anonymous function expression, you can assume that an arrow function works the same way.

12.6.1.2 Default values
If a function is a default value, it gets its name from its variable or parameter:

let [func1 = function () {}] = [];
console.log(func1.name); // func1

let { f2: func2 = function () {} } = {};
console.log(func2.name); // func2

function g(func3 = function () {}) {
    return func3.name;
}
console.log(g()); // func3
12.6.1.3 Named function definitions
Function declarations and function expression are function definitions. This scenario has been supported for a long time: a function definition with a name passes it on to the name property.

For example, a function declaration:

function foo() {}
console.log(foo.name); // foo
The name of a named function expression also sets up the name property.

const bar = function baz() {};
console.log(bar.name); // baz
Because it comes first, the function expression’s name baz takes precedence over other names (e.g. the name bar provided via the variable declaration):

However, as in ES5, the name of a function expression is only a variable inside the function expression:

const bar = function baz() {
    console.log(baz.name); // baz
};
bar();
console.log(baz); // ReferenceError
12.6.1.4 Methods in object literals
If a function is the value of a property, it gets its name from that property. It doesn’t matter if that happens via a method definition (line A), a traditional property definition (line B), a property definition with a computed property key (line C) or a property value shorthand (line D).

function func() {}
let obj = {
    m1() {}, // (A)
    m2: function () {}, // (B)
    ['m' + '3']: function () {}, // (C)
    func, // (D)
};
console.log(obj.m1.name); // m1
console.log(obj.m2.name); // m2
console.log(obj.m3.name); // m3
console.log(obj.func.name); // func
The names of getters are prefixed with 'get', the names of setters are prefixed with 'set':

let obj = {
    get foo() {},
    set bar(value) {},
};
let getter = Object.getOwnPropertyDescriptor(obj, 'foo').get;
console.log(getter.name); // 'get foo'

let setter = Object.getOwnPropertyDescriptor(obj, 'bar').set;
console.log(setter.name); // 'set bar'
12.6.1.5 Methods in class definitions
The naming of methods in class definitions is similar to object literals:

class C {
    m1() {}
    ['m' + '2']() {} // computed property key

    static classMethod() {}
}
console.log(C.prototype.m1.name); // m1
console.log(new C().m1.name); // m1

console.log(C.prototype.m2.name); // m2

console.log(C.classMethod.name); // classMethod
Getters and setters again have the name prefixes 'get' and 'set', respectively:

class C {
    get foo() {}
    set bar(value) {}
}
let getter = Object.getOwnPropertyDescriptor(C.prototype, 'foo').get;
console.log(getter.name); // 'get foo'

let setter = Object.getOwnPropertyDescriptor(C.prototype, 'bar').set;
console.log(setter.name); // 'set bar'
12.6.1.6 Methods whose keys are symbols
In ES6, the key of a method can be a symbol. The name property of such a method is still a string:

If the symbol has a description, the method’s name is the description in square brackets.
Otherwise, the method’s name is the empty string ('').
const key1 = Symbol('description');
const key2 = Symbol();

let obj = {
    [key1]() {},
    [key2]() {},
};
console.log(obj[key1].name); // '[description]'
console.log(obj[key2].name); // ''
12.6.1.7 Class definitions
Remember that class definitions create functions. Those functions also have their property name set up correctly:

class Foo {}
console.log(Foo.name); // Foo

const Bar = class {};
console.log(Bar.name); // Bar
12.6.1.8 Default exports
All of the following statements set name to 'default':

export default function () {}
export default (function () {});

export default class {}
export default (class {});

export default () => {};
12.6.1.9 Other programming constructs
Generator functions and generator methods get their names the same way that normal functions and methods do.
new Function() produces functions whose name is 'anonymous'. A webkit bug describes why that is necessary on the web.
func.bind(···) produces a function whose name is 'bound '+func.name:
  function foo(x) {
      return x
  }
  const bound = foo.bind(undefined, 123);
  console.log(bound.name); // 'bound foo'
12.6.2 Caveats
12.6.2.1 Caveat: the name of a function is always assigned at creation
Function names are always assigned during creation and never changed later on. That is, JavaScript engines detect the previously mentioned patterns and create functions that start their lives with the correct names. The following code demonstrates that the name of the function created by functionFactory() is assigned in line A and not changed by the declaration in line B.

function functionFactory() {
    return function () {}; // (A)
}
const foo = functionFactory(); // (B)
console.log(foo.name.length); // 0 (anonymous)
One could, in theory, check for each assignment whether the right-hand side evaluates to a function and whether that function doesn’t have a name, yet. But that would incur a significant performance penalty.

12.6.2.2 Caveat: minification
Function names are subject to minification, which means that they will usually change in minified code. Depending on what you want to do, you may have to manage function names via strings (which are not minified) or you may have to tell your minifier what names not to minify.

12.6.3 Changing the names of functions
These are the attributes of property name:

> let func = function () {}
> Object.getOwnPropertyDescriptor(func, 'name')
{ value: 'func',
  writable: false,
  enumerable: false,
  configurable: true }
The property not being writable means that you can’t change its value via assignment:

> func.name = 'foo';
> func.name
'func'
The property is, however, configurable, which means that you can change it by re-defining it:

> Object.defineProperty(func, 'name', {value: 'foo', configurable: true});
> func.name
'foo'
If the property name already exists then you can omit the descriptor property configurable, because missing descriptor properties mean that the corresponding attributes are not changed.

If the property name does not exist yet then the descriptor property configurable ensures that name remains configurable (the default attribute values are all false or undefined).

12.6.4 The function property name in the spec
The spec operation SetFunctionName() sets up the property name. Search for its name in the spec to find out where that happens.
The third parameter of that operations specifies a name prefix. It is used for:
Getters and setters (prefixes 'get' and 'set')
Function.prototype.bind() (prefix 'bound')
Anonymous function expressions not having a property name can be seen by looking at their runtime semantics:
The names of named function expressions are set up via SetFunctionName(). That operation is not invoked for anonymous function expressions.
The names of function declarations are set up when entering a scope (they are hoisted!).
When an arrow function is created, no name is set up, either (SetFunctionName() is not invoked).
12.7 FAQ: callable entities
12.7.1 Why are there “fat” arrow functions (=>) in ES6, but no “thin” arrow functions (->)?
ECMAScript 6 has syntax for functions with a lexical this, so-called arrow functions. However, it does not have arrow syntax for functions with dynamic this. That omission was deliberate; method definitions cover most of the use cases for thin arrows. If you really need dynamic this, you can still use a traditional function expression.

12.7.2 How do I determine whether a function was invoked via new?
ES6 has a new protocol for subclassing, which is explained in the chapter on classes. Part of that protocol is the meta-property new.target, which refers to the first element in a chain of constructor calls (similar to this in a chain for supermethod calls). It is undefined if there is no constructor call. We can use that to enforce that a function must be invoked via new or that it must not be invoked via it. This is an example for the latter:

function realFunction() {
    if (new.target !== undefined) {
        throw new Error('Can’t be invoked via `new`');
    }
    ···
}
In ES5, this was usually checked like this:

function realFunction() {
    "use strict";
    if (this !== undefined) {
        throw new Error('Can’t be invoked via `new`');
    }
    ···
}
