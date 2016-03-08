#해체(Destructuring)

ECMAScript6 에서는 객체나 배열(possibly nested)의 값을 추출하기위한 편리한 방법으로 *해체(destructuring)*를 지원한다. 이번 챕터에서는 해체가 어떻게 동작하는지 설명하고 유용한 예제도 함께 제공한다.

##10.1 개요
해체는 데이터 할당을 받는 곳에서(이를테면 할당 연산자의 좌변), 데이터의 일부를 추출하기 위한 패턴 사용을 가능하게 한다.

##10.1.1 객체 해체(Object destructuring)
객체 해체하기:
```javascript
const obj = { first: 'Jane', last: 'Doe' };
const {first: f, last: l} = obj;
// f = 'Jane'; l = 'Doe'

// {prop} is short for {prop: prop}
const {first, last} = obj;
// first = 'Jane'; last = 'Doe'
```    
    
해체는 반환된 값의 처리를 돕는다 : 
```javascript
const obj = { foo: 123 };

const {writable, configurable} = Object.getOwnPropertyDescriptor(obj, 'foo');

console.log(writable, configurable); // true true
```
##10.1.2 배열해체(Array destructuring)

모든 이터러블은 배열 해체가 가능하다.
```javascript
const iterable = ['a', 'b'];
const [x, y] = iterable;
    // x = 'a'; y = 'b'
```
해체는 반환된 값의 처리를 돕는다 : 

```javascript
const [all, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```

##10.1.3 해체는 어디에서 쓰이는가?
해체는 다음과 같은 곳에서 쓰일 수 있다:
```javascript
// 변수 선언:
const [x] = ['a'];
let [x] = ['a'];
var [x] = ['a'];

// 할당:
[x] = ['a'];

// 매개변수 선언:

function f([x]) { ··· }
f(['a']);
```
해체는 for-of 루프에서도 동작한다:

```javascript
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
```javascript
const obj = {};
obj.first = 'Jane';
obj.last = 'Doe';
```
그리고 데이터 추출을 위한 operations도 가진다:
```javascript
const f = obj.first;
const l = obj.last;
```
우리가 객체 생성에 사용해오던 똑같은 문법임을 주목하라.

객체생성을 위한 더 나은 문법인 객체 리터럴 :
```javascript
const obj = { first: 'Jane', last: 'Doe' };
```

ECMAScript6의 해체는 데이터 추출을 위해 객체 패턴으로 불리는 곳에서 동일한 문법을 가능케한다.:
```javascript
const { first: f, last: l } = obj;
```
객체 리터럴을 통해 여러개의 프로퍼티를 한 번에 만들 수 있듯이, 객체 해체 패턴도 여러래의 프로퍼티를 한 번에 추출 할 수 있다.  


이러한 패턴들을 통하여 배열을 해체하는 것도 가능하다 :
```javascript
const [x, y] = ['a', 'b']; // x = 'a'; y = 'b'
```

##10.3 패턴(Patterns)

다음은 해체에 관련된 용어를 설명한다.:

Destructuring source: 해체될 데이터. 예를 들면, 해체 할당식의 우변.

Destructuring target: 해체를 위해 사용되는 패턴. 해체 할당식의 좌변.

해체 대상은 다음중 하나이다.:

할당 대상. 예 : 없음.
변수선언과 매개변수 정의에서는, 오직 변수에 대한 참조만 허용된다. 해체 할당에서 여러가지 옵션이 있지만 나중에 설명하겠다.

객체 패턴. 예: { first: «pattern», last: «pattern» }
객체 패턴의 parts는 프로퍼티이고 이 프로퍼티 값은 다시 패턴이다(재귀적으로).

배열 배턴. 예: [ «pattern», «pattern» ]
배열패턴의 parts는 요소이고 이 요소들은 다시 패턴이다(재귀적으로).

말인즉슨, 얼마든지 깊게 패턴을 응용할 수 있다 :
```javascript
const obj = { a: [{ foo: 123, bar: 'abc' }, {}], b: true };
const { a: [{foo: f}] } = obj; // f = 123
```

##10.3.1 관심사 선택
객체를 해체한다면 오직 프로퍼티만이 관심사이다 :
```javascript
const { x: x } = { x: 7, y: 3 }; // x = 7
```
배열을 해체한다면 prefix추출만을 택할 수 있다.
```javascript
const [x,y] = ['a', 'b', 'c']; // x='a'; y='b';
```
##10.4 패턴이 값의 내부에 접근하는 방법?
할당패턴 = 값, 이러한 패턴이 어떻게 값에 접근 할 수 있을까?


##10.4.1 객체 패턴은 값을 객체로 강제한다.
객체 패턴은 프로퍼티에 접근하기 전에 오브젝트로 소스 해체를 강제한다. 그 말인 즉슨 이것은 primitive 값에도 동작한다.
```javascript
const {length : len} = 'abc'; // len = 3
const {toString: s} = 123; // s = Number.prototype.toString
```
##10.4.1.1 객체 해체의 실패 
Object() 를 통한 객체변환은 수행되지 않지만, 내부 연산자인  ToObject() 는 가능하다. Object()는 절대 실패하지 않는다.:
```javascript
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
ToObject undefined나 null에 대응 할 경우에는 타입에러를 낸다. 그러므로 해체가 프로퍼티에 접근하기 이전이라고 하더라도 다음의 해체는 실패한다. :
```javascript
const { prop: x } = undefined; // TypeError
const { prop: y } = null; // TypeError
```
위 결과로써, 값이 객체에 강제되는지 여부를 알기위해 빈 객체 패턴{}을 사용 할 수 있음을 알 수 있다. 우리가 봤듯이 오직 undeifned와 null만 그렇지 않다.:
```javascript
({} = [true, false]); // OK, 배열은 객체에 coercible 하다.
({} = 'abc'); // OK, 문자열은 객체에 coercible 하다.

({} = undefined); // TypeError
({} = null); // TypeError
```javascript
자바스크립트에서 문(statements)은 중괄호로 시작되면 안되기 때문에 표현식을 둘러싸고 있는 소괄호가 필요하다.

##10.4.2 이터러블과 함께 동작하는 배열 패턴
배열해체는 소스의 요소(Elements)를 얻기위해 이터레이터를 사용한다. 그러므로 어떤 값이 이터러블이라면 배열해체가 가능하다.
이터러블 값의 예제를 보자. 

문자열은 이터러블이다 : 
```javascript
const [x,...y] = 'abc'; // x='a'; y=['b', 'c']
```

이터레이터는 code points (“Unicode characters”, 21 bits)를 반환하는 것이지 code units(“JavaScript characters”, 16 bits)를 반환하는게 아님을 유념하라. (자세한 내용은 “Chapter 24. Unicode and JavaScript” in “Speaking JavaScript”. 를 참고하라. 예를 들어 :
```javascript
const [x,y,z] = 'a\uD83D\uDCA9c'; // x='a'; y='\uD83D\uDCA9'; z='c'
```

인덱스로 Set 의 요소에 접근 할 수는 없지만, 이터레이터를 이용하면 접근이 가능하다. 그러므로 Set도 배열해체가 가능하다. :
```javascript
const [x,y] = new Set(['a', 'b']); // x='a'; y='b’;
```
Set 이터레이터는 요소가 삽입된 그 위치에서 순서대로 요소(elements)를 리턴한다. 이것이 이전의 해체 결과가 항상 같은 이유이다.

무한수열. 해체는 무한 수열에도 동작한다. 제너레이터 allNaturalNumbers() 는 0,1,2.... 을 yield하는 이터레이터를 반환한다.

```javascript
function* allNaturalNumbers() {
  for (let n = 0; ; n++) {
    yield n;
  }
}
```

무한 수열에서 처음 3개의 요소를 추출하는 해체 코드이다.
```javascript
const [x, y, z] = allNaturalNumbers(); // x=0; y=1; z=2
```

##10.4.2.1 값 배열해체의 실패 Failing to Array-destructure a value
객체를 반환하는 Symbol.iterator가 key인 메소드를 가지고 있다면 이 값은 이터러블이다. 배열해체는 해체하려는 값이 이터러블이 아니면 타입에러를 낸다. :
```javascript
let x;
[x] = [true, false]; // OK, Arrays are iterable
[x] = 'abc'; // OK, strings are iterable
[x] = { * [Symbol.iterator]() { yield 1 } }; // OK, iterable

[x] = {}; // TypeError, empty objecdts are not iterable
[x] = undefined; // TypeError, not iterable
[x] = null; // TypeError, not iterable
```
이 타입에러는 이터러블의 각 요소에 접근하기 이전에도 발생한다. 이 말인 즉슨, 값이 이터러블인지 아닌지 체크하기 위해 빈배열 패턴을 사용 할 수 있다. 
```javascript
[] = {}; // TypeError, empty objects are not iterable
[] = undefined; // TypeError, not iterable
[] = null; // TypeError, not iterable
```

##10.5 매칭되는 부분이 없다면?
자바스크립트가 존재하지 않는 프로퍼티나 배열 요소를 다루는 방법과 유사하게, 소스에서 대상이 존재하지 않으면 해체는 조용히 실패한다.: 
이러한 부분의 내부는 undefined와 매칭된다. 내부가 변수라면 해당 변수는 undefined가 세팅된것이다.:
```javascript
const [x] = []; // x = undefined
const {prop:y} = {}; // y = undefined
```

객체 패턴과 배열 패턴이 undefined로 매치되면 타입에러를 내는것을 기억하라 

##10.5.1 기본값

기본값은 패턴의 특징 중에 하나이다.: 객체패턴 또는 배열 패턴에서 매칭되는 부분이 없다면 이것은 다음으로 매칭된다:

기본값.(명시되었다면)
undefined(그렇지 않다면)
따라서 기본값을 제공하는 것은 부가적이다.

예제를 살펴보자. 다음 해체 구문에서, 인덱스가 0인 요소는 우변과 매치되지 않는다. 그러므로, 해체는 x에 3을 매칭하는 것으로 계속 한다.
```javascript
const [x=3, y] = []; // x = 3; y = undefined
```
객체 패턴의 기본값 사용도 가능하다:
```javascript
const {foo: x=3, bar: y} = {}; // x = 3; y = undefined
```
##10.5.1.1 undefined 는 기본값을 발생시킨다.
만약 매칭되는 값이 없거나 undefined 인 경우에는 기본값이 사용된다.:
```javascript
const [x=1] = [undefined]; // x = 1
const {prop: y=2} = {prop: undefined}; // y = 2
```
이런 동작을 위한 합리적인 이유는, 파라미터 기본값 섹션인다음 장에서 설명한다. 

##10.5.1.2 기본값은 필요한 경우에 대입된다.
필요한 경우 기본값은 스스로 연산된다. 다시 말하면 이 해체 구문은 :

```javascript
const {prop: y=someFunc()} = someValue;
```
위 코드는 아래와 같다.

```javascript
let y;
if (someValue.prop === undefined) {
    y = someFunc();
} else {
    y = someValue.prop;
}
```

console.log()를 사용해서 이를 관찰 할 수 있다.
```javascript
> function log(x) { console.log(x); return 'YES' }

> const [a=log('hello')] = [];
hello
> a
'YES'

> const [b=log('hello')] = [123];
> b
123
```

두 번째 해체 구문에서 초기값은 triggered 되지 않고 log() 또한 호출되지 않는다.


##10.5.1.3 기본 값은 패턴안의 다른 변수를 참조 할 수 있다.
기본값은 같은 패턴안의 다른 변수를 포함하여 어떤 변수도 참조가능하다 : 
```javascript
const [x=3, y=x] = [];     // x=3; y=3
const [x=3, y=x] = [7];    // x=7; y=7
const [x=3, y=x] = [7, 2]; // x=7; y=2
```

다른 방법들 : 변수 x와 y 는 좌변에서 우변으로 선언되고 만약 선언되기 이전에 접근 한다면 ReferenceError 를 낸다.:
```javascript
const [x=y, y=3] = []; // ReferenceError
```

##10.5.1.4 패턴의 기본값
우리는 변수의 기본값만을 봐왔지만, 기본값은 패턴에도 연관지을 수 있다 :
```javascript
const [{ prop: x } = {}] = [];
```
이게 무엇을 의미하는가? 기본값 법칙을 다시 보자

소스에서 매칭되는 부분이 없다면 해체는 기본값으로 계속한다.[…]

0번 인덱스의 요소는 매칭되지 않고 이것이 해체가 지속되는 이유이다.:
```javascript
const { prop: x } = {}; // x = undefined
```
 패턴을 { prop: x} 인 변수 패턴으로 바꾼다면 왜 이게 동작하는지 쉽게 알수있다.
 
```javascript
const [pattern = {}] = [];
```
더 복잡한 기본값. 패턴을 위한 기본값을 더 살표보자. 다음 예제에서 우리는 기본값을 이용해 x에 값을 할당한다. { prop: 123 }:
```javascript
const [{ prop: x } = { prop: 123 }] = [];
```
딘덱스 0번의 배열요소는 우변에 매칭되지 않기 때문에 해체는 계속 되고 x는 123으로 세팅된다.

```javascript
const { prop: x } = { prop: 123 };  // x = 123
```
그러나 우변이 0번 인덱스의 요소를 가지고 있다면 x는 이 과정에서 값이 할당되지 않는다. 왜냐하면 기본값이 트리거 되지 않기 때문이다.

```javascript
const [{ prop: x } = { prop: 123 }] = [{}];
```
여기서 해체는 다음과 같이 계속 된다.:
```javascript
const { prop: x } = {}; // x = undefined
```

할당되길 원하면 객체나 속성이 없고 x가 123이 되길 원하면 x의 기본값을 명시해야 한다.:
```javascript
const [{ prop: x=123 } = {}] = [{}];
```
여기 객체는 다음처럼 우변이 [{}] 나 []에 상관없이 독립적으로 지속된다.

```javascript
const { prop: x=123 } = {}; // x = 123
```

아직 혼란스러운가?
나중에 알고리즘으로써의 관점에서 해체를 살펴보겠다. 이것은 또 다른 인사이트를 줄 것이다.

##10.6 더 많은 객체 해체 약칭
##10.6.1 속성값 바로 가기

속성값의 약칭은 객체 리터럴의 특징이다.: 만약 속성값이 이름이 키와 같은 변수를 통해 제공된다면 키를 제외해도 해체는 동작한다.:

```javascript
const { x, y } = { x: 11, y: 8 }; // x = 11; y = 8
```

이 선언은 다음과 비교할만하다.
```javascript

const { x: x, y: y } = { x: 11, y: 8 };
```
또한 기본값과 함께 속성값 약칭을 섞을 수 있다.:
```javascript
const { x, y = 1 } = {}; // x = undefined; y = 1
```

##10.6.2 계산된 속성키
계산된 속성키는 다른 해체를 위해 동작하는 객체 리터럴 특징이다.: 대괄호를 이용한다면 표현식을 통해 속성키를 명시할 수 있다. :
```javascript
const FOO = 'foo';
const { [FOO]: f } = { foo: 123 }; // f = 123
```
계산된 속성키는 심볼키인 프로퍼티의 해체를 허용한다.:
```javascript
// 심볼키를 가지는 속성의 생성과 해체
const KEY = Symbol();
const obj = { [KEY]: 'abc' };
const { [KEY]: x } = obj; // x = 'abc'

// Array.prototype[Symbol.iterator] 추출
const { [Symbol.iterator]: func } = [];
console.log(typeof func); // function
```

##10.7 더 많은 배열 해체의 특징
##10.7.1 생략
해체 구문 사이의 배열의 빈 구멍은 생략할 수 있다.:
```javascript
const [,, x, y] = ['a', 'b', 'c', 'd']; // x = 'c'; y = 'd'
```

##10.7.2 나머지 연산자(rest operator) (...)
나머지 연산자는 배열의 각 요소의 추출한 배열가능케 한다. 배열 패턴의 마지막 파라미터로 나머지 연산자를 사용할 수 있다.:
```javascript
const [x, ...y] = ['a', 'b', 'c']; // x='a'; y=['b', 'c']
```
나머지 연산자는 데이터를 추출한다. 펼침 연산자(...) 의 문법도 똑같다. 펼침 연산자는 데이터를 배열 리터럴과 함수 호출에 공헌한다. 이건 다음 챕터에서 설명한다.

만약 연산자가 요소를 찾지 못한다면, 빈배열로 매칭된다. 때문에 undefined나 null은 절대 할당되지 않는다.

```javascript
const [x, y, ...z] = ['a']; // x='a'; y=undefined; z=[]
```
나머지 연산자의 피연산자가 변수일 필요는 없다. 이 역시 패턴을 사용하면 된다. :
```javascript
const [x, ...[y, z]] = ['a', 'b', 'c'];
    // x = 'a'; y = 'b'; z = 'c'
```
나머지 연산자는 다음의 해체를 발생시킨다:
```javascript
[y, z] = ['b', 'c']
```
펼침 연산자는  (...) 나머지 연산자와 완전히 똑같이 생겼다. 그러나 나머지 연산자는 함수 호출과 배열 리터럴 안에서만 사용된다.(해체 패턴 안이 아니라)

10.8 변수가 아닌 곳에도 할당가능하다.
해체를 통한 할당이라면, 각 할당 타겟은 객체의 프로퍼티(obj.prop) 참조나 배열의 요소(arr[0]) 참조를 포함하여 일반적인 할당문의 좌변에서 허용되는 모든 것이 될 수 있다. 
```javascript
const obj = {};
const arr = [];

({ foo: obj.prop, bar: arr[0] }) = { foo: 123, bar: true };

console.log(obj); // {prop:123}
console.log(arr); // [true]
```

또한 나머지 연산자(...)를 이용해서 객체 프로퍼티와 배열 요소를 할당 할 수도 있다.

```javascript
const obj = {};
[first, ...obj.prop] = ['a', 'b', 'c'];
    // first = 'a'; obj.prop = ['b', 'c']
```
만약 해체를 이용해서 변수를 선언하거나 파라미터를 정의 한다면, simple identifiers를 사용해야한다. 객체 프로퍼티나 배열의 요소를 참조하면 안된다.


##10.9 해체의 Pitfalls
해체를 사용할 때 2가지 유념해야할 사항이 있다.

중괄호로 선언문을 시작하면 안된다.
You can’t start a statement with a curly brace.

해체문에는 변수 선언이나 변수 할당이 가능하다. 둘 다 동시에 하는 것은 불가능하다.
다음 두 섹션에 자세한 내용이 있다.

##10.9.1 중괄호로 선언문을 시작하지 말아라
코드 블럭이 중괄호로 시작하기 때문에, 선언문은 그렇게 하면 안된다. 이것은 할당문에서 객체 해체를 사용할 때 불편하다 :
```javascript
{ a, b } = someObject; // SyntaxError
```
감싸는 괄호안에 완전한 표현식을 써넣어야 한다:

({ a, b } = someObject); // ok

##10.9.2 이미 존재하는 변수에 선언과 할당을 조합하면 안된다.
변수선언을 해체하는 곳에서, 소스의 모든 변수는 선언된다. 다음 예제를 보자. 변수 b를 선언하고 존재하는 변수 f를 참조하지만 동작하지 않는다.
```javascript
let f;
···
let { foo: f, bar: b } = someObject;
    // During parsing (before running the code):
    // SyntaxError: Duplicate declaration, f
```
b를 선언하기 위해 해체 할당을 사용하고 있다.:

```javascript
let f;
···
let b;
({ foo: f, bar: b }) = someObject;
```

##10.10 해체의 예제
몇가지 간단한 예제로 시작해보자.

for-of 반복문은 해체를 지원한다:
```javascript
const map = new Map().set(false, 'no').set(true, 'yes');
for (const [key, value] of map) {
  console.log(key + ' is ' + value);
}
```
값을 바꾸기 위해 해체를 사용 할 수 있다. 엔진이 최적화를 해줄수 있으므로 배열이 생성되지 않는다.

```javascript
[a, b] = [b, a];
```
배열을 스플릿 하기 위해 해체를 사용 할 수 있다
:
```javascript
const [first, ...rest] = ['a', 'b', 'c'];
    // first = 'a'; rest = ['b', 'c']
```
##10.10.1 반환된 배열 해체하기

자바스크립트에 내장된 몇가지 기능은 배열을 반환한다. 해체는 이러한 처리를 돕는다:
```javascript
const [all, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```
그룹(완벽히 일치하지 않는)을 찾고 싶다면, 0번 인덱스에서 배열요소를 건너뛰는것으로 생략 가능하다:
```javascript
const [, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec('2999-12-31');
```
exec() 는 표현식이 매칭되지 않으면 null을 반환한다. 불행히도 null을 기본값으로 다루지는 못한다. 이것은 ||연산자를 사용해야만 하는 이유가 되는 예제이다:

```javascript
const [, year, month, day] = /^(\d\d\d\d)-(\d\d)-(\d\d)$/.exec(someStr) || [];
```

Array.prototype.split()은 배열을 반환한다. 그러므로 해체는 배열이 아닌 각 요소 안에서 유용하다:
```javascript
const cells = 'Jane\tDoe\tCTO'
const [firstName, lastName, title] = cells.split('\t');
console.log(firstName, lastName, title);
```

##10.10.2 반환된 객체 해체하기
해체는 함수나 메소드에 의해 반환된 객체의 데이터를 추출하는데도 유용하다. 예를 들면 이터레이터 메소드인 next()는 done과 value 두가지 속성을 갖는 객체를 반환한다. 다음 코드는 이터레이터를 이용해 배열의 모든 요소 로그를 찍는다. 해체는 line A에서 사용된다.

```javascript
const arr = ['a', 'b'];
const iter = arr[Symbol.iterator]();
while (true) {
    const {done,value} = iter.next(); // (A)
    if (done) break;
    console.log(value);
}
```

##10.10.3 이터러블 값의 배열해체
이터러블 값의 배열해체도 동작 한다. 이것은 때때로 유용하다:

```javascript
const [x,y] = new Set().add('a').add('b');
    // x = 'a'; y = 'b'

const [a,b] = 'foo';
    // a = 'f'; b = 'o'
```

##10.10.4 값을 여러번 반환하기

값을 여러번 반환하는 유용함을 알아보기 위해, findElement(a, p)를 구현해보자. 이 함수는 true를 리턴하는 함수 p의 배열에서 첫 번째 요소를 찾는다. 문제는 그 함수가 무엇을 반환해야 하는가 이다.
```javascript
function findElement(array, predicate) {
    for (const [index, element] of array.entries()) { // (A)
        if (predicate(element)) {
            return { element, index }; // (B)
        }
    }
    return { element: undefined, index: -1 };
}
```

line A에서, 배열 메소드 entries() 는 [index, element] 쌍으로 가진 이터러블을 반환한다. 한번의 이터레이션 마다 한 쌍을 해체한다. line B에서 객체  { element: element, index: index } 를 반환하기 위해 속성값 약칭을 사용한다.

findElement()를 사용해봅시다. 다음 예제에서, 몇몇 ECMAScript 6 특징들은 더 간략한 코딩을 가능하게 해준다.: 콜백은 arrow function 으로, 반환값은 속성값의 약칭인 객체 패턴으로 해체된다
```javascript
const arr = [7, 8, 6];
const {element, index} = findElement(arr, x => x % 2 === 0);
    // element = 8, index = 1
```
속성 키를 참조하는 인덱스와 요소 때문에 우리가 언급한 그 순서는 상관이 없다. 
```javascript
const {index, element} = findElement(···);
```
인덱스와 요소 모두 요구되는 케이스를 성공적으로 다뤘다. 그 것들중에 하나에만 관심이 있었다면 어떨까? 이것은 ECMAScript6에 이르러 해결되었다. 우리의 구현은 이제 그게 가능하다. 그리고 문법 과부하는 하나의 반환값 으로 비교된다.

```javascript
const a = [7, 8, 6];

const {element} = findElement(a, x => x % 2 === 0);
    // element = 8

const {index} = findElement(a, x => x % 2 === 0);
    // index = 1
```

매번 우리가 필요한 속성값을 추출한다 

##10.11 해체 알고리즘
이번 섹션은 다른관점에서 해체를 바라볼 것이다.: 재귀적 패턴 매칭 알고리즘

이 다른 관점은 특히 기본값을 이해하는데 도움을 준다. 만약 완벽히 이해되지 않는다면 계속 읽어봐라

마지막으로 다음 2개의 함수선언문의 차이점을 설명하기 위한 알고리즘을 사용 할 것이다.
```javascript
function move({x=0, y=0} = {})         { ··· }
function move({x, y} = { x: 0, y: 0 }) { ··· }
```
##10.11.1 알고리즘
해체 할당은 다음과 같이 생겼다.

«pattern» = «value»
값으로 부터 데이터 추출을 위한 패턴을 사용을 원한다. 이제부터 그 방법에 대한 알고리즘을 설명 할 것이다. 이것은 패턴 매칭에 의한 함수형 프로그래밍으로 알려져 있다.(이하 매칭). 이 알고리즘은 값의 패턴매칭이 되는 해체 할당을 위해 ← 를 연산자로 규정하고 이러한 과정이 진행되며 변수에 할당한다.

«pattern» ← «value»

이 알고리즘은 좌측을 가르키는 연산자를 받는 재귀적 법칙에 의해 명시된다. 이 선언적인 표기법은 종종 사용되지만 알고리즘의 스펙을 더 간결하게 만든다. 각각의 법칙은 두 부분으로 나뉜다.

head는 룰에 의해 어떤 피연산자부가 다뤄질지 명시한다.
body는 다음에 무엇을 할지 명시한다.

오직 해체 할당을 위한 알고리즘만 보여준다. 변수 선언을 해체하는 것과와 파라미터선언을 해체하는 것은 비슷하게 작동한다.

고급 특징들을 다루지는 않는다. (계산된 속성 키, 속성값, 할당자로써의 객체 속성 또는 배열요소). 기본만 다룬다.


##10.11.1.1 패턴
하나의 패턴은 다음과 같다:

변수: x
객체 패턴: {«properties»}
배열 패턴: [«elements»]
각각의 섹션은 다음중 하나의 상황을 설명한다.

##10.11.1.2 변수
(1) x ← value (including undefined and null)
  x = value
  
##10.11.1.3 객체패턴
(2a) {«properties»} ← undefined  
  throw new TypeError();  
(2b) {«properties»} ← null  
  throw new TypeError();  
(2c) {key: «pattern», «properties»} ← obj  
  «pattern» ← obj.key  
  {«properties»} ← obj  
(2d) {key: «pattern» = default_value, «properties»} ← obj  
  const tmp = obj.key;  
    ```javascript
  if (tmp !== undefined) {
      «pattern» ← tmp
  } else {
      «pattern» ← default_value
  }
  ```
  
  {«properties»} ← obj  
(2e) {} ← obj
  // 좌변의 빈 프로퍼티, 아무것도 안함
  
##10.11.1.4 배열 패턴
배열 패턴과 이터러블. 배열해체를 위한 알고리즘은 배열 패턴과 이터러블로 시작한다.

(3a) [«elements»] ← non_iterable  
assert(!isIterable(non_iterable))  
  throw new TypeError();  
(3b) [«elements»] ← iterable  
assert(isIterable(iterable))  
  const iterator = iterable[Symbol.iterator]();  
  «elements» ← iterator  
헬퍼 함수:  

```javascript
function isIterable(value) {
    return (value !== null
        && typeof value === 'object'
        && typeof value[Symbol.iterator] === 'function');
}
```
배열 요소와 이터레이터. 이 알고리즘은 패턴(화살표의 좌변부)의 요소와 이터러블에서 얻어진 이터레이터(화살표의 우변부)로 함께 지속된다.

(3c) «pattern», «elements» ← iterator  
  «pattern» ← getNext(iterator) // undefined after last item  
  «elements» ← iterator  
(3d) «pattern» = default_value, «elements» ← iterator  
```javascript
  const tmp = getNext(iterator);  // undefined after last item  
  if (tmp !== undefined) {
      «pattern» ← tmp
  } else {
      «pattern» ← default_value
  }
  ```
  «elements» ← iterator  
  
(3e) , «elements» ← iterator (hole, elision)  
  getNext(iterator); // skip  
  «elements» ← iterator  
(3f) ...«pattern» ← iterator (always last part!)  
```javascript
  const tmp = [];
  for (const elem of iterator) {
      tmp.push(elem);
  }
  ```
  «pattern» ← tmp  
  
(3g) ← iterator  
  // No elements left, nothing to do  
헬퍼 함수:  

```javascript
function getNext(iterator) {
    const {done,value} = iterator.next();
    return (done ? undefined : value);
}
```

##10.11.2 알고리즘 적용
다음의 함수선언문은 *이름있는 파라미터(named parameters)*를 가지고 있다. 이 기술은 파라미터 핸들링 챕터에서 때때로 options object 라고 불리거나 설명된다. 이 파라미터는 해체를 사용하고 x와 y가 생략될 수 있는 방법의 기본값을 사용한다. 그러나 아래 코드의 마지막 라인에서 보듯이 객체와 함께있는 파라미터도 생략 가능하다. 이 특징은 함수 선언문의 머리에서  {} 를 통해 사용된다.

```javascript
function move1({x=0, y=0} = {}) {
    return [x, y];
}
move1({x: 3, y: 8}); // [3, 8]  
move1({x: 3}); // [3, 0]  
move1({}); // [0, 0]  
move1(); // [0, 0]  
```
그러나 이전에 왜 코드 스니핏에서 파라미터를 선언해야 하는가? 다음은 완벽히 적합한 ES6 code코드 인가?

```javascript
function move2({x, y} = { x: 0, y: 0 }) {
    return [x, y];
}
```
move1()이 왜 올바른지 알기위해 두 가지 예제를 위한 함수를 사용해보자. 그 전에 우선, 어떻게 파라미터 전달이 매칭에 의해 설명 될수 있는지를 먼저 보자. 

##10.11.2.1 배경: 매칭을 이용한 파라미터 전달
함수 호출을 위해, 일반적인 파라미터(함수 선언 내)들은 실제 파라미터로 매칭된다.(함수 호출 내) 예제와 같이 다음 함수선언과 호출을 하라.
```javascript
function func(a=0, b=0) { ··· }
func(1, 2);
```
파라미터 a와 b는 다음의 해체와 유사한 과정으로 세팅된다.  

[a=0, b=0] ← [1, 2]  

##10.11.2.2 사용하기  move2()  
 move2() 해체가 어떻게 이뤄지는지 보자.  


예제 1. move2()의 해체 과정:  

[{x, y} = { x: 0, y: 0 }] ← []  
오직 좌변부의 배열 요소는 우변과 매치되지 않는다 이것은 {x,y} 가 기본값에 매칭되는 이유이고 우변부의 데이터에 반대된다. (rules 3b, 3d):

{x, y} ← { x: 0, y: 0 }  
좌변은 속성값 약칭을 소유한다 이것은 다음과 같다.

{x: x, y: y} ← { x: 0, y: 0 }  
이 해체는 아래 내용처럼 두 개의 할당과 같은 결과로 이끈다.(rule 2c, 1):  

x = 0;  
y = 0;  

##10.11.3 결론 
예제에서는 기본값이 패턴부(객체의 프로퍼티 또는 배열 요소)의 특징임을 증명합니다. 일치하는 부분이 없거나 undefined 라면 기본값이 사용 됩니다. 말인즉슨, 패턴은 기본값과 일치됩니다.

