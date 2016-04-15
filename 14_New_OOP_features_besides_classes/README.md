14. 클래스를 포함한 새로운 OOP기능

클래스는 es6에 도입된 주요한 새 OOP특징이다. 하지만 오브젝트리터럴과 Objectㅇ 메소드에도 새로운 기능이 포함되어있다.
이 쳅터에서는 이들을 다룬다.

14.1 개요

14.1.1 새 오브젝트리터럴

메소드선언:
```javascript
const obj = {
    myMethod(x, y) {
        ···
    }
};
```
속성값의 간단한 표현:

```javascript
const first = 'Jane';
const last = 'Doe';

const obj = { first, last };
// Same as:
const obj = { first: first, last: last };
```
계산된 속성키:
```javascript
const propKey = 'foo';
const obj = {
    [propKey]: true,
    ['b'+'ar']: 123
};
```
계산된 속성키는 메소드선언에도 사용됨:
```javascript
const obj = {
    ['h'+'ello']() {
        return 'hi';
    }
};
console.log(obj.hello()); // hi
```
계산된 속성키의 주된 활용처는 속성키로 심볼사용을 쉽게 만들어주는 것이다.

14.1.2 Object의 새로운 메소드
Object의 새 메소드중 가장 중요한 것ㅇㄴ assign()이다. 전통적으로 이 기능은 자바스크립트세계에서 extend()라 불렸다.
고전적인 동작과는 대조적으로 Object.assign()은 오직 자신의(상속되지않은)속성만 고려한다.
```javascript
const obj = { foo: 123 };
Object.assign(obj, { bar: true });
console.log(JSON.stringify(obj));
    // {"foo":123,"bar":true}
```
14.2 오브젝트리터럴의 새로운 기능
14.2.1 메소드 정의
es5에서 메소드는 함수값을 갖는 속성이다:
```javascript
var obj = {
    myMethod: function (x, y) {
        ···
    }
};
```
es6에서는 메소드가 여전히 함수를 값으로 갖고 있는 속성이긴 하지만 훨씬 간단한 방법으로 정의할 수 있다:

```javascript
const obj = {
    myMethod(x, y) {
        ···
    }
};
```
es5에서 사용되던 Getter와 Setter도 잘 작동한다.
```javascript
const obj = {
    get foo() {
        console.log('GET foo');
        return 123;
    },
    set bar(value) {
        console.log('SET bar to '+value);
        // return value is ignored
    }
};
```
obj를 써보자:
```javascript
> obj.foo
GET foo
123
> obj.bar = true
SET bar to true
true
```
제네레이터 함수를 값으로 하는 속성을 간단하게 정의하는 방법 또한 제공된다:
```javascript
const obj = {
    * myGeneratorMethod() {
        ···
    }
};
```
이 코드의 의미는 아래와 같다.
```javascript
const obj = {
    myGeneratorMethod: function* () {
        ···
    }
};
```
14.2.2 속성값 단축표현
속성값 단축표현으로 오브젝트 리터럴에서 속성정의를 간단히 할 수 있다: 속성의 값과 키이름이 변수와 둘다 일치하는 경우 키를 생략할 수 있다. 아래와 같이 사용된다.
```javascript
const x = 4;
const y = 1;
const obj = { x, y };
```
마지막줄은 아래와 같다:
```javascript
const obj = { x: x, y: y };
```
속성값 단축표현은 해체와 함께 잘 작동한다:
```javascript
const obj = { x: 4, y: 1 };
const {x,y} = obj;
console.log(x); // 4
console.log(y); // 1
```
속성값 단축표현의 또다른 사용처는 여러개의 값을 반환하는 경우다(해체절에서 설명한다)

14.2.3 계산된 속성 키
속성을 지정할때 두가지 방법이 있다.

고정이름으로: ```obj.foo = true;```
표현식으로: ```obj['b'+'ar'] = 123;```
ES5의 오브젝트 리터럴은 오직 고정이름만 가능하다. ES6에서는 표현식으로도 키이름을 정의할 수 있다:
```javascript
const propKey = 'foo';
const obj = {
    [propKey]: true,
    ['b'+'ar']: 123
};
```
이 새로운 문법은 메소드 정의에도 사용된다:
```javascript
const obj = {
    ['h'+'ello']() {
        return 'hi';
    }
};
console.log(obj.hello()); // hi
```
계산된 속성키의 주 사용처는 심볼이다:공개심볼을 정의하고 항상 유일한 특정 키를 정의하는데 사용할 수 있다. 특히 Symbol.iterator에 저장된 심볼은 참고해볼만한 예다. 어떤 오브젝트가 이 키로 메서드를 갖고 있다면 이터러블(iterable)이 된다:이 메소드는 반드시 이터레이터(iterator)를 반환해야하는데, 이터레이터는 객체를 순회하기 위한 for-of루프 같은 구조에서 사용된다. 아래 코드에 예시가 있다.
```javascript
const obj = {
    * [Symbol.iterator]() { // (A)
        yield 'hello';
        yield 'world';
    }
};
for (const x of obj) {
    console.log(x);
}
// Output:
// hello
// world
```
A줄은 저장된 심볼인 Symbol.iterator을 계산된 키로 정의된 제네레이터메소드의 시작부다.

14.3 Object의 새로운 메소드
14.3.1 Object.assign(target, source_1, source_2, ···)
이 메소드는 소스를 타겟에게 머지한다: 우선 source_1의 모든 열거가능한 본인 소유의 속성을 target에 복사한 뒤, source_2를 복사하는 식으로 모든 인자에 주어진 source를 target에 머지한다. 마지막에는 target을 반환한다.
```javascript
const obj = { foo: 123 };
Object.assign(obj, { bar: true });
console.log(JSON.stringify(obj));
    // {"foo":123,"bar":true}
```
Object.assign()가 어떻게 작동하는지 보다 구체적으로 살펴보자:

두가지 속성키종류: Object.assign()는 문자열과 심볼 두가지다 속성키로 안다.
자신의 열거가능한 속성만: Object.assign()는 상속받은 속성이나 열거불가 속성을 무시한다.
source로부터 읽을 수 있는 값: 보통 “get” 연산 (const value = source[propKey]). 즉 source의 키가 getter라면 실행된 값이 복사될 것이다. Object.assign()으로 생성된 모든 속성은 값속성이다. target에 getter는 전달되지 않는다.
타켓에 쓸 값: 보통l “set” 연산 (target[propKey] = value). 즉 target키가 setter라면 실행된 값이 복사될 것이다.

따라서 target의 getter나 setter의 계산된 결과가 아니라 바르게 getter와 setter을 전달하면서, (열거가능한 것 뿐 아니라) 모든 속성을 복사하는 코드는 아래와 같다:
```javascript
function copyAllProperties(target, ...sources) {
    for (const source of sources) {
        for (const key of Reflect.ownKeys(source)) {
            const desc = Object.getOwnPropertyDescriptor(source, key);
            Object.defineProperty(target, key, desc);
        }
    }
    return target;
}
```
14.3.1.1 경고: Object.assign()은 이동메소드에 대해서는 잘 작동하지 않는다.
한편, super를 사용해 메소드를 이동할 수 없다: 내부속성인 [[HomeObject]]같은 메소드는 생성된 오브젝트에 묶여있다. Object.assign()으로 이를 옮겨도 여전히 원본 객체의 super속성을 참조할 것이다. 자세한 내용은 classes챕터에서 설명한다.

또다른 한편으로 열거가능성은 이동메소드를 클래스의 프로토타입에서 객체 리터럴로 생성했다면 틀리다. 보통 메소드는 모두 열거가능이지만(반대로 어쨌든 Object.assign()는 그들을 볼 수 없다), 프로토타입은 기본적으로는 오직 열거불가 메소드만 갖을 수 있다.

14.3.1.2 Object.assign()의 사용예
몇 가지 사용예를 살펴보자.

14.3.1.2.1 this에 속성을 추가하기
Object.assign() 생성자에서 this에 속성을 추가하기 위해 사용될 수 있다.
```javascript
class Point {
    constructor(x, y) {
        Object.assign(this, {x, y});
    }
}
```
14.3.1.2.2 오브젝트 속성에 기본값을 제공하기
Object.assign()는 잃어버린 속성에 대한 기본값을 채울때도 쓸모가 있다. 다음의 예에서 인자로 받은 options객체 앞서 DEFAULT오브젝트가 속성에 대한 기본값을 제공한다.
```javascript
const DEFAULTS = {
    logLevel: 0,
    outputFormat: 'html'
};
function processContent(options) {
    options = Object.assign({}, DEFAULTS, options); // (A)
    ···
}
```
A줄에서, 새 객체를 만들고 기본값을 복사한뒤 기본값을 덮어쓰며 options를 복사했다. Object.assign()은 이렇게 처리된 결과를 반환한다.

14.3.1.2.3 오브젝트에 메소드를 추가할때
오브젝트에 메소드를 추가할때도 사용할 수 있다:
```javascript
Object.assign(SomeClass.prototype, {
    someMethod(arg1, arg2) {
        ···
    },
    anotherMethod() {
        ···
    }
});
```
직접 함수를 추가할 수 있지만 매번 SomeClass.prototype를 기술해야하고 이를 대체할 마땅한 문법이 없다:
```javascript
SomeClass.prototype.someMethod = function (arg1, arg2) {
    ···
};
SomeClass.prototype.anotherMethod = function () {
    ···
};
```
14.3.1.2.4 오브젝트 클론하기
마지막으로 살펴볼 Object.assign()의 사용처는 오브젝트의 클론을 만드는 빠른 방법이다:
```javascript
function clone(orig) {
    return Object.assign({}, orig);
}
```
이 방법으로 만들어진 클론은 또한 좀 지저분한데 원본의 속성에 대한 기술을 가져오지 않기 때문이다. 이 부분을 원한다면 속성기술자를 사용해야만한다.

원본과 동일한 프로토타입의 클론을 만들고 싶다면 Object.getPrototypeOf()와 Object.create()를 사용해야한다:
```javascript
function clone(orig) {
    const origProto = Object.getPrototypeOf(orig);
    return Object.assign(Object.create(origProto), orig);
}
```
14.3.2 Object.getOwnPropertySymbols(obj)
Object.getOwnPropertySymbols(obj)은 obj의 상속받지 않은 모든 자신소유의 속성인 심볼을 찾는다.  이는 자신의 모든 문자열키를 찾는  Object.getOwnPropertyNames()를 보완한다. 속성키를 순환할 때 더 자세한 내용은 이후 섹션에서 다룬다.

14.3.3 Object.is(value1, value2)
항등비교는 기대했던 것과는 다르게 두개의 값을 다룬다.

일단, NaN은 자신과도 같지 않다.
```javascript
> NaN === NaN
false
```
이로 인해 NaN을 찾을 수 없게 된다:
```javascript
> [0,NaN,2].indexOf(NaN)
-1
```
두번째로, JavaScript는 두개의 0값을 갖고 있다. 하지만 항등비교는 이를 같은 값으로 처리한다:
```javascript
> -0 === +0
true
```
이런 사항들은 일반적인 상황에서는 문제없다.

Object.is()는 ===보다 좀더 정확한 비교를 제공한다. 아래와 같이 작동한다:
```javascript
> Object.is(NaN, NaN)
true
> Object.is(-0, +0)
false
```
그외는 ===와 동일하다.

14.3.3.1 Object.is()를 배열의 요소를 찾는데 사용하기
ES6의 배열 메소드인 findIndex()와 Object.is()를 조합하면 배열에서 NaN을 찾을 수 있다:
```javascript
function myIndexOf(arr, elem) {
    return arr.findIndex(x => Object.is(x, elem));
}

myIndexOf([0,NaN,2], NaN); // 1
```
반대로 indexOf()는 NaN을 잘 다루지 못한다:
```javascript
> [0,NaN,2].indexOf(NaN)
-1
```
14.3.4 Object.setPrototypeOf(obj, proto)
이 메소드는 obj의 프로토타입을 proto로 지정한다. ES5에서 비공식적인 방법으로 많은 엔진에서 지원되었던 방법은 __proto__라는 특별한 속성에 할당하는 것이다. 프로토타입을 지정할때 추천되는 방법은 ES5와 동일하다: 오브젝트를 생성할 때 Object.create()를 이용하는 것이다. 이 쪽이 생성한 뒤 prototype을 지정하는 것보다 항상 더 빠르다. 당연하지만 기존 객체의 프로토타입을 변경하려하면 작동하지 않는다.

14.4 ES6에서 속성키의 순회
ES6에서 속성키는 문자열이나 심볼 둘다 가능하다. 어떤 오브젝트 obj에 대해 속성키를 찾는 메소드는 다섯개나 있다:
```javascript
Object.keys(obj) : Array<string>
```
(상속되지않은)자신의 모든 열거가능한 문자열 키를 찾는다.
```javascript
Object.getOwnPropertyNames(obj) : Array<string>
```
자신의 모든 문자열 속성키를 찾는다.
```javascript
Object.getOwnPropertySymbols(obj) : Array<symbol>
```
자신의 모든 심볼 속성키를 찾는다.
```javascript
Reflect.ownKeys(obj) : Array<string|symbol>
```
자신의 모든 속성키를 찾는다.
```javascript
Reflect.enumerate(obj) : Iterator
```
열거가능한 모든 문자열키를 찾는다.
14.4.1 속성키의 순회시의 정렬
ES6의 다음 연산은 자신의 속성키에 대해 순회한다:

Object.keys()
Object.getOwnPropertyNames()
Reflect.ownKeys()

자신의 속성키들은 다음의 순서로 순회한다:

우선, 정수인덱스(다음섹션에서 설명한다)가 작은수에서 큰수로 정렬된다.
그 뒤에 나은 모든 문자열키는 오브젝트에 추가된 순서로 정렬된다.
마지막으로 모든 심볼키가 오브젝트에 추가된 순서로 정렬된다.
많은 엔진들은 정수형키가 여전히 문자열임(적어도 ES6의 현재 사양까지는)에도 특별하게 취급한다.
그러므로 정수형키는 별도의 카테고리로 취급할 필요가 있다.

14.4.1.1 정수 인덱스
대략, 정수인덱스는 문자열로 53-bit non-negative로 변환된 결과와 같은 값이다. 그러므로 다음과 같다:

'10' 과 '2'는 정수인덱스다.
'02' 는 정수인덱스가 아니다. 이를 정수로 변환해보면 변환된 결과는 문자열 '2'와 다른 값이 된다.
'3.141' 는 정수인덱스가 아니다. 3.141 자체가 정수가 아니기 때문이다.
ES6에서, 문자열과 Typed Arrays는 정수인덱스를 갖는다. 일반 배열에 있는 인덱스는 정수인덱스의 부분집합이다: 일반배열의 인덱스는 32bit의 범위보다 작다. 배열 인덱스에 대한 더 자세한 정보는 자바스크립트를 말하다 책의 배열인덱스 상세편을 보라.

정수인덱스는 53-bit 범위를 가지며, 이는 자바스크립트가 다룰 수 있는 가장 큰 정수의 범위다. 자세한 내용은 "안전한 정수"섹션을 참고한다.

14.4.1.2 예제
아래 코드는 오브젝트가 자신의 키를 순회할대의 순서를 보여준다:
```javascript
const obj = {
    [Symbol('first')]: true,
    '02': true,
    '10': true,
    '01': true,
    '2': true,
    [Symbol('second')]: true,
};
Reflect.ownKeys(obj);
    // [ '2', '10', '02', '01',
    //   Symbol('first'), Symbol('second') ]
```
설명:

'2' 과 '10'는 정수인덱스로 제일 처음 나오고 숫자의 크기로 정렬되어있다(이들은 추가된 순서로 정렬되지 않았다).
'02' 과 '01'는 보통의 문자열키로 취급되며 뒤이어 나오는데 이들은 obj에 추가된 순서대로 나온다.
Symbol('first')와 Symbol('second')는 심볼로 마지막에 오며, obj에 추가된 순서대로 나온다.

14.4.1.3 반환되는 속성키가 정렬되는 것이 왜 표준 사양화 되었을까?
Tab Atkins Jr.의 답변이다:

적어도 오브젝트에 대해 모든 구현체는 대략 같은 순서를 사용한다(현재 스펙에 부합하는), 많은 코드가 실수로 그 순서에 의존적으로 작성되어 있으며 다른 순서가 되면 정지하게 될 것이다. 브라우저들은 웹에서 호환성을 갖추려고 특정 순서를 구현했고, 이에 대한 표준화에 대한 요구사항이 있어 스펙화 되었다.

Map과 Set에서는 이러한 순서를 깨트리자는 몇몇 의견이 있었다. 하지만 그렇게 하면 코드에서 의존할 수 없는 형태의 특별한 순서를 요구받는다. 이는 순서가 특정할 수 없을 뿐 아니라 랜덤으로 주어지도록 위임해야한다. 이는 많은 노력을 요구하므로 생성순서를 사용하는 편이 합리적이다.(예를들어 파이썬의 OrderedDict을 보라) 그러므로 Map과 Set도 Object와 동일하게 결정했다.

14.4.1.4 사양에서에서 속성들의 순서
사양에는 관련된 부분이 두 군데 있다:

Array exotic객체 섹션에서 배열의 인덱스에 대한 노트가 있다.
내부 [[OwnPropertyKeys]] 메소드 섹션에서 반환되는 속성들의 순서를 정의한다.

14.5 할당 대 속성정의
이 섹션은 마지막 섹션에서 필요한 배경지식을 제공한다.

어떤 오브젝트 obj의 속성 prop를 추가하는 비슷한 두가지 방법이 있다:

할당하기: ```obj.prop = 123```
속성정의: ```Object.defineProperty(obj, 'prop', 123)```
할당하기가 아직 존재하지 않아도자신의 속성prop를 생성할 수 없는 3가지 경우가 있다:

1. 읽기전용 속성인 prop가 프로토타입체인에 존재하는 경우라면 할당하기는 스트릭트모드에서 TypeError를 일으킨다.
2. 프로토타입체인에 prop라는 setter가 존재하면 setter가 호출된다.
3. 프로토타입체인에 setter는 없지만 getter로 prop가 존재하면 스트릭트모드에서 TypeError가 일어난다. 이는 첫번째 경우와 비슷하다.

Object.defineProperty()는 자신의 속성을 정의할때 위와 같은 문제를 일으키지 않는다. 다음 섹션에서는 3번째 경우를 보다 자세히 살펴본다.

14.5.1 상속받은 읽기전용 속성을 오버라이딩하기
만약 객체 obj의 읽기전용인 prop속성을 상속받았다면 그 속성에 할당할 수 없다:
```javascript
const proto = Object.defineProperty({}, 'prop', {
    writable: false,
    configurable: true,
    value: 123,
});
const obj = Object.create(proto);
obj.prop = 456;
    // TypeError: Cannot assign to read-only property
```
이는 setter는 없지만 getter만 제공되는 속성을 상속받은 것과 비슷하다. 상속된 속성의 값을 변경하기 위해 할당한다는 관점이 같다.
이는 너무 비파괴적이다(견고하다): 원본은 수정하지 않고 자신의 속성을 새롭게 생성하여 덮어쓴다. 그러므로 상속된 읽기전용속성과 setter없는 속성은 둘다 할당으로부터의 변화되지 않는다. 그러나 속성정의로 자신의 속성을 생성하면 강제할 수 있다:
```javascript
const proto = Object.defineProperty({}, 'prop', {
    writable: false,
    configurable: true,
    value: 123,
});
const obj = Object.create(proto);
Object.defineProperty(obj, 'prop', {value: 456});
console.log(obj.prop); // 456
```
14.6 ES6에서 __proto__
__proto__ 속성( “dunder proto”라 발음함)은 대부분의 자바스크립트 엔진에 존재했다. 이 섹션에서는 ES6이전에 어떻게 작동했고 ES6에서는 무엇이 변했는지 설명한다.

이 섹션에서, 프로토타입체인이 뭔지 알고 있다면 도움이 된다. 필요하다면 자바스크립트를 말하다의  “Layer 2: The Prototype Relationship Between Objects”를 참고하라.

14.6.1 ES6이전의 __proto__
14.6.1.1 Prototypes
자바스크립트의 각 객체는 하나이상의 오브젝트 체인으로 시작되는데 이를 프로토타입체인이라 부른다. 각 오브젝트는 내부 속성[[Prototype]]통해 자신의 프로토타입을 상속받은 객체로 가리킨다(상속자가 없는 경우는 null이다). 그 속성은 내부에서 호출되고 언어 사양에만 존재하기 때문에 자바스크립에서 직접 접근할 수 없다. ES5에서 오브젝트 ob의 프로토타입을 얻는 표준적인 방법은 다음과 같다:
```javascript
var p = Object.getPrototypeOf(obj);
```
존재하는 오브젝트의 프로토타입을 바꾸는 표준적인 방법은 없지만 주어진 프로토타입 p로 새로운 오브젝트 obj를 생성할수는 있다:
```javascript
var obj = Object.create(p);
```
14.6.1.2 __proto__
오래전, 파이어폭스는 비표준 속성인 __proto__를 갖고 있었다. 다른 브라우저들은 결국 그 사양이 인기있었기 때문에 따라했다.

ES6이전에, __proto__ 는 모호하게 작동했다:

아무 오브젝트의 프로토타입에 set하거나 get하기 위해 사용할 수 있었다:
```javascript
  var obj = {};
  var p = {};

  console.log(obj.__proto__ === p); // false
  obj.__proto__ = p;
  console.log(obj.__proto__ === p); // true
```
하지만, 절대로 실제 속성은 아니다.
```javascript
  > var obj = {};
  > '__proto__' in obj
  false
```
14.6.1.3 Array를 상속한 클래스와 __proto__
__proto__가 인기있었던 주된 이유는 ES5에서 Array를 상속할 수 있는 유일한 방법이었기 때문이다: 배열 인스턴스는 보통 생성자로는 생성할 수 없는 exotic 오브젝트다. 그러므로 다음의 트릭이 사용되었다:
```javascript
function MyArray() {
    var instance = new Array(); // exotic object
    instance.__proto__ = MyArray.prototype;
    return instance;
}
MyArray.prototype = Object.create(Array.prototype);
MyArray.prototype.customMethod = function (···) { ··· };
```
ES6에서 클래스를 상속하는 것은 ES5와 다르게 작동하며 빌트인객체에 대한 상속을 지원한다.

14.6.1.4 왜 ES5에서 __proto__은 문제가 되었나
중요한 문제는 __proto__ 가 두개의 층을 섞어버린다는 것이다: 오브젝트층 (값을 갖고 있는 일반 속성)과  메타층.

만약 실수로 일반속성층에서 데이터를 저장하기 위해 __proto__를 사용한다면 두개의 층이 충돌하여 문제에 봉착할 것이다. ES5에서 Map에 맞는 빌트인 객체가 없기 때문에 Map로서 작동하는 오브젝트를 사용해야하므로 상황은 더 악화된다.
Map은 임의의 키를 갖을 수 있지만 Map으로 사용될 오브젝트에 '__proto__'키는 사용할 수 없다.

이론적으로, 특별한 이름 __proto__을 대신할 심볼을 사용하면 한 가지는 해결되겠지만 (Object.getPrototypeOf()를 통해)메타연산을 완전히 분리하는 것이 최적의 접근이다.

14.6.2 ES6에서 두가지 종류의  __proto__
 __proto__가 광범위하게 지원되므로 ES6에서 표준화하기로 결정되었다. 그러나 본래 문제가 있으므로 파기될 항목으로 추가되었다. 이러한 항목들은 스펙문서의 annexB에 있으면 다음과 같이 기술되어있다:

ECMAScript언어 문법과 의미론은 브라우저에서 ECMAScript 호스트가 웹브라우저인 경우 부록에 있는 정의를 요구한다. 이 부록의 항목들은 웹브라우저가 아닌 호스트에서는 규범적이지만 선택적이다

자바스크립트는 웹상에 있는 상당량의 코드에서 요구하는 몇가지 바람직하지 않은 기능을 갖고 있다. 그러므로 웹브라우저는 부록을 반드시 구현해야하지만 다른 자바스크립트 엔진은 그럴 필요가 없다.

__proto__뒤의 마법을 설명하기 위해 ES6에서는 두 가지 매커니즘을 소개한다:

Object.prototype.__proto__에 getter와 setter가 구현되어있다
오브젝트 리터럴에서 속성키'__proto__'를 생성된 오브젝트의 프로토타입을 특정짓는 특수기능으로 고려해야한다.
14.6.2.1 Object.prototype.__proto__
ES6는 __proto__속성을 얻거나 설정하는게 가능하고 이는 Object.prototype에 getter와 setter로 저장되어있다. 직접 이를 구현하고 싶다면 대략적으로 아래와 같을 것이다:
```javascript
Object.defineProperty(Object.prototype, '__proto__', {
    get() {
        const _thisObj = Object(this);
        return Object.getPrototypeOf(_thisObj);
    },
    set(proto) {
        if (this === undefined || this === null) {
            throw new TypeError();
        }
        if (!isObject(this)) {
            return undefined;
        }
        if (!isObject(proto)) {
            return undefined;
        }
        const status = Reflect.setPrototypeOf(this, proto);
        if (! status) {
            throw new TypeError();
        }
    },
});
function isObject(value) {
    return Object(value) === value;
}
```
ES6사양에서 __proto__에 대한 getter와 setter:
```javascript
get Object.prototype.__proto__
set Object.prototype.__proto__
```
14.6.2.2 오브젝트 리터럴에서 연산자로서의 __proto__키
만약 __proto__가 오브젝트 리터럴에서 따옴표 또는 따옴표가 없는 속성키로 등장한다면 리터럴로 생성된 오브젝트의 프로토타입은 그 속성의 값이다:
```javascript
> Object.getPrototypeOf({ __proto__: null })
null
> Object.getPrototypeOf({ '__proto__': null })
null
```
문자열 '__proto__'을 계산된 속성키로 사용하면 프로토타입을 변경하지 않고 자신의 속성으로 생성한다:
```javascript
> const obj = { ['__proto__']: null };
> Object.getPrototypeOf(obj) === Object.prototype
true
> Object.keys(obj)
[ '__proto__' ]
```
ES6사양에서 특별한 속성키'__proto__':
오브젝트 초기화시 __proto__ 속성이름

14.6.3 __proto__의 마법을 피하기
14.6.3.1 __proto__를 속성정의하기(할당하지않고)
ES6에서 자신의 __proto__속성을 정의하면 Object.prototype.__proto__의 getter와 setter를 덮어써 특별한 기능이 발동하지 않게 한다:
```javascript
const obj = {};
Object.defineProperty(obj, '__proto__', { value: 123 })

Object.keys(obj); // [ '__proto__' ]
console.log(obj.__proto__); // 123
```
14.6.3.2 오브젝트는 프로토타입으로 Object.prototype를 갖지 않음
 __proto__ getter/setter는 Object.prototype에서 제공된다. 그러므로 자신의 프로토타입체인에 Object.prototype가 없는 오브젝트는 getter/setter가 없다. 다음 코드에서 dict은 그러한 객체의 예다 – 프로토타입을 갖고 있지 않은. 결과적으로 __proto__은 다른 속성처럼 작동한다:
```javascript
> const dict = Object.create(null);
> '__proto__' in dict
false
> dict.__proto__ = 'abc';
> dict.__proto__
'abc'
```
14.6.3.3 __proto__와 dict 오브젝트
오브젝트를 딕셔너리처럼 사용하려면 프로토타입을 갖지 않게 하는게 최고다. 프로토타입이 없는 객체가 왜 dict오브젝트라고도 불리는지에 대한 이유다. ES6에서 dict오브젝트에서 '__proto__'를 예외처리할 필요도 없다. 왜냐면 아무런 특수기능이 발동되지 않기 때문이다.

오브젝트리터럴에서 연산자로서 __proto__는  보다 간결하게 dict오브젝트를 만들 수 있게 한다:
```javascript
const dictObj = {
    __proto__: null,
    yes: true,
    no: false,
};
```
ES6에선 보통 내장객체 Map을 dict오브젝트보다 선호하며 특히 키가 고정되어있지 않은 경우는 더욱 그렇다.

14.6.3.4 __proto__ 과 JSON
ES6이전에는 다음의 코드가 자바스크립트 엔진에서 일어날 수 있었다:
```javascript
> JSON.parse('{"__proto__": []}') instanceof Array
true
```
__proto__가 getter/setter 인 ES6에서도, JSON.parse()는 속성을 할당하지 않고 정의하므로 잘 작동한다(정확히 하지면, V8의 오래된 버전에서는 할당한다).

JSON.stringify() 또한 자신의 속성만 고려하기 때문에 __proto__가 영향을 받지 않는다. __proto__를 자신의 속성이름으로 갖고 있는 오브젝트는 잘 작동한다:
```javascript
> JSON.stringify({['__proto__']: true})
'{"__proto__":true}'
```
14.6.4 ES6방식의 __proto__를 지원하는지 알아내기
ES6방식의 __proto__를 지원하는지는 엔진에서 엔진까지 있다.  Consult kangax’ ECMAScript 6 compatibility table에 항목에 정보가 있다:
Object.prototype.__proto__
__proto__ in object literals
다음 두 섹션에서 __proto__의 두 종류를 엔진에서 지원하는 프로그래밍적으로 알아내는 방법을 설명한다.

14.6.4.1 getter/setter로 __proto__ 가 작동하는가
getter/setter를 위한 간단한 체크:
```javascript
var supported = {}.hasOwnProperty.call(Object.prototype, '__proto__');
```
보다 정교한 체크:
```javascript
var desc = Object.getOwnPropertyDescriptor(Object.prototype, '__proto__');
var supported = (
    typeof desc.get === 'function' && typeof desc.set === 'function'
);
```
14.6.4.2 오브젝트 리터럴에서 __proto__가 연산자로 작동하는가
다음을 이용해 체크:
```javascript
var supported = Object.getPrototypeOf({__proto__: null}) === null;
```
14.6.5 __proto__은 “dunder proto”라 발음됨.

두개의 언더스코어로 감싸진 이름은 보통 파이쎤에서 메타데이터와 데이터 사이의 이름충돌을 피하기 위한것이다.

Bracketing names with double underscores is a common practice in Python to avoid name clashes between meta-data (such as __proto__) and data (user-defined properties). That practice will never become common in JavaScript, because it now has symbols for this purpose. However, we can look to the Python community for ideas on how to pronounce double underscores.

The following pronunciation has been suggested by Ned Batchelder:

An awkward thing about programming in Python: there are lots of double underscores. For example, the standard method names beneath the syntactic sugar have names like __getattr__, constructors are __init__, built-in operators can be overloaded with __add__, and so on. […]

My problem with the double underscore is that it’s hard to say. How do you pronounce __init__? “underscore underscore init underscore underscore”? “under under init under under”? Just plain “init” seems to leave out something important.

I have a solution: double underscore should be pronounced “dunder”. So __init__ is “dunder init dunder”, or just “dunder init”.

Thus, __proto__ is pronounced “dunder proto”. The chances for this pronunciation catching on are good, JavaScript creator Brendan Eich uses it.

14.6.6 Recommendations for __proto__
It is nice how well ES6 turns __proto__ from something obscure into something that is easy to understand.

However, I still recommend not to use it. It is effectively a deprecated feature and not part of the core standard. You can’t rely on it being there for code that must run on all engines.

More recommendations:

Use Object.getPrototypeOf() to get the prototype of an object.
Prefer Object.create() to create a new object with a given prototype. Avoid Object.setPrototypeOf(), which hampers performance on many engines.
I actually like __proto__ as an operator in an object literal. It is useful for demonstrating prototypal inheritance and for creating dict objects. However, the previously mentioned caveats do apply.
14.7 Enumerability in ECMAScript 6
Enumerability is an attribute of object properties. This section explains how it works in ECMAScript 6. Let’s first explore what attributes are.

14.7.1 Property attributes
Each object has zero or more properties. Each property has a key and three or more attributes, named slots that store the data of the property (in other words, a property is itself much like a JavaScript object or like a record with fields in a database).

ECMAScript 6 supports the following attributes (as does ES5):

All properties have the attributes:
enumerable: Setting this attribute to false hides the property from some operations.
configurable: Setting this attribute to false prevents several changes to a property (attributes except value can’t be change, property can’t be deleted, etc.).
Normal properties (data properties, methods) have the attributes:
value: holds the value of the property.
writable: controls whether the property’s value can be changed.
Accessors (getters/setters) have the attributes:
get: holds the getter (a function).
set: holds the setter (a function).
You can retrieve the attributes of a property via Object.getOwnPropertyDescriptor(), which returns the attributes as a JavaScript object:
```javascript
> const obj = { foo: 123 };
> Object.getOwnPropertyDescriptor(obj, 'foo')
{ value: 123,
  writable: true,
  enumerable: true,
  configurable: true }
```
This section explains how the attribute enumerable works in ES6. All other attributes and how to change attributes is explained in Sect. “Property Attributes and Property Descriptors” in “Speaking JavaScript”.

14.7.2 Constructs affected by enumerability
ECMAScript 5:

for-in loop: iterates over the string keys of own and inherited enumerable properties.
Object.keys(): returns the string keys of enumerable own properties.
JSON.stringify(): only stringifies enumerable own properties with string keys.
ECMAScript 6:

Object.assign(): only copies enumerable own properties (both string keys and symbol keys are considered).
Reflect.enumerate(): returns all property names that for-in iterates over.
for-in and Reflect.enumerate() are the only built-in operations where enumerability matters for inherited properties. All other operations only work with own properties.

14.7.3 Use cases for enumerability
Unfortunately, enumerability is quite an idiosyncratic feature. This section presents several use cases for it and argues that, apart from protecting legacy code from breaking, its usefulness is limited.

14.7.3.1 Use case: Hiding properties from the for-in loop
The for-in loop iterates over all enumerable properties of an object, own and inherited ones. Therefore, the attribute enumerable is used to hide properties that should not be iterated over. That was the reason for introducing enumerability in ECMAScript 1.

14.7.3.1.1 Non-enumerability in the language
Non-enumerable properties occur in the following locations in the language:

All prototype properties of built-in classes are non-enumerable:
```javascript
  > const desc = Object.getOwnPropertyDescriptor.bind(Object);
  > desc(Object.prototype, 'toString').enumerable
  false
```
All prototype properties of classes are non-enumerable:
```javascript
  > desc(class {foo() {}}.prototype, 'foo').enumerable
  false
```
In Arrays, length is not enumerable, which means that for-in only iterates over indices. (However, that can easily change if you add a property via assignment, which is makes it enumerable.)
```javascript
  > desc([], 'length').enumerable
  false
  > desc(['a'], '0').enumerable
  true
```
The main reason for making all of these properties non-enumerable is to hide them (especially the inherited ones) from legacy code that uses the for-in loop or $.extend() (and similar operations that copy both inherited and own properties; see next section). Both operations should be avoided in ES6. Hiding them ensures that the legacy code doesn’t break.

14.7.3.2 Use case: Marking properties as not to be copied
14.7.3.2.1 Historical precedents
When it comes to copying properties, there are two important historical precedents that take enumerability into consideration:

Prototype’s Object.extend(destination, source)
```javascript
  const obj1 = Object.create({ foo: 123 });
  Object.extend({}, obj1); // { foo: 123 }

  const obj2 = Object.defineProperty({}, 'foo', {
      value: 123,
      enumerable: false
  });
  Object.extend({}, obj2) // {}
```
jQuery’s $.extend(target, source1, source2, ···) copies all enumerable own and inherited properties of source1 etc. into own properties of target.
```javascript
  const obj1 = Object.create({ foo: 123 });
  $.extend({}, obj1); // { foo: 123 }

  const obj2 = Object.defineProperty({}, 'foo', {
      value: 123,
      enumerable: false
  });
  $.extend({}, obj2) // {}
```
Problems with this way of copying properties:

Turning inherited source properties into own target properties is rarely what you want. That’s why enumerability is used to hide inherited properties.
Which properties to copy and which not often depends on the task at hand, it rarely makes sense to have a single flag for everything. A better choice is to provide the copying operation with a predicate (a callback that returns a boolean) that tells it when to consider a property.
The only instance property that is non-enumerable in the standard library is property length of Arrays. However, that property only needs to be hidden due to it magically updating itself via other properties. You can’t create that kind of magic property for your own objects (short of using a Proxy).

14.7.3.2.2 ES6: Object.assign()
In ES6, Object.assign(target, source_1, source_2, ···) can be used to merge the sources into the target. All own enumerable properties of the sources are considered (that is, keys can be either strings or symbols). Object.assign() uses a “get” operation to read a value from a source and a “set” operation to write a value to the target.

With regard to enumerability, Object.assign() continues the tradition of Object.extend() and $.extend(). Quoting Yehuda Katz:

Object.assign would pave the cowpath of all of the extend() APIs already in circulation. We thought the precedent of not copying enumerable methods in those cases was enough reason for Object.assign to have this behavior.

In other words: Object.assign() was created with an upgrade path from $.extend() (and similar) in mind. Its approach is cleaner than $.extend’s, because it ignores inherited properties.

14.7.3.3 Marking properties as private
If you make a property non-enumerable, it can’t by seen by Object.keys() and the for-in loop, anymore. With regard to those mechanisms, the property is private.

However, there are several problems with this approach:

When copying an object, you normally want to copy private properties. That clashes making properties non-enumerable that shouldn’t be copied (see previous section).
The property isn’t really private. Getting, setting and several other mechanisms make no distinction between enumerable and non-enumerable properties.
When working with code either as source or interactively, you can’t immediately see whether a property is enumerable or not. A naming convention (such as prefixing property names with an underscore) is easier to discover.
You can’t use enumerability to distinguish between public and private methods, because methods in prototypes are non-enumerable by default.
14.7.3.4 Hiding own properties from JSON.stringify()
JSON.stringify() does not include properties in its output that are non-enumerable. You can therefore use enumerability to determine which own properties should be exported to JSON. This use case is similar to marking properties as private, the previous use case. But it is also different, because this is more about exporting and slightly different considerations apply. For example: Can an object be completely reconstructed from JSON?

An alternative for specifying how an object should be converted to JSON is to use toJSON():
```javascript
const obj = {
    foo: 123,
    toJSON() {
        return { bar: 456 };
    },
};
JSON.stringify(obj); // '{"bar":456}'
```
I find toJSON() cleaner than enumerability for the current use case. It also gives you more control, because you can export properties that don’t exist on the object.

14.7.4 Naming inconsistencies
In general, a shorter name means that only enumerable properties are considered:

Object.keys() ignores non-enumerable properties
Object.getOwnPropertyNames() lists all property names
However, Reflect.ownKeys() deviates from that rule, it ignores enumerability and returns the keys of all properties. Additionally, starting with ES6, the following distinction is made:

Property keys are either strings or symbols.
Property names are only strings.
Therefore, a better name for Object.keys() would now be Object.names().

14.7.5 Looking ahead
It seems to me that enumerability is only suited for hiding properties from the for-in loop and $.extend() (and similar operations). Both are legacy features, you should avoid them in new code. As for the other use cases:

I don’t think there is a need for a general flag specifying whether or not to copy a property.
Non-enumerability does not work well as a way to keep properties private.
The toJSON() method is more powerful and explicit than enumerability when it comes to controlling how to convert an object to JSON.
I’m not sure what the best strategy is for enumerability going forward. If, with ES6, we had started to pretend that it didn’t exist (except for making prototype properties non-enumerable so that old code doesn’t break), we might eventually have been able to deprecate enumerability. However, Object.assign() considering enumerability runs counter that strategy (but it does so for a valid reason, backward compatibility).

In my own ES6 code, I’m not using enumerability, except (implicitly) for classes whose prototype methods are non-enumerable.

Lastly, when using an interactive command line, I occasionally miss an operation that returns all property keys of an object, not just the own ones (Reflect.ownKeys) or not just string-valued enumerable ones (Reflect.enumerate). Such an operation would provide a nice overview of the contents of an object.

14.8 Customizing basic language operations via well-known symbols
This section explains how you can customize basic language operations by using the following well-known symbols as property keys:
Symbol.hasInstance (method)
Lets an object C customize the behavior of x instanceof C.
Symbol.toPrimitive (method)
Lets an object customize how it is converted to a primitive value. This is the first step whenever something is coerced to a primitive type (via operators etc.).
Symbol.toStringTag (string)
Called by Object.prototype.toString() to compute the default string description of an object obj: ‘[object ‘+obj[Symbol.toStringTag]+’]’.
Symbol.unscopables (Object)
Lets an object hide some properties from the with statement.
14.8.1 Property key Symbol.hasInstance (method)
An object C can customize the behavior of the instanceof operator via a method with the key Symbol.hasInstance that has the following signature:
[Symbol.hasInstance](potentialInstance : any)
x instanceof C works as follows in ES6:

If C is not an object, throw a TypeError.
If the method exists, call C[Symbol.hasInstance](x), coerce the result to boolean and return it.
Otherwise, compute and return the result according to the traditional algorithm (C must be callable, C.prototype in the prototype chain of x, etc.).
14.8.1.1 Uses in the standard library
The only method in the standard library that has this key is:

Function.prototype[Symbol.hasInstance]()
This is the implementation of instanceof that all functions (including classes) use by default. Quoting the spec:

This property is non-writable and non-configurable to prevent tampering that could be used to globally expose the target function of a bound function.

The tampering is possible because the traditional instanceof algorithm, OrdinaryHasInstance(), applies instanceof to the target function if it encounters a bound function.

Given that this property is read-only, you can’t use assignment to override it, as mentioned earlier.

14.8.1.2 Example: checking whether a value is an object
As an example, let’s implement an object ReferenceType whose “instances” are all objects, not just objects that are instances of Object (and therefore have Object.prototype in their prototype chains).
```javascript
const ReferenceType = {
    [Symbol.hasInstance](value) {
        return (value !== null
            && (typeof value === 'object'
                || typeof value === 'function'));
    }
};
const obj1 = {};
console.log(obj1 instanceof Object); // true
console.log(obj1 instanceof ReferenceType); // true

const obj2 = Object.create(null);
console.log(obj2 instanceof Object); // false
console.log(obj2 instanceof ReferenceType); // true
```
14.8.2 Property key Symbol.toPrimitive (method)
Symbol.toPrimitive lets an object customize how it is coerced (converted automatically) to a primitive value.

Many JavaScript operations coerce values to the types that they need.

The multiplication operator (*) coerces its operands to numbers.
new Date(year, month, date) coerces its parameters to numbers.
parseInt(string , radix) coerces its first parameter to a string.
The following are the most common types that values are coerced to:

Boolean: Coercion returns true for truthy values, false for falsy values. Objects are always truthy (even new Boolean(false)).
Number: Coercion converts objects to primitives first. Primitives are then converted to numbers (null → 0, true → 1, '123' → 123, etc.).
String: Coercion converts objects to primitives first. Primitives are then converted to strings (null → 'null', true → 'true', 123 → '123', etc.).
Object: The coercion wraps primitive values (booleans b via new Boolean(b), numbers n via new Number(n), etc.).
Thus, for numbers and strings, the first step is to ensure that a value is any kind of primitive. That is handled by the spec-internal operation ToPrimitive(), which has three modes:

Number: the caller needs a number.
String: the caller needs a string.
Default: the caller needs either a number or a string.
The default mode is only used by:

Equality operator (==)
Addition operator (+)
new Date(value) (exactly one parameter!)
If the value is a primitive then ToPrimitive() is already done. Otherwise, the value is an object obj, which is converted to a primitive as follows:

Number mode: Return the result of obj.valueOf() if it is primitive. Otherwise, return the result of obj.toString() if it is primitive. Otherwise, throw a TypeError.
String mode: works like Number mode, but toString() is called first, valueOf() second.
Default mode: works exactly like Number mode.
This normal algorithm can be overridden by giving an object a method with the following signature:

[Symbol.toPrimitive](hint : 'default' | 'string' | 'number')
In the standard library, there are two such methods:

Symbol.prototype[Symbol.toPrimitive](hint) prevents toString() from being called (which throws an exception).
Date.prototype[Symbol.toPrimitive](hint) This method implements behavior that deviates from the default algorithm. Quoting the specification: “Date objects are unique among built-in ECMAScript object in that they treat 'default' as being equivalent to 'string'. All other built-in ECMAScript objects treat 'default' as being equivalent to 'number'.”
14.8.2.1 Example
The following code demonstrates how coercion affects the object obj.
```javascript
const obj = {
    [Symbol.toPrimitive](hint) {
        switch (hint) {
            case 'number':
                return 123;
            case 'string':
                return 'str';
            case 'default':
                return 'default';
            default:
                throw new Error();
        }
    }
};
console.log(2 * obj); // 246
console.log(3 + obj); // '3default'
console.log(obj == 'default'); // true
console.log(String(obj)); // 'str'
```
14.8.3 Property key Symbol.toStringTag (string)
In ES5 and earlier, each object had the internal own property [[Class]] whose value hinted at its type. You could not access it directly, but its value was part of the string returned by Object.prototype.toString(), which is why that method was used for type checks, as an alternative to typeof.

In ES6, there is no internal property [[Class]], anymore, and using Object.prototype.toString() for type checks is discouraged. In order to ensure the backwards-compatibility of that method, the public property with the key Symbol.toStringTag was introduced. You could say that it replaces [[Class]].
Object.prototype.toString() now works as follows:

Convert this to an object obj.
Determine the toString tag tst of obj.
Return '[object ' + tst + ']'.
14.8.3.1 Default toString tags
The default values for various kinds of objects are shown in the following table.

Value	toString tag
undefined	'Undefined'
null	'Null'
An Array object	'Array'
A string object	'String'
arguments	'Arguments'
Something callable	'Function'
An error object	'Error'
A boolean object	'Boolean'
A number object	'Number'
A date object	'Date'
A regular expression object	'RegExp'
(Otherwise)	'Object'
Most of the checks in the left column are performed by looking at internal properties. For example, if an object has the internal property [[Call]], it is callable.

The following interaction demonstrates the default toString tags.
```javascript
> Object.prototype.toString.call(null)
'[object Null]'
> Object.prototype.toString.call([])
'[object Array]'
> Object.prototype.toString.call({})
'[object Object]'
> Object.prototype.toString.call(Object.create(null))
'[object Object]'
```
14.8.3.2 Overriding the default toString tag
If an object has an (own or inherited) property whose key is Symbol.toStringTag then its value overrides the default toString tag. For example:
```javascript
> ({}.toString())
'[object Object]'
> ({[Symbol.toStringTag]: 'Foo'}.toString())
'[object Foo]'
```
Instances of user-defined classes get the default toString tag (of objects):
```javascript
class Foo { }
console.log(new Foo().toString()); // [object Object]
```
One option for overriding the default is via a getter:
```javascript
class Bar {
    get [Symbol.toStringTag]() {
      return 'Bar';
    }
}
console.log(new Bar().toString()); // [object Bar]
```
In the JavaScript standard library, there are the following custom toString tags. Objects that have no global names are quoted with percent symbols (for example: %TypedArray%).

Module-like objects:
JSON[Symbol.toStringTag] → 'JSON'
Math[Symbol.toStringTag] → 'Math'
Actual module objects M: M[Symbol.toStringTag] → 'Module'
Built-in classes
ArrayBuffer.prototype[Symbol.toStringTag] → 'ArrayBuffer'
DataView.prototype[Symbol.toStringTag] → 'DataView'
Map.prototype[Symbol.toStringTag] → 'Map'
Promise.prototype[Symbol.toStringTag] → 'Promise'
Set.prototype[Symbol.toStringTag] → 'Set'
get %TypedArray%.prototype[Symbol.toStringTag] → 'Uint8Array' etc.
WeakMap.prototype[Symbol.toStringTag] → 'WeakMap'
WeakSet.prototype[Symbol.toStringTag] → 'WeakSet'
Iterators
%MapIteratorPrototype%[Symbol.toStringTag] → 'Map Iterator'
%SetIteratorPrototype%[Symbol.toStringTag] → 'Set Iterator'
%StringIteratorPrototype%[Symbol.toStringTag] → 'String Iterator'
Miscellaneous
Symbol.prototype[Symbol.toStringTag] → 'Symbol'
Generator.prototype[Symbol.toStringTag] → 'Generator'
GeneratorFunction.prototype[Symbol.toStringTag] → 'GeneratorFunction'
All of the built-in properties whose keys are Symbol.toStringTag have the following property descriptor:
```javascript
{
    writable: false,
    enumerable: false,
    configurable: true,
}
```
As mentioned earlier, you can’t use assignment to override those properties, because they are read-only.

14.8.4 Property key Symbol.unscopables (Object)
Symbol.unscopables lets an object hide some properties from the with statement.

The reason for doing so is that it allows TC39 to add new methods to Array.prototype without breaking old code. Note that current code rarely uses with, which is forbidden in strict mode and therefore ES6 modules (which are implicitly in strict mode).

Why would adding methods to Array.prototype break code that uses with (such as the widely deployed Ext JS 4.2.1)? Take a look at the following code. The existence of a property Array.prototype.values breaks foo(), if you call it with an Array:
```javascript
function foo(values) {
    with (values) {
        console.log(values.length); // abc (*)
    }
}
Array.prototype.values = { length: 'abc' };
foo([]);
```
Inside the with statement, all properties of values become local variables, shadowing even values itself. Therefore, if values has a property values then the statement in line * logs values.values.length and not values.length.

Symbol.unscopables is used only once in the standard library:

Array.prototype[Symbol.unscopables]
Holds an object with the following properties (which are therefore hidden from the with statement): copyWithin, entries, fill, find, findIndex, keys, values
14.9 FAQ: object literals
14.9.1 Can I use super in object literals?
Yes you can! Details are explained in the chapter on classes.

