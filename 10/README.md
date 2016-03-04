#해체(Destructuring)

ECMAScript6 에서는 객체나 배열(possibly nested)의 값을 추출하기위한 편리한 방법으로 *해체(destructuring)*를 지원한다. 이번 챕터에서는 해체가 어떻게 동작하는지 설명하고 유용한 예제도 함께 제공한다.

##10.1 개요
해체는 데이터 할당을 받는 곳에서(이를테면 할당 연산자의 좌측), 데이터의 일부를 추출하기 위한 패턴 사용을 가능하게 한다.

##10.1.1 객체 해체(Object destructuring)
객체 해체하기:
```
const obj = { first: 'Jane', last: 'Doe' };
const {first: f, last: l} = obj;
// f = 'Jane'; l = 'Doe'

// {prop} is short for {prop: prop}
const {first, last} = obj;
// first = 'Jane'; last = 'Doe'
```    
    
해체는 반환된 값의 처리를 돕는다 : 
```
const obj = { foo: 123 };

const {writable, configurable} = Object.getOwnPropertyDescriptor(obj, 'foo');

console.log(writable, configurable); // true true
```
##10.1.2 배열해체(Array destructuring)

모든 이터러블은 배열 해체가 가능하다.
```
const iterable = ['a', 'b'];
const [x, y] = iterable;
    // x = 'a'; y = 'b'
```
해체는 반환된 값의 처리를 돕는다 : 

```
const [all, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```

##10.1.3 해체는 어디에서 쓰이는가?
해체는 다음과 같은 곳에서 쓰일 수 있다:
```
// 변수 선언:
const [x] = ['a'];
let [x] = ['a'];
var [x] = ['a'];

// 할당:
[x] = ['a'];

// 매개변수 선언:
`
function f([x]) { ··· }
f(['a']);
```
해체는 for-of 루프에서도 동작한다:

```
const arr1 = ['a', 'b'];
for (const [index, element] of arr1.entries()) {
    console.log(index, element);
}
// 결과:
// 0 a
// 1 b

const arr2 = [
    {name: 'Jane', age: 41},
    {name: 'John', age: 40},
];

for (const {name, age} of arr2) {
    console.log(name, age);
}
// 결과:
// Jane 41
// John 40
```

##10.2 배경지식: 데이터 생성 vs 데이터 추출

해체가 무엇인지 완벽히 이해하기 위해서 먼저 broader context를 알아보자. 자바스크립트는 데이터 생성을 위한 operations을 가진다 :
```
const obj = {};
obj.first = 'Jane';
obj.last = 'Doe';
```
그리고 데이터 추출을 위한 operations도 가진다:
```
const f = obj.first;
const l = obj.last;
```
우리가 constructing에 사용해오던 똑같은 문법임을 주목하라.

constructing 을 위한 더 나은 문법인 객체 리터럴 :
```
const obj = { first: 'Jane', last: 'Doe' };
```
ECMAScript6의 해체는 데이터 추출을 위한 동일한 문법을 가능케한다. 
Destructuring in ECMAScript 6 enables the same syntax for extracting data, where it is called an object pattern:
```
const { first: f, last: l } = obj;
```
객체 리터럴을 통해 여러개의 프로퍼티를 한 번에 만들 수 있듯이, 객체 해체 패턴도 여러래의 프로퍼티를 한 번에 추출 할 수 있다.  


이러한 패턴들을 통하여 배열을 해체하는 것도 가능하다 :
```
const [x, y] = ['a', 'b']; // x = 'a'; y = 'b'
```

##10.3 패턴(Patterns)


The following two parties are involved in destructuring:

Destructuring source: the data to be destructured. For example, the right-hand side of a destructuring assignment.

Destructuring target: the pattern used for destructuring. For example, the left-hand side of a destructuring assignment.
The destructuring target is either one of three patterns:

Assignment target. For example: x

변수선언과 매개변수 정의에서는, 오직 변수에 대한 참조만 허용된다. 해체 할당에서 여러가지 옵션이 있지만 나중에 설명하겠다.
객체 패턴. 예: { first: «pattern», last: «pattern» }
객체 패턴의 parts는 프로퍼티이고 이 프로퍼티 값은 다시 패턴이다(재귀적으로).
배열 배턴. 예: [ «pattern», «pattern» ]
배열패턴의 parts는 요소이고 이 요소들은 다시 패턴이다(재귀적으로).

That means that you can nest patterns, arbitrarily deeply:
```
const obj = { a: [{ foo: 123, bar: 'abc' }, {}], b: true };
const { a: [{foo: f}] } = obj; // f = 123
```

##10.3.1 Pick what you need
객체를 해체한다면 오직 프로퍼티만이 관심사이다 :
```
const { x: x } = { x: 7, y: 3 }; // x = 7
```
배열을 해체한다면 prefix추출만을 택할 수 있다.
```
const [x,y] = ['a', 'b', 'c']; // x='a'; y='b';
```
##10.4 패턴이 값의 내부에 접근하는 방법?
In an assignment pattern = someValue, how does the pattern access what’s inside someValue?


##10.4.1 객체 패턴은 객체에 값을 강제한다.
객체 패턴은 프로퍼티에 접근하기 전에 오브젝트에 소스 해체를 강제한다. 그 말인 즉슨 it works with primitive values:
```
const {length : len} = 'abc'; // len = 3
const {toString: s} = 123; // s = Number.prototype.toString
```
##10.4.1.1 Failing to object-destructure a value
The coercion to object is not performed via Object(), but via the internal operation ToObject(). Object() never fails:
```
> typeof Object('abc')
'object'
> var obj = {};
> Object(obj) === obj
true
> Object(undefined)
{}
> Object(null)
{}
```
ToObject() throws a TypeError if it encounters undefined or null. Therefore, the following destructurings fail, even before destructuring accesses any properties:
```
const { prop: x } = undefined; // TypeError
const { prop: y } = null; // TypeError
```
위 결과로써, 값이 객체에 강제되는지 여부를 알기위해 빈 객체 패턴{}을 사용 할 수 있음을 알 수 있다.
As a consequence, you can use the empty object pattern {} to check whether a value is coercible to an object. As we have seen, only undefined and null aren’t:
```
({} = [true, false]); // OK, 배열은 객체에 coercible 하다.
({} = 'abc'); // OK, 문자열은 객체에 coercible 하다.

({} = undefined); // TypeError
({} = null); // TypeError
```
자바스크립트에서 문(statements)은 중괄호로 시작되면 안되기 때문에 표현식을 둘러싸고 있는 소괄호가 필요하다.
The parentheses around the expressions are necessary because statements must not begin with curly braces in JavaScript.

##10.4.2 Array patterns work with iterables
배열해체는 소스의 요소(Elements)를 얻기위해 이터레이터를 사용한다. 그러므로 어떤 값이 이터러블이라면 배열해체가 가능하다.
이터러블 값의 예제를 보자. 

문자열은 이터러블이다 : 
```
const [x,...y] = 'abc'; // x='a'; y=['b', 'c']
```

이터레이터는 code points (“Unicode characters”, 21 bits)를 반환하는 것이지 code units(“JavaScript characters”, 16 bits)를 반환하는게 아님을 유념하라. (자세한 내용은 “Chapter 24. Unicode and JavaScript” in “Speaking JavaScript”. 를 참고하라. 예를 들어 :
```
const [x,y,z] = 'a\uD83D\uDCA9c'; // x='a'; y='\uD83D\uDCA9'; z='c'
```

인덱스로 Set 의 요소에 접근 할 수는 없지만, 이터레이터를 이용하면 접근이 가능하다. 그러므로 Set도 배열해체가 가능하다. :
```
const [x,y] = new Set(['a', 'b']); // x='a'; y='b’;
```
Set 이터레이터는 요소가 삽입된 그 위치에서 순서대로 요소(elements)를 리턴한다. 이것이 이전의 해체 결과가 항상 같은 이유이다.

무한수열. 해체는 무한 수열에도 동작한다. 제너레이터 allNaturalNumbers() 는 0,1,2.... 을 yield하는 이터레이터를 반환한다.

```
function* allNaturalNumbers() {
  for (let n = 0; ; n++) {
    yield n;
  }
}
```

무한 수열에서 처음 3개의 요소를 추출하는 해체 코드이다.
```
const [x, y, z] = allNaturalNumbers(); // x=0; y=1; z=2
```

##10.4.2.1 값 배열해체의 실패 Failing to Array-destructure a value
객체를 반환하는 Symbol.iterator가 key인 메소드를 가지고 있다면 이 값은 이터러블이다. 배열해체는 해체하려는 값이 이터러블이 아니면 타입에러를 낸다. :
```
let x;
[x] = [true, false]; // OK, Arrays are iterable
[x] = 'abc'; // OK, strings are iterable
[x] = { * [Symbol.iterator]() { yield 1 } }; // OK, iterable

[x] = {}; // TypeError, empty objecdts are not iterable
[x] = undefined; // TypeError, not iterable
[x] = null; // TypeError, not iterable
```
이 타입에러는 이터러블의 각 요소에 접근하기 이전에도 발생한다. 이 말인 즉슨, 값이 이터러블인지 아닌지 체크하기 위해 빈배열 패턴을 사용 할 수 있다. 
```
[] = {}; // TypeError, empty objects are not iterable
[] = undefined; // TypeError, not iterable
[] = null; // TypeError, not iterable
```

##10.5 매칭되는 부분이 없다면?
Similarly to how JavaScript handles non-existent properties and Array elements, destructuring fails silently if the target mentions a part that doesn’t exist in the source: the interior of the part is matched against undefined. If the interior is a variable that means that the variable is set to undefined:
```
const [x] = []; // x = undefined
const {prop:y} = {}; // y = undefined
```
Remember that object patterns and Array patterns throw a TypeError if they are matched against undefined.

##10.5.1 Default values
Default values are a feature of patterns: If a part (an object property or an Array element) has no match in the source, it is matched against:

its default value (if specified)
undefined (otherwise)
That is, providing a default value is optional.

Let’s look at an example. In the following destructuring, the element at index 0 has no match on the right-hand side. Therefore, destructuring continues by matching x against 3, which leads to x being set to 3.
```
const [x=3, y] = []; // x = 3; y = undefined
```
You can also use default values in object patterns:
```
const {foo: x=3, bar: y} = {}; // x = 3; y = undefined
```
##10.5.1.1 undefined triggers default values
Default values are also used if a part does have a match and that match is undefined:
```
const [x=1] = [undefined]; // x = 1
const {prop: y=2} = {prop: undefined}; // y = 2
```
The rationale for this behavior is explained in the next chapter, in the section on parameter default values.

##10.5.1.2 Default values are computed on demand
The default values themselves are only computed when they are needed. In other words, this destructuring:
```
const {prop: y=someFunc()} = someValue;
```
위 코드는 아래와 같다.
```
let y;
if (someValue.prop === undefined) {
    y = someFunc();
} else {
    y = someValue.prop;
}
```

You can observe that if you use console.log():
```
> function log(x) { console.log(x); return 'YES' }

> const [a=log('hello')] = [];
hello
> a
'YES'

> const [b=log('hello')] = [123];
> b
123
```
In the second destructuring, the default value is not triggered and log() is not called.

##10.5.1.3 Default values can refer to other variables in the pattern
A default value can refer to any variable, including another variable in the same pattern:
```
const [x=3, y=x] = [];     // x=3; y=3
const [x=3, y=x] = [7];    // x=7; y=7
const [x=3, y=x] = [7, 2]; // x=7; y=2
```
However, order matters: the variables x and y are declared from left to right and produce a ReferenceError if they are accessed before their declaration:
```
const [x=y, y=3] = []; // ReferenceError
```

##10.5.1.4 Default values for patterns
So far we have only seen default values for variables, but you can also associate them with patterns:
```
const [{ prop: x } = {}] = [];
```
What does this mean? Recall the rule for default values:

If the part has no match in the source, destructuring continues with the default value […].

The element at index 0 has no match, which is why destructuring continues with:
```
const { prop: x } = {}; // x = undefined
```
You can more easily see why things work this way if you replace the pattern { prop: x } with the variable pattern:
```
const [pattern = {}] = [];
```
More complex default values. Let’s further explore default values for patterns. In the following example, we assign a value to x via the default value { prop: 123 }:
```
const [{ prop: x } = { prop: 123 }] = [];
```

Because the Array element at index 0 has no match on the right-hand side, destructuring continues as follows and x is set to 123.
```
const { prop: x } = { prop: 123 };  // x = 123
```

However, x is not assigned a value in this manner if the right-hand side has an element at index 0, because then the default value isn’t triggered.
```
const [{ prop: x } = { prop: 123 }] = [{}];
```
In this case, destructuring continues with:
```
const { prop: x } = {}; // x = undefined
```
Thus, if you want x to be 123 if either the object or the property is missing, you need to specify a default value for x itself:
```
const [{ prop: x=123 } = {}] = [{}];
```
Here, destructuring continues as follows, independently of whether the right-hand side is [{}] or [].
```
const { prop: x=123 } = {}; // x = 123
```

Still confused?
A later section explains destructuring from a different angle, as an algorithm. That may give you additional insight.

##10.6 More object destructuring features
##10.6.1 Property value shorthands
Property value shorthands are a feature of object literals: If the value of a property is provided via a variable whose name is the same as the key, you can omit the key. This works for destructuring, too:
```
const { x, y } = { x: 11, y: 8 }; // x = 11; y = 8
```

This declaration is equivalent to:

const { x: x, y: y } = { x: 11, y: 8 };
You can also combine property value shorthands with default values:
```
const { x, y = 1 } = {}; // x = undefined; y = 1
```

##10.6.2 Computed property keys
Computed property keys are another object literal feature that also works for destructuring: You can specify the key of a property via an expression, if you put it in square brackets:
```
const FOO = 'foo';
const { [FOO]: f } = { foo: 123 }; // f = 123
```
Computed property keys allow you to destructure properties whose keys are symbols:

// Create and destructure a property whose key is a symbol
const KEY = Symbol();
const obj = { [KEY]: 'abc' };
const { [KEY]: x } = obj; // x = 'abc'

// Extract Array.prototype[Symbol.iterator]
const { [Symbol.iterator]: func } = [];
console.log(typeof func); // function
##10.7 More Array destructuring features
##10.7.1 Elision
Elision lets you use the syntax of Array “holes” to skip elements during destructuring:
```
const [,, x, y] = ['a', 'b', 'c', 'd']; // x = 'c'; y = 'd'
```

##10.7.2 Rest operator (...)
The rest operator lets you extract the remaining elements of an Array into an Array. You can only use the operator as the last part inside an Array pattern:
```
const [x, ...y] = ['a', 'b', 'c']; // x='a'; y=['b', 'c']
```
The rest operator operator extracts data. The same syntax (...) is used by the spread operator, which contributes data to Array literals and function calls and is explained in the next chapter.

If the operator can’t find any elements, it matches its operand against the empty Array. That is, it never produces undefined or null. For example:
```
const [x, y, ...z] = ['a']; // x='a'; y=undefined; z=[]
```
The operand of the rest operator doesn’t have to be a variable, you can use patterns, too:
```
const [x, ...[y, z]] = ['a', 'b', 'c'];
    // x = 'a'; y = 'b'; z = 'c'
```

The rest operator triggers the following destructuring:

[y, z] = ['b', 'c']
The spread operator (...) looks exactly like the rest operator, but it is used inside function calls and Array literals (not inside destructuring patterns).

10.8 You can assign to more than just variables
If you assign via destructuring, each assignment target can be everything that is allowed on the left-hand side of a normal assignment, including a reference to a property (obj.prop) and a reference to an Array element (arr[0]).
```
const obj = {};
const arr = [];

({ foo: obj.prop, bar: arr[0] }) = { foo: 123, bar: true };

console.log(obj); // {prop:123}
console.log(arr); // [true]
```

You can also assign to object properties and Array elements via the rest operator (...):
```
const obj = {};
[first, ...obj.prop] = ['a', 'b', 'c'];
    // first = 'a'; obj.prop = ['b', 'c']
```
If you declare variables or define parameters via destructuring then you must use simple identifiers, you can’t refer to object properties and Array elements.

##10.9 해체의 Pitfalls
해체를 사용할 때 2가지 유념해야할 사항이 있다.

중괄호로 선언문을 시작하면 안된다.
You can’t start a statement with a curly brace.

During destructuring, you can either declare variables or assign to them, but not both.
The next two sections have the details.

##10.9.1 중괄호로 선언문을 시작하지 말아라
Because code blocks begin with a curly brace, statements must not begin with one. This is unfortunate when using object destructuring in an assignment:

{ a, b } = someObject; // SyntaxError
The work-around is to put the complete expression in parentheses:

({ a, b } = someObject); // ok
##10.9.2 You can’t mix declaring and assigning to existing variables
Within a destructuring variable declaration, every variable in the source is declared. In the following example, we are trying to declare the variable b and refer to the existing variable f, which doesn’t work.

let f;
···
let { foo: f, bar: b } = someObject;
    // During parsing (before running the code):
    // SyntaxError: Duplicate declaration, f
The fix is to use a destructuring assignment and to declare b beforehand:

let f;
···
let b;
({ foo: f, bar: b }) = someObject;
##10.10 Examples of destructuring
Let’s start with a few smaller examples.

The for-of loop supports destructuring:

const map = new Map().set(false, 'no').set(true, 'yes');
for (const [key, value] of map) {
  console.log(key + ' is ' + value);
}
You can use destructuring to swap values. That is something that engines could optimize, so that no Array would be created.

[a, b] = [b, a];
You can use destructuring to split an Array:

const [first, ...rest] = ['a', 'b', 'c'];
    // first = 'a'; rest = ['b', 'c']
##10.10.1 Destructuring returned Arrays
Some built-in JavaScript operations return Arrays. Destructuring helps with processing them:

const [all, year, month, day] =
    /^(\d\d\d\d)-(\d\d)-(\d\d)$/
    .exec('2999-12-31');
If you are only interested in the groups (and not in the complete match, all), you can use elision to skip the array element at index 0:
```
const [, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```
exec() returns null if the regular expression doesn’t match. Unfortunately, you can’t handle null via default values, which is why you must use the Or operator (||) in this case:
```
const [, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec(someStr) || [];
```
Array.prototype.split() returns an Array. Therefore, destructuring is useful if you are interested in the elements, not the Array:
```
const cells = 'Jane\tDoe\tCTO'
const [firstName, lastName, title] = cells.split('\t');
console.log(firstName, lastName, title);
```

##10.10.2 Destructuring returned objects
Destructuring is also useful for extracting data from objects that are returned by functions or methods. For example, the iterator method next() returns an object with two properties, done and value. The following code logs all elements of Array arr via the iterator iter. Destructuring is used in line A.
```
const arr = ['a', 'b'];
const iter = arr[Symbol.iterator]();
while (true) {
    const {done,value} = iter.next(); // (A)
    if (done) break;
    console.log(value);
}
```

##10.10.3 Array-destructuring iterable values
Array-destructuring works with any iterable value. That is occasionally useful:
```
const [x,y] = new Set().add('a').add('b');
    // x = 'a'; y = 'b'

const [a,b] = 'foo';
    // a = 'f'; b = 'o'
```

##10.10.4 Multiple return values
To see the usefulness of multiple return values, let’s implement a function findElement(a, p) that searches for the first element in the Array a for which the function p returns true. The question is: what should that function return? Sometimes one is interested in the element itself, sometimes in its index, sometimes in both. The following implementation returns both.
```
function findElement(array, predicate) {
    for (const [index, element] of array.entries()) { // (A)
        if (predicate(element)) {
            return { element, index }; // (B)
        }
    }
    return { element: undefined, index: -1 };
}
```

In line A, the Array method entries() returns an iterable over [index,element] pairs. We destructure one pair per iteration. In line B, we use property value shorthands to return the object { element: element, index: index }.

findElement()를 사용해봅시다. In the following example, several ECMAScript 6 features allow us to write more concise code: The callback is an arrow function, the return value is destructured via an object pattern with property value shorthands.
```
const arr = [7, 8, 6];
const {element, index} = findElement(arr, x => x % 2 === 0);
    // element = 8, index = 1
```
Due to index and element also referring to property keys, the order in which we mention them doesn’t matter:

const {index, element} = findElement(···);
We have successfully handled the case of needing both index and element. What if we are only interested in one of them? It turns out that, thanks to ECMAScript 6, our implementation can take care of that, too. And the syntactic overhead compared to functions with single return values is minimal.
```
const a = [7, 8, 6];

const {element} = findElement(a, x => x % 2 === 0);
    // element = 8

const {index} = findElement(a, x => x % 2 === 0);
    // index = 1
```

Each time, we only extract the value of the one property that we need.

##10.11 The destructuring algorithm
This section looks at destructuring from a different angle: as a recursive pattern matching algorithm.

This different angle should especially help with understanding default values. If you feel you don’t fully understand them yet, read on.

At the end, I’ll use the algorithm to explain the difference between the following two function declarations.
```
function move({x=0, y=0} = {})         { ··· }
function move({x, y} = { x: 0, y: 0 }) { ··· }
```
##10.11.1 The algorithm
A destructuring assignment looks like this:

«pattern» = «value»
We want to use pattern to extract data from value. I’ll now describe an algorithm for doing so, which is known in functional programming as pattern matching (short: matching). The algorithm specifies the operator ← (“match against”) for destructuring assignment that matches a pattern against a value and assigns to variables while doing so:

«pattern» ← «value»
The algorithm is specified via recursive rules that take apart both operands of the ← operator. The declarative notation may take some getting used to, but it makes the specification of the algorithm more concise. Each rule has two parts:

The head specifies which operands are handled by the rule.
The body specifies what to do next.
I only show the algorithm for destructuring assignment. Destructuring variable declarations and destructuring parameter definitions work similarly.

I don’t cover advanced features (computed property keys; property value shorthands; object properties and array elements as assignment targets), either. Only the basics.

##10.11.1.1 Patterns
A pattern is either:

A variable: x
An object pattern: {«properties»}
An Array pattern: [«elements»]
Each of the following sections describes one of these three cases.

##10.11.1.2 Variable
(1) x ← value (including undefined and null)
  x = value
##10.11.1.3 Object pattern
(2a) {«properties»} ← undefined
  throw new TypeError();
(2b) {«properties»} ← null
  throw new TypeError();
(2c) {key: «pattern», «properties»} ← obj
  «pattern» ← obj.key
  {«properties»} ← obj
(2d) {key: «pattern» = default_value, «properties»} ← obj
  const tmp = obj.key;
  if (tmp !== undefined) {
      «pattern» ← tmp
  } else {
      «pattern» ← default_value
  }
  {«properties»} ← obj
(2e) {} ← obj
  // No properties left, nothing to do
##10.11.1.4 Array pattern
Array pattern and iterable. The algorithm for Array destructuring starts with an Array pattern and an iterable:

(3a) [«elements»] ← non_iterable
assert(!isIterable(non_iterable))
  throw new TypeError();
(3b) [«elements»] ← iterable
assert(isIterable(iterable))
  const iterator = iterable[Symbol.iterator]();
  «elements» ← iterator
Helper function:

function isIterable(value) {
    return (value !== null
        && typeof value === 'object'
        && typeof value[Symbol.iterator] === 'function');
}
Array elements and iterator. The algorithm continues with the elements of the pattern (left-hand side of the arrow) and the iterator that was obtained from the iterable (right-hand side of the arrow).

(3c) «pattern», «elements» ← iterator
  «pattern» ← getNext(iterator) // undefined after last item
  «elements» ← iterator
(3d) «pattern» = default_value, «elements» ← iterator
  const tmp = getNext(iterator);  // undefined after last item
  if (tmp !== undefined) {
      «pattern» ← tmp
  } else {
      «pattern» ← default_value
  }
  «elements» ← iterator
(3e) , «elements» ← iterator (hole, elision)
  getNext(iterator); // skip
  «elements» ← iterator
(3f) ...«pattern» ← iterator (always last part!)
  const tmp = [];
  for (const elem of iterator) {
      tmp.push(elem);
  }
  «pattern» ← tmp
(3g) ← iterator
  // No elements left, nothing to do
Helper function:

function getNext(iterator) {
    const {done,value} = iterator.next();
    return (done ? undefined : value);
}
##10.11.2 알고리즘 적용
The following function definition has named parameters, a technique that is sometimes called options object and explained in the chapter on parameter handling. The parameters use destructuring and default values in such a way that x and y can be omitted. But the object with the parameter can be omitted, too, as you can see in the last line of the code below. This feature is enabled via the = {} in the head of the function definition.

function move1({x=0, y=0} = {}) {
    return [x, y];
}
move1({x: 3, y: 8}); // [3, 8]
move1({x: 3}); // [3, 0]
move1({}); // [0, 0]
move1(); // [0, 0]
But why would you define the parameters as in the previous code snippet? Why not as follows – which is also completely legal ES6 code?

function move2({x, y} = { x: 0, y: 0 }) {
    return [x, y];
}
To see why move1() is correct, let’s use both functions for two examples. Before we do that, let’s see how the passing of parameters can be explained via matching.

##10.11.2.1 Background: passing parameters via matching
For function calls, formal parameters (inside function definitions) are matched against actual parameters (inside function calls). As an example, take the following function definition and the following function call.

function func(a=0, b=0) { ··· }
func(1, 2);
The parameters a and b are set up similarly to the following destructuring.

[a=0, b=0] ← [1, 2]
10.11.2.2 Using move2()
Let’s examine how destructuring works for move2().

Example 1. move2() leads to this destructuring:

[{x, y} = { x: 0, y: 0 }] ← []
The only Array element on the left-hand side does not have a match on the right-hand side, which is why {x,y} is matched against the default value and not against data from the right-hand side (rules 3b, 3d):

{x, y} ← { x: 0, y: 0 }
The left-hand side contains property value shorthands, it is an abbreviation for:

{x: x, y: y} ← { x: 0, y: 0 }
This destructuring leads to the following two assignments (rule 2c, 1):

x = 0;
y = 0;
However, this is the only case in which the default value is used.

Example 2. Let’s examine the function call move2({z:3}) which leads to the following destructuring:

[{x, y} = { x: 0, y: 0 }] ← [{z:3}]
There is an Array element at index 0 on the right-hand side. Therefore, the default value is ignored and the next step is (rule 3d):

{x, y} ← { z: 3 }
That leads to both x and y being set to undefined, which is not what we want.

10.11.2.3 Using move1()
Let’s try move1().

Example 1: move1()

[{x=0, y=0} = {}] ← []
We don’t have an Array element at index 0 on the right-hand side and use the default value (rule 3d):

{x=0, y=0} ← {}
The left-hand side contains property value shorthands, which means that this destructuring is equivalent to:

{x: x=0, y: y=0} ← {}
Neither property x nor property y have a match on the right-hand side. Therefore, the default values are used and the following destructurings are performed next (rule 2d):

x ← 0
y ← 0
That leads to the following assignments (rule 1):

x = 0
y = 0
Example 2: move1({z:3})

[{x=0, y=0} = {}] ← [{z:3}]
The first element of the Array pattern has a match on the right-hand side and that match is used to continue destructuring (rule 3d):

{x=0, y=0} ← {z:3}
Like in example 1, there are no properties x and y on the right-hand side and the default values are used:

x = 0
y = 0
10.11.3 Conclusion
The examples demonstrate that default values are a feature of pattern parts (object properties or Array elements). If a part has no match or is matched against undefined then the default value is used. That is, the pattern is matched against the default value, instead.
