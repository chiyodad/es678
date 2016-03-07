#  9. Variables and scoping
변수와 스코핑

이번 장은 변수와 스코핑이 ECMAScript 6 에서 어떻게 핸들링 되는지 고찰해본다

##  9.1 개요
ES6에서는 두가지의 새로운 변수 선언을 제공한다. let 과 const 이다. var 를 사용하는 ES5의 변수 선언법을 거의 대체한다.

###  9.1.1 let
let 은 var 와 비슷하게 동작하지만, 그것이 선언하는 변수는 블럭 스코프이다. 그것은 단지 현재 블럭에서만 존재한다. var 는 함수 스코프이다.

다음 코드에 따르면 당신은 let 선언 변수인 tmp 가 단지 A 라인에서 시작한 블럭에만 존재하는걸 볼 수 있다


```javascript
function order(x, y) {
    if (x > y) { // (A)
        let tmp = x;
        x = y;
        y = tmp;
    }
    console.log(tmp===x); // ReferenceError: tmp is not defined
    return [x, y];
}
```

### 9.1.2 const
const 는 let 과 비슷하게 동작하지만 당신은 나중에 값을 변경할 수 없으며, 즉시 초기화로만 선언해야하는 변수이다.

```javascript
const foo; // SyntaxError: missing = in const declaration

const bar = 123;
bar = 456; // TypeError: `bar` is read-only
```

for-of 루프 순회당 한번의 바인딩 (변수를 위한 저장 공간을) 생성하기에 루프 변수로서의 const 선언은 OK 이다. 

```javascript
for (const x of ['a', 'b']) {
    console.log(x);
}
// Output:
// a
// b
```

###  9.1.3 변수 선언 방법들
다음 테이블은 ES6에서 변수 선언이 가능한 여섯가지 방법을 보여주고 있다.

| | Hoisting | Scope | Creates global properties |
| -------- | ----- | ------- | ------ | ---------- |
| var | Declaration | Function | Yes |
| let | Temporal dead zone | Block | No |
| const | Temporal dead zone | Block | No |
| function | Complete | Block | Yes |
| class | No | Block | No |
| import | Complete | Module-global | No |

> [Temporal dead zone (TDZ)]
>
> 임시 사각 지대. 해당 영역에 선언은 되지만 참조 불가의 undefined 상태로 선언되는 것을 의미한다.
> let, const 가 이에 해당되고 함수의 기본 파라미터도 TDZ 에 해당하기에 조심해야 한다

##  9.2 let 과 const 를 통한 블럭 스코핑

let 과 const 는 블럭 스코프 변수를 생성한다 - 그것들은 단지 자신을 둘러싸고 있는 블럭 내부에서만 존재한다. 다음 코드는 const 로 선언된 변수 tmp 가 단지 if 문의 다음 블럭 안에서만 존재하는걸 보여준다

```javascript
function func() {
    if (true) {
        const tmp = 123;
    }
    console.log(tmp); // ReferenceError: tmp is not defined
}
```

대조적으로, var 선언 변수는 함수 스코프이다

```javascript
function func() {
    if (true) {
        var tmp = 123;
    }
    console.log(tmp); // 123
}
```

블럭 스코핑은 당신이 변수를 함수 안에서 가릴 수 있다는 걸 뜻한다

```javascript
function func() {
  const foo = 5;
  if (···) {
     const foo = 10; // shadows outer `foo`
     console.log(foo); // 10
  }
  console.log(foo); // 5
}
```

## 9.3 const 는 불변 (immutable) 변수를 생성한다
let 으로 생성한 변수는 변한다 (mutable)

```javascript
let foo = 'abc';
foo = 'def';
console.log(foo); // def
```

const 로 생성된 상수들, 변수들은 불변이다. - 당신은 그 변수들에 다른 값을 할당할 수 없다.

```javascript
const foo = 'abc';
foo = 'def'; // TypeError
```

스펙에 따르면 const 변수의 변경은 항상 TypeError 를 던진다

일반적으로 strict mode에서 불변 바인딩의 변경은 SetMutableBinding 에 따라서, 항상 예외가 일어나지만, const 변수 선언은 항상 엄격한 바인딩을 생성한다.

step 35.b.i.1 장의 FunctionDeclarationInstantiation(func, argumentsList) 을 참고하라.

### 9.3.1 함정!(Pitfall) : const 는 값의 불변을 만들지 않는다.
const only means that a variable always has the same value, but it does not mean that the value itself is or becomes immutable. For example, obj is a constant, but the value it points to is mutable – we can add a property to it:

const 는 단지 변수가 항상 같은 값을 가지는 것을 뜻하지만, 그것이 값 자체이거나 불변이 되는 것은 아니다.

예를 들면, obj 는 상수이다. 하지만 그 값은 변경 가능한 포인트 - 우리는 그것에 속성을 추가할 수 있다.

```javascript
const obj = {};
obj.prop = 123;
console.log(obj.prop); // 123
```

하지만 우린 obj 에 다른 값을 할당할 수 없다. 

```javascript
obj = {}; // TypeError
```

만일 당신이 obj 값의 불면을 원한다면, 당신 자신이 알아서 해야 한다. 예를 들면 그걸 프리징한다든가.

```javascript
const obj = Object.freeze({});
obj.prop = 123; // TypeError
```

#### 9.3.1.1 함정!: Object.freeze() 는 얕다.
Keep in mind that Object.freeze() is shallow, it only freezes the properties of its argument, not the objects stored in its properties. For example, the object obj is frozen:
Object.freeze() 는 얕다는걸 알아둬라. 그건 단지 그 인수의 프로퍼티들을 프리징할 뿐, 속성에 저장된 객체에는 아니다.

예를 들면, 오브젝트 obj 는 얼었다 (frozen)

```javascript
const obj = Object.freeze({ foo: {} });
obj.bar = 123
// TypeError: Can't add property bar, object is not extensible
obj.foo = {}
// TypeError: Cannot assign to read only property 'foo' of #<Object>
```

하지만, object obj.foo 는 아니다.

```javascript
obj.foo.qux = 'abc';
obj.foo.qux
// 'abc'
```

### 9.3.2 루프 바디 안에서의 const

일단 const 변수가 생성된 후엔 변경할 수 없다. 하지만 그게 당신이 새로운 스코프에 새 변수로 새로운 시작(start fresh)을 할 수 없다는 걸 의미하진 않는다.

예를들면 루프를 통해서.

```javascript
function logArgs(...args) {
    for (const [index, elem] of args.entries()) {
        const message = index + '. ' + elem;
        console.log(message);
    }
}
logArgs('Hello', 'everyone');

// Output:
// 0. Hello
// 1. everyone
```

## 9.4 임시 사각 지대 (The temporal dead zone)

let 혹은 const 변수선언은 소위 임시 사각 지대 (temporal dead zone - TDZ) 를 갖는다.

스코프에 들어가면, 실행 선언문에 도달할 때까지 그것에 엑세스 될(얻거나, 혹은 할당하거나) 수 없다.

var 선언 변수 (TDZ 를 가지고 있지 않은)와 let 선언 변수(TDZ를 가진) 의 라이프 사이클을 비교해보자

### 9.4.1 var로 선언된 변수의 라이프사이클
var 변수는 임시 사각 지대를 갖지 않는다. 그들의 라이프사이클은 다음 스텝을 포함한다.

1. var 변수가 스코프(function 으로 감싸진) 에 들어가면, 변수를 위한 저장 공간이 만들어진다 (바인딩). 그 변수는 즉시 *underfined* 로 초기화된다. 
2. 실행이 선언문에 도달하면, 그 변수는 특정 이니셜라이저 (할당) 결과로 값이 있을 경우 설정된다. 만일 없다면 변수의 값은 여전히 undefined 상태로 남아있다.

### 9.4.2 let 으로 선언된 변수의 라이프사이클
let 을 통한 변수 선언은 임시 사각 지대 (temporal dead zone) 를 가지고, 그 라이프사이클은 이것과 같다.

1. let 변수가 스코프(블럭으로 감싸진) 에 들어가면, 변수를 위한 저장 공간이 만들어진다 (바인딩). 그 변수는 *초기화되지 않았다*. 
2. 초기화되지 않은 변수에 값을 얻기 혹은 설정하기는 ReferenceError 를 발생시킨다.
3. 실행이 선언문에 도달하면, 그 변수는 특정 이니셜라이저 (할당) 결과로 값이 있을 경우 설정된다. 만일 값이 없다면, 변수 값은 *undefined* 설정된다. 

const 변수도 let 변수와 비슷한 동작을 한다. 그러나 반드시 이니셜라이저를 가져야 하고 (예를 들면 즉시 값 설정이 되어야 한다는 뜻이다) 변경할 수 없다.

### 9.4.3 예제
Within a TDZ, an exception is thrown if a variable is got or set:

TDZ 내에서 변수를 얻거나 설정하면 예외가 던져진다.

```javascript
let tmp = true;
if (true) { // enter new scope, TDZ starts
    // Uninitialized binding for `tmp` is created
    console.log(tmp); // ReferenceError

    let tmp; // TDZ ends, `tmp` is initialized with `undefined`
    console.log(tmp); // undefined

    tmp = 123;
    console.log(tmp); // 123
}
console.log(tmp); // true
```
이니셜라이저라면, TDZ는 할당이 된 뒤 종료한다.(???)
(If there is an initializer then the TDZ ends after the assignment was made)

```javascript
let foo = console.log(foo); // ReferenceError
```

The following code demonstrates that the dead zone is really temporal (based on time) and not spatial (based on location):
다음 코드는 사각 지대가 정말 일시적(시간 기준) 이고 공간 (지역 기준)이 아닌걸 보여준다.

```javascript
if (true) { // enter new scope, TDZ starts
    const func = function () {
        console.log(myVar); // OK!
    };

    // Here we are within the TDZ and
    // accessing `myVar` would cause a `ReferenceError`

    let myVar = 3; // TDZ ends
    func(); // called outside TDZ
}
```

### 9.4.4 typeof 는 TDZ 안에서 변수에 사용할 때 ReferenceError 를 던진다

당신이 typeof 를 통해 임시 사각 지대안의 변수에 접근하면 에러를 얻는다.

```javascript
if (true) {
    console.log(typeof foo); // ReferenceError (TDZ)
    console.log(typeof aVariableThatDoesntExist); // 'undefined'
    let foo;
}
```

왜? 그 답은 다음과 같다: foo 는 선언되지 않았고, 초기화되지 않았다. 당신은 그 존재를 인식하지만 그러지 않는다. 따라서 경고됨이 바람직해 보인다.

또한 이런 종류의 체크는 단지 조건부 전역 변수를 만들기에 유용할 뿐이다. 그건 숙련된 자바스크립트 프로그래머가 해야할 일(Something) 이고, var 를 통해서도 할 수 있다.

typeof 를 사용하지 않고, 전역 변수의 존재를 체크하는 방법이 있다.

```javascript
// With `typeof`
if (typeof someGlobal === 'undefined') {
    var someGlobal = { ··· };
}

// Without `typeof`
if (!('someGlobal' in window)) {
    window.someGlobal = { ··· };
}
```

전역 변수를 생성하는 전자의 방법은 단지 글로벌 스코프에서만 동작한다 (따라서, ES6의 모듈이 아닌 경우이다.)

### 9.4.5 왜 임시 사각 지대가 있지?

To catch programming errors: Being able to access a variable before its declaration is strange. If you do so, it is normally by accident and you should be warned about it.
For const: Making const work properly is difficult. Quoting Allen Wirfs-Brock: “TDZs … provide a rational semantics for const. There was significant technical discussion of that topic and TDZs emerged as the best solution.” let also has a temporal dead zone so that switching between let and const doesn’t change behavior in unexpected ways.
Future-proofing for guards: JavaScript may eventually have guards, a mechanism for enforcing at runtime that a variable has the correct value (think runtime type check). If the value of a variable is undefined before its declaration then that value may be in conflict with the guarantee given by its guard.

1. 프로그래밍 오류를 잡으려면 : 선언 전에 변수에 접근이 가능하다는 건 이상하다. 만일 그렇게 되면 보통 이런 건에 대해 경고해야 한다.
2. 2. const 를 들면, const 는 적당히 만들기는 어려운 일이다. Allen Wirfs-Brock 을 인용하면, 
> "TDZs … 는 const 를 만들기 위한 합리적인 문법을 제공한다. 이 주제에 대해서 여러 중요한 기술적 의견들이 있었고 TDZ는 최적의 기술로 부상했다".
let 또한 예상치못한 동작으로 let 과 const 을 변경하직 않도록 TDZ가 있다.(?)
(let also has a temporal dead zone so that switching between let and const doesn’t change behavior in unexpected ways.)
3. 미래 교정 방지


