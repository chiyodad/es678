#  9. Variables and scoping

이번 장은 변수와 스코핑이 ECMAScript 6 에서 어떻게 핸들링 되는지 살펴본다.

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

*:notebook: 스펙에 따르면 const 변수의 변경은 항상 TypeError 를 던진다*

일반적으로 strict mode에서 불변 바인딩의 변경은 SetMutableBinding 에 따라서, 항상 예외의 원인이지만, const 변수 선언은 항상 엄격한 바인딩을 생성한다.

step 35.b.i.1 장의 [FunctionDeclarationInstantiation(func, argumentsList)](http://www.ecma-international.org/ecma-262/6.0/#sec-functiondeclarationinstantiation) 을 참고하라.

### 9.3.1 함정!(Pitfall) : const 는 값의 불변을 만들지 않는다.

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
초기화가있는 경우 할당(이니셜라이즈)이 이루어진 다음에 TDZ는 종료된다.

```javascript
let foo = console.log(foo); // ReferenceError
```

다음 코드는 사각 지대가 정말 일시적(시간 기준)이고 공간(지역 기준)이 아닌걸 보여준다.

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

1. 프로그래밍 오류를 잡으려면 : 선언 전에 변수에 접근이 가능하다는 건 이상하다. 만일 그렇게 되면 보통 이런 건에 대해 경고해야 한다.
2. const : const 는 적당히 만들기는 어려운 일이다. Allen Wirfs-Brock 을 인용하면, *"TDZs … 는 const 를 만들기 위한 합리적인 문법을 제공한다. 이 주제에 대해서 여러 중요한 기술적 의견들이 있었고 TDZ는 최적의 기술로 부상했다"* let 또한 예상치못한 동작으로 let 과 const 을 변경하지 않도록 TDZ가 있다.(?)
3. 감시를 통한 미래보강 : JavaScript 은 결국 변수가 올바른 값을 갖는지 런타임 시 수행하는 메카니즘의 (런타임 체크로 생각하라) 감시가 필요하다. 만일 그 변수의 값 선언 하기 전의 undefined 라면 그 값은 감시에 의해 보증되어 충돌할 것이다.


### 9.4.6 추가 읽을거리
이 섹션에서 참고한 자료
- [“Performance concern with let/const”](https://esdiscuss.org/topic/performance-concern-with-let-const)
- [“Bug 3009 – typeof on TDZ variable”](https://bugs.ecmascript.org/show_bug.cgi?id=3009)

## 9.5 루프 헤드 안에서의 let 과 const
다음 루프는 당신이 그들의 머리에 변수를 선언하는걸 허용한다.

+ for
+ for-in
+ for-of

선언할때, 당신은 var, let 혹은 const를 사용할 수 있다. 그들 각자는 다른 효과를 가지며 다음에 설명한다.

### 9.5.1 for loop

헤드 안에서의 var 선언 변수는 변수를 위해 하나의 바인딩(저장 공간)을 만든다.

```javascript
const arr = [];
for (var i=0; i < 3; i++) {
    arr.push(() => i);
}
arr.map(x => x()); // [3,3,3]
```

Every i in the bodies of the three arrow functions refers to the same binding, which is why they all return the same value.
모든 세개의 *arrow function* 바디 안의 *i* 는 같은 바인딩이며, 모두 같은 값을 반환하는 이유가 된다.

If you let-declare a variable, a new binding is created for each loop iteration:
만일 let으로 변수를 선언하면, 새로운 바인딩이 루프 이터레이션마다 생성된다.

```javascript
const arr = [];
for (let i=0; i < 3; i++) {
    arr.push(() => i);
}
arr.map(x => x()); // [0,1,2]
```

이번엔 각각의 *i*는 하나의 특정 반복의 바인딩을 참조하여 해당 시점에서의 현재의 값을 유지한다. 따라서 각 화살표 함수가 다른 값을 반환한다.

const 는 var 처럼 동작하지만, 상수 선언 변수(const-declared) 의 초기값을 바꿀 수 없다.

반복마다 새 바인딩을 얻는다면 처음엔 이상하게 보이지만, 당신이 루프 변수를 참조하는 함수를 (이벤트 처리 등의 콜백) 만들때 매우 유용하다.

*:notebook: for loop: 스펙별 각 이터레이션 바인딩
[루프의 평가](http://www.ecma-international.org/ecma-262/6.0/#sec-for-statement-runtime-semantics-labelledevaluation)는 두번째의 var 와 세번째 경우의 let/const 처럼 처리된다. let 선언 변수만 리스트의 각 순환(perIterationLets - step 9) 에 추가되는데,  [ForBodyEvaluation](http://www.ecma-international.org/ecma-262/6.0/#sec-forbodyevaluation) 의 두번째부터 마지막 인자를 전달하는 perIterationBindings 이다*

### 9.5.2 for-of loop 와 for-in loop
for-of 루프에서는 var 는 싱글 바인딩을 생성한다.

```javascript
const arr = [];
for (var i of [0, 1, 2]) {
    arr.push(() => i);
}
arr.map(x => x()); // [2,2,2]
```

let 은 이터레이션마다 하나의 바인딩을 생성한다.

```javascript
const arr = [];
for (let i of [0, 1, 2]) {
    arr.push(() => i);
}
arr.map(x => x()); // [0,1,2]
```

const 또한 각 이터레이션마다 하나의 바인딩을 생성하지만, 불변으로 생성된 바인딩이다.

for-in 루프는 for-of 루프와 비슷한 동작을 한다.

:notebook: *for-of loop: 스펙안에서의 이터레이션의 바인딩*
for-of의 반복 바인딩은 [ForIn/OfBodyEvaluation](http://www.ecma-international.org/ecma-262/6.0/#sec-runtime-semantics-forin-div-ofbodyevaluation-lhs-stmt-iterator-lhskind-labelset) 으로 처리된다. 단계 5.B에서 새로운 환경(Environment)가 만들어지고 바인딩은 [BindingInstantiation](http://www.ecma-international.org/ecma-262/6.0/#sec-runtime-semantics-bindinginstantiation) 을 통해 그것에 추가된다  (let 을 위한 가변, const 를 위한 불변). 현재 반복 값은 nextValue 변수에 저장되고, 바인딩을 초기화하여 사용하는 두가지 방법 중 하나의 방법을 사용한다:

+ 싱글 변수의 선언(단계 5.hi) : [InitializeReferencedBinding](http://www.ecma-international.org/ecma-262/6.0/#sec-initializereferencedbinding) 을 통해 처리된다.
+ 해체(단계 5.i.iii를) : [BindingInitialization (ForDeclaration)](http://www.ecma-international.org/ecma-262/6.0/#sec-for-in-and-for-of-statements-runtime-semantics-bindinginitialization) 을 통해 처리되거나, 다른 경우 [BindingInitialization (BindingPattern)](http://www.ecma-international.org/ecma-262/6.0/#sec-destructuring-binding-patterns-runtime-semantics-bindinginitialization) 로 처리된다.

### 9.5.3 왜 이터레이션 마다의 바인딩으 유용할까?
다음은 세가지 링크를 표현하는 HTML 페이지이다.

1. 당신이 yes 를 클릭하면 그것은 일본어(ja) 로 번역된다.
2. 당신이 no 를 클릭하면 그것은 러시아어(nein) 로 번역된다.
3. 당신이 perhaps 를 클릭하면 그것은 독일어(vielleicht) 로 번역된다.

```html
<!doctype html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
    <div id="content"></div>
    <script>
        const entries = [
            ['yes', 'ja'],
            ['no', 'nein'],
            ['perhaps', 'vielleicht'],
        ];
        const content = document.getElementById('content');
        for (let [source, target] of entries) { // (A)
            content.insertAdjacentHTML('beforeend',
                `<div><a id="${source}" href="">${source}</a></div>`);
            document.getElementById(source).addEventListener(
                'click', (event) => {
                    event.preventDefault();
                    alert(target); // (B)
                });
        }
    </script>
</body>
</html>
```

