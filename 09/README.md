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
> 일시적 사각 지대. 해당 영역에 선언은 되지만 참조 불가의 undefined 상태로 선언되는 것을 의미한다.
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

일단 const 변수가 생성된 후엔 변경할 수 없다. 하지만 그게 당신이 새로운 스코프에 새 변수로 리프레쉬할수 없다는 걸 의미하진 않는다.

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

