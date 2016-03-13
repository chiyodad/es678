# 15. 클래스 `Classes`

이 장에서는 ES6 클래스가 작동하는 방법에 대해 설명합니다. `This chapter explains how ES6 classes work.`

## 15.1 개요 `Overview`

클래스와 서브 클래스 `A class and a subclass:`

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `(${this.x}, ${this.y})`;
    }
}

class ColorPoint extends Point {
    constructor(x, y, color) {
        super(x, y);
        this.color = color;
    }
    toString() {
        return super.toString() + ' in ' + this.color;
    }
}
```

클래스 사용: `Using the classes:`

```javascript
> const cp = new ColorPoint(25, 8, 'green');

> cp.toString();
'(25, 8) in green'

> cp instanceof ColorPoint
true
> cp instanceof Point
true
```

그들은 주로 구식 생성자 함수를 만들 수있는 편리한 구문을 제공 : 후드, ES6 클래스는 근본적으로 새로운 무언가 없습니다. 당신은 당신의 typeof 사용하면 볼 수 있습니다 : `Under the hood, ES6 classes are not something that is radically new: They mainly provide more convenient syntax to create old-school constructor functions. You can see that if you use typeof:`

```javascript
> typeof Point
'function'
```

## 15.2 필수 `The essentials`

### 15.2.1 기본 클래스 `Base classes`

클래스는 ECMAScript6에서 다음과 같이 정의된다 : `A class is defined like this in ECMAScript 6:`

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `(${this.x}, ${this.y})`;
    }
}
```

당신은 ES5 생성자 함수처럼이 클래스를 사용 : `You use this class just like an ES5 constructor function:`

```javascript
> var p = new Point(25, 8);
> p.toString()
'(25, 8)'
```

사실, 클래스 정의의 결과는 함수이다 : `In fact, the result of a class definition is a function:`

```javascript
> typeof Point
'function'
```

그러나, 당신은뿐만 아니라 함수 호출 (이 뒤에있는 근거가 나중에 설명)를 통해, 새로운 통해 클래스를 호출 할 수 있습니다 : `However, you can only invoke a class via new, not via a function call (the rationale behind this is explained later):`

```javascript
> Point()
TypeError: Classes can’t be function-called
```

> 스펙에서, 함수 호출 클래스가 내부 방법에 방지 함수 객체의 [[Call]]. `In the spec, function-calling classes is prevented in the internal method [[Call]] of function objects.`

#### 15.2.1.1 클래스 선언은 호이스팅되지 않습니다 `Class declarations are not hoisted`

함수 선언이 호이스팅되어 범위를 입력 할 때, 그 안에 선언 된 함수는 즉시 사용할 수 있습니다 - 독립 선언 일 경우의. 즉, 나중에 선언 된 함수를 호출 할 수 있다는 것을 의미한다 : `Function declarations are hoisted: When entering a scope, the functions that are declared in it are immediately available – independently of where the declarations happen. That means that you can call a function that is declared later:`

```javascript
foo(); // works, because `foo` is hoisted

function foo() {}
```

반면, 클래스 선언은 게양되지 않습니다. 실행의 정의에 도달하고 평가 후 따라서, 클래스는 존재한다. 미리에 액세스하면 ReferenceError가 발생 : `In contrast, class declarations are not hoisted. Therefore, a class only exists after execution reached its definition and it was evaluated. Accessing it beforehand leads to a ReferenceError:`

```javascript
new Foo(); // ReferenceError

class Foo {}
```

이 제한에 대한 이유는 클래스가 값이 임의의 표현이다 절을 확장 할 수 있다는 것입니다. 그 표현이 적절한 "위치"에서 평가되어야하며, 그 평가는 호이스팅 할 수 없습니다. `The reason for this limitation is that classes can have an extends clause whose value is an arbitrary expression. That expression must be evaluated in the proper “location”, its evaluation can’t be hoisted.`

호이스팅 되지 않는 것은 당신이 생각하는 것보다 제한 이하이다. 예를 들어, 클래스 선언 앞에 오는 기능은 여전히 클래스를 참조 할 수 있습니다,하지만 당신은 당신이 함수를 호출하기 전에 클래스 선언이 평가 될 때까지 기다려야합니다. `Not having hoisting is less limiting than you may think. For example, a function that comes before a class declaration can still refer to that class, but you have to wait until the class declaration has been evaluated before you can call the function.`

```javascript
function functionThatUsesBar() {
    new Bar();
}

functionThatUsesBar(); // ReferenceError
class Bar {}
functionThatUsesBar(); // OK
```

#### 15.2.1.2 클래스 식 `Class expressions`

클래스 선언과 클래스 식 : 마찬가지로 함수, 클래스 정의 이가지, 두 개의 클래스를 정의하는 방법이 있습니다. `Similarly to functions, there are two kinds of class definitions, two ways to define a class: class declarations and class expressions.`

유사 표현을 작동하려면, 클래스 표현식은 익명으로 할 수 있습니다 : `Similarly to function expressions, class expressions can be anonymous:`

```javascript
const MyClass = class {
    ···
};
const inst = new MyClass();
```

또한 유사하게 표현 기능, 클래스 표현은 그들 내부에서만 볼 수 있습니다 이름을 가질 수 있습니다 : `Also similarly to function expressions, class expressions can have names that are only visible inside them:`

```javascript
const MyClass = class Me {
    getClassName() {
        return Me.name;
    }
};
const inst = new MyClass();

console.log(inst.getClassName()); // Me
console.log(Me.name); // ReferenceError: Me is not defined
```

마지막 두 라인 째는 클래스 변수 밖에되지 않지만, 내부에 사용될 수 있음을 보여준다. `The last two lines demonstrate that Me does not become a variable outside of the class, but can be used inside it.`

### 15.2.2 클래스 정의 체내 `Inside the body of a class definition`

클래스 본문은 방법,하지만 데이터 속성을 포함 할 수 있습니다. 데이터 특성을 갖는 프로토 타입은 일반적으로 안티 패턴으로 간주됩니다, 그래서 이것은 단지 최선의 방법을 적용합니다. `A class body can only contain methods, but not data properties. Prototypes having data properties is generally considered an anti-pattern, so this just enforces a best practice.`

#### 15.2.2.1 생성자, 정적 메소드, 프로토 타입 메소드 `constructor, static methods, prototype methods`

이제 당신은 종종 클래스 정의에서 찾을 방법의 3 종류 살펴 보자. `Let’s examine three kinds of methods that you often find in class definitions.`

```javascript
class Foo {
    constructor(prop) {
        this.prop = prop;
    }
    static staticMethod() {
        return 'classy';
    }
    prototypeMethod() {
        return 'prototypical';
    }
}
const foo = new Foo(123);
```

다음과 같이 클래스 선언에 대한 객체도 보인다. 그것을 이해하기위한 팁 : 프로토 타입 값이 목적은 일반 속성이있는 동안 [[Prototype]], 객체 사이의 상속 관계이다. 새로운 운영자가 자신이 생성 인스턴스의 프로토 타입으로 값을 사용하기 때문에 속성 프로토 타입은 특별하다. `The object diagram for this class declaration looks as follows. Tip for understanding it: [[Prototype]] is an inheritance relationship between objects, while prototype is a normal property whose value is an object. The property prototype is only special because the new operator uses its value as the prototype for instances it creates.`

![classes----methods.jpg](images/classes----methods.jpg)

첫째, 의사 메소드 생성자입니다. 이 클래스를 나타내는 함수를 정의하는이 방법은 특별하다 : `First, the pseudo-method constructor. This method is special, as it defines the function that represents the class:`

```javascript
> Foo === Foo.prototype.constructor
true
> typeof Foo
'function'
```

그것은 때로는 클래스 생성자라고합니다. 정상 생성자 함수가없는 기능 (나중에 설명 슈퍼 통해 자사의 슈퍼 생성자 () - 전화 생성자에 주로 능력)을가집니다. `It is sometimes called a class constructor. It has features that normal constructor functions don’t have (mainly the ability to constructor-call its superconstructor via super(), which is explained later).`

둘째, 정적 방법. 정적 특성 (또는 클래스 속성은) Foo 자체의 속성입니다. 정적와 메소드 정의 앞에 경우에, 당신은 클래스 메소드를 만듭니다 `Second, static methods. Static properties (or class properties) are properties of Foo itself. If you prefix a method definition with static, you create a class method:`

```javascript
> typeof Foo.staticMethod
'function'
> Foo.staticMethod()
'classy'
```

셋째, 프로토 방법. Foo의 프로토 타입 속성 Foo.prototype의 재산입니다. 그들은 일반적으로 방법과 Foo의 인스턴스에 의해 상속됩니다. `Third, prototype methods. The prototype properties of Foo are the properties of Foo.prototype. They are usually methods and inherited by instances of Foo.`

```javascript
> typeof Foo.prototype.prototypeMethod
'function'
> foo.prototypeMethod()
'prototypical'
```

#### 15.2.2.2 정적 데이터 등록 `Static data properties`

지금은 클래스는 당신이 정적 메서드가 아닌 정적 데이터 속성을 만들 수 있습니다. 그에 대한 해결 방법에는 두 가지가 있습니다. `For now, classes only let you create static methods, but not static data properties. There are two work-arounds for that.`

첫째, 당신은 수동으로 정적 속성을 추가 할 수 있습니다 : `First, you can manually add a static property:`

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
}
Point.ZERO = new Point(0, 0);
```

둘째, 정적 게터를 만들 수 있습니다 : `Second, you can create a static getter:`

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    static get ZERO() {
        return new Point(0, 0);
    }
}
```

두 경우 모두, 당신은 당신이 읽을 수있는 속성 Point.ZERO을 찾으실 수 있습니다. 전자의 경우, 당신은 읽기 전용 속성을 만들 Object.defineProperty ()를 사용할 수 있습니다,하지만 난 과제의 단순함을 좋아한다. `In both cases, you get a property Point.ZERO that you can read. In the former case, you could use Object.defineProperty() to create a read-only property, but I like the simplicity of an assignment.`

#### 15.2.2.3 게터와 세터 `Getters and setters`

getter와 setter의 구문은 단지 인 ECMAScript 5 객체 리터럴에서 같다 : `The syntax for getters and setters is just like in ECMAScript 5 object literals:`

```javascript
class MyClass {
    get prop() {
        return 'getter';
    }
    set prop(value) {
        console.log('setter: '+value);
    }
}
```

다음과 같이 MyClass를 사용합니다. `You use MyClass as follows.`

```javascript
> const inst = new MyClass();
> inst.prop = 123;
setter: 123
> inst.prop
'getter'
```

#### 15.2.2.4 계산 메소드 이름 `Computed method names`

당신이 괄호에 넣어 경우, 식을 통해 메소드의 이름을 정의 할 수 있습니다. 예를 들어, Foo를 정의하는 다음과 같은 방법으로 모두 동일합니다. `You can define the name of a method via an expression, if you put it in square brackets. For example, the following ways of defining Foo are all equivalent.`

```javascript
class Foo() {
    myMethod() {}
}

class Foo() {
    ['my'+'Method']() {}
}

const m = 'myMethod';
class Foo() {
    [m]() {}
}
```

ECMAScript를 6에서 몇 가지 특별한 방법이 상징 키를 가지고있다. 계산 방법 이름은 당신이 그런 방법을 정의 할 수 있습니다. 객체가 그 키 Symbol.iterator 인 방법을 가지고 예를 들어, 그것은 반복 가능하다. 즉, 그 내용은 for에 대한 루프 및 기타 언어 메커니즘에 의해 이상 반복 할 수 있다는 것을 의미한다. `Several special methods in ECMAScript 6 have keys that are symbols. Computed method names allow you to define such methods. For example, if an object has a method whose key is Symbol.iterator, it is iterable. That means that its contents can be iterated over by the for-of loop and other language mechanisms.`

```javascript
class IterableClass {
    [Symbol.iterator]() {
        ···
    }
}
```

#### 15.2.2.5 제너레이터 메소드 `Generator methods`

별표 (\*)에있어서 정의 접두사 경우 발생 방법이된다. 무엇보다도, 제너레이터는 그 키 Symbol.iterator있는 방법을 정의하는 데 유용합니다. 그 작동 방법 다음 코드는 보여줍니다. `If you prefix a method definition with an asterisk (*), it becomes a generator method. Among other things, a generator is useful for defining the method whose key is Symbol.iterator. The following code demonstrates how that works.`

```javascript
class IterableArguments {
    constructor(...args) {
        this.args = args;
    }
    * [Symbol.iterator]() {
        for (const arg of this.args) {
            yield arg;
        }
    }
}

for (const x of new IterableArguments('hello', 'world')) {
    console.log(x);
}

// Output:
// hello
// world
```

### 15.2.3 서브클래싱 `Subclassing`

절은 당신이 (또는 클래스를 통해 정의되지 않았을 수도 있습니다) 기존의 생성자의 서브 클래스를 만들 수 확장 : `The extends clause lets you create a subclass of an existing constructor (which may or may not have been defined via a class):`

```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `(${this.x}, ${this.y})`;
    }
}

class ColorPoint extends Point {
    constructor(x, y, color) {
        super(x, y); // (A)
        this.color = color;
    }
    toString() {
        return super.toString() + ' in ' + this.color; // (B)
    }
}
```

당신이 기대하는 것처럼 또 다시,이 클래스가 사용됩니다 : `Again, this class is used like you’d expect:`

```javascript
> const cp = new ColorPoint(25, 8, 'green');
> cp.toString()
'(25, 8) in green'

> cp instanceof ColorPoint
true
> cp instanceof Point
true
```

클래스의 두 가지 종류가 있습니다 : `There are two kinds of classes:`

- 이 절을 확장하지 않기 때문에 Point, 기본 클래스입니다. `Point is a base class, because it doesn’t have an extends clause.`
- ColorPoint는 파생 클래스입니다. `ColorPoint is a derived class.`

super를 사용하는 두 가지 방법이 있습니다 : `There are two ways of using super:`

- 클래스 생성자 (클래스 정의에 유사 방법 생성자) 함수 호출처럼 사용 (super (···)), 슈퍼 생성자 호출 (라인 A)를 확인하기 위해. `A class constructor (the pseudo-method constructor in a class definition) uses it like a function call (super(···)), in order to make a superconstructor call (line A).`
- (또는 정적없이 객체 리터럴 또는 클래스에서) 메소드의 정의는 super 특성 (라인 B)를 참조하기 위해, 속성 참조 (super.prop) 또는 메서드 호출 (super.method (···))처럼 사용합니다. `Method definitions (in object literals or classes, with or without static) use it like property references (super.prop) or method calls (super.method(···)), in order to refer to superproperties (line B).`

#### 15.2.3.1 서브 클래스의 프로토 타입은 슈퍼 클래스입니다 `The prototype of a subclass is the superclass`

서브 클래스의 프로토 타입은 ECMAScript를 6 슈퍼 클래스입니다 : `The prototype of a subclass is the superclass in ECMAScript 6:`

```javascript
> Object.getPrototypeOf(ColorPoint) === Point
true
```

즉, 정적 속성이 상속된다는 것을 의미한다 : `That means that static properties are inherited:`

```javascript
class Foo {
    static classMethod() {
        return 'hello';
    }
}

class Bar extends Foo {
}
Bar.classMethod(); // 'hello'
```

정적 메소드를 super 호출 할 수 있습니다 : `You can even super-call static methods:`

```javascript
class Foo {
    static classMethod() {
        return 'hello';
    }
}

class Bar extends Foo {
    static classMethod() {
        return super.classMethod() + ', too';
    }
}
Bar.classMethod(); // 'hello, too'
```

#### 15.2.3.2 슈퍼 생성자 호출 `Superconstructor calls`

사용하기 전에 파생 클래스에서는 super ()를 호출해야합니다 : `In a derived class, you must call super() before you can use this:`

```javascript
class Foo {}

class Bar extends Foo {
    constructor(num) {
        const tmp = num * 2; // OK
        this.num = num; // ReferenceError
        super();
        this.num = num; // OK
    }
}
```

암시 적 super()는 에러가 발생를 호출하지 않고 파생 생성자를 떠나 : `Implicitly leaving a derived constructor without calling super() also causes an error:`

```javascript
class Foo {}

class Bar extends Foo {
    constructor() {
    }
}

const bar = new Bar(); // ReferenceError
```

#### 15.2.3.3 생성자의 결과를 재정의 `Overriding the result of a constructor`

그냥 ES5처럼, 당신은 명시 적으로 객체를 반환하여 생성자의 결과를 재정의 할 수 있습니다 : `Just like in ES5, you can override the result of a constructor by explicitly returning an object:`

```javascript
class Foo {
    constructor() {
        return Object.create(null);
    }
}
console.log(new Foo() instanceof Foo); // false
```

이렇게하면,이 초기화되었는지 아닌지 상관 없다. 즉 : 이 방식으로 결과를 무시할 경우 파생 생성자 () super를 호출 할 필요가 없습니다. `If you do so, it doesn’t matter whether this has been initialized or not. In other words: you don’t have to call super() in a derived constructor if you override the result in this manner.`

#### 15.2.3.4 클래스의 기본 생성자 `Default constructors for classes`

기본 클래스의 생성자를 지정하지 않을 경우, 다음과 같은 정의가 사용된다 : `If you don’t specify a constructor for a base class, the following definition is used:`

```javascript
constructor() {}
```

파생 클래스의 경우, 다음과 같은 기본 생성자가 사용된다 : `For derived classes, the following default constructor is used:`

```javascript
constructor(...args) {
    super(...args);
}
```

#### 15.2.3.5 기본 생성자를 서브 클래싱 `Subclassing built-in constructors`

ECMAScript를 6에서, 마지막으로 모든 내장 생성자 (가 ES5에 대한 해결 방법이 있지만,이 상당한 제한이 있습니다)를 서브 클래 싱 할 수 있습니다. `In ECMAScript 6, you can finally subclass all built-in constructors (there are work-arounds for ES5, but these have significant limitations).`

예를 들어, 지금 자신의 예외 클래스 (즉, 대부분의 엔진에 스택 트레이스를 갖는 기능을 상속합니다) 만들 수 있습니다 : `For example, you can now create your own exception classes (that will inherit the feature of having a stack trace in most engines):`

```javascript
class MyError extends Error {
}
throw new MyError('Something happened!');
```

또한 인스턴스 제대로 길이를 처리 할 배열의 하위 클래스를 만들 수 있습니다 : `You can also create subclasses of Array whose instances properly handle length:`

```javascript
class MyArray extends Array {
    constructor(len) {
        super(len);
    }
}

// Instances of of `MyArray` work like real Arrays:
const myArr = new MyArray(0);
console.log(myArr.length); // 0
myArr[0] = 'foo';
console.log(myArr.length); // 1
```

엔진이 기본적으로 지원해야 뭔가가 서브 클래스 기본 생성자를 참고, 당신은 transpilers를 통해이 기능을받지 않습니다. `Note that subclassing built-in constructors is something that engines have to support natively, you won’t get this feature via transpilers.`

## 15.3 클래스의 Private 데이터 `Private data for classes`

이 섹션에서는 ES6 클래스에 대한 private 정보를 관리하기위한 네 가지 방법을 설명합니다 : `This section explains four approaches for managing private data for ES6 classes:`

1. 클래스 생성자의 환경에서 private 정보를 유지 `Keeping private data in the environment of a class constructor`
2. 이름 지정 규칙 (예를 들어, 접두사 밑줄)를 통해 private 특성을 표시 `Marking private properties via a naming convention (e.g. a prefixed underscore)`
3. WeakMaps에 private 정보를 유지 `Keeping private data in WeakMaps`
4. private 속성에 대한 키로 기호를 사용하여 `Using symbols as keys for private properties`

접근 방법 \#1과 \#2는 생성자를 들어, ES5 이미 일반적이었다. \#3, \#4 ES6의 새로운 접근. 의는 접근 각을 통해, 같은 예를 네 번을 구현할 수 있습니다. `Approaches #1 and #2 were already common in ES5, for constructors. Approaches #3 and #4 are new in ES6. Let’s implement the same example four times, via each of the approaches.`

### 15.3.1 생성자 환경을 통해 private 데이터 `Private data via constructor environments`

실행 예는 (초기 값으로 카운터)는 0에 도달 한 번 카운터 콜백 동작을 호출하는 클래스 카운트이다. 두 개의 매개 변수 작업 및 카운터는 private 데이터로 저장해야합니다. `Our running example is a class Countdown that invokes a callback action once a counter (whose initial value is counter) reaches zero. The two parameters action and counter should be stored as private data.`

첫 번째 구현에서, 우리는 클래스 생성자의 환경에서 작업하고 카운터를 저장합니다. 환경은 자바 스크립트 엔진은 새로운 범위가 (예를 들어, 함수 호출이나 생성자 호출을 통해)에 입력 될 때마다 존재하게 매개 변수 및 로컬 변수들을 저장하는 내부 데이터 구조이다. 이 코드입니다 : `In the first implementation, we store action and counter in the environment of the class constructor. An environment is the internal data structure, in which a JavaScript engine stores the parameters and local variables that come into existence whenever a new scope is entered (e.g. via a function call or a constructor call). This is the code:`

```javascript
class Countdown {
    constructor(counter, action) {
        Object.assign(this, {
            dec() {
                if (counter < 1) return;
                counter--;
                if (counter === 0) {
                    action();
                }
            }
        });
    }
}
```

Countdown을 사용하면 다음과 같다 : `Using Countdown looks like this:`

```javascript
> const c = new Countdown(2, () => console.log('DONE'));
> c.dec();
> c.dec();
DONE
```

장점 : `Pros:`

- private 정보는 완전히 안전하다 `The private data is completely safe`
- private 속성의 이름은 (슈퍼 클래스와 서브 클래스의) 다른 private 속성의 이름과 충돌하지 않습니다. `The names of private properties won’t clash with the names of other private properties (of superclasses or subclasses).`

단점 : `Cons:`

- 생성자 내부에서, 인스턴스 (private 데이터에 액세스해야 적어도 그 방법을) 모든 방법을 추가 할 필요가 있기 때문에 코드는 덜 우아한된다. `The code becomes less elegant, because you need to add all methods to the instance, inside the constructor (at least those methods that need access to the private data).`
- 인해 인스턴스 메소드에, 코드는 메모리를 낭비한다. 방법은 프로토 타입 방법이었다, 그들은 공유 할 것입니다. `Due to the instance methods, the code wastes memory. If the methods were prototype methods, they would be shared.`

이 기술에 대한 추가 정보 : 분파. "자바 스크립트를 말하기"에서 "생성자의 환경 (Crockford에 private 정보 보호 패턴)의 private 정보". `More information on this technique: Sect. “Private Data in the Environment of a Constructor (Crockford Privacy Pattern)” in “Speaking JavaScript”.`

### 15.3.2 이름 지정 규칙을 통해 private 데이터 `Private data via a naming convention`

다음 코드는 이름을 접두어로 밑줄로 표시 속성에서 private 데이터를 유지합니다 : `The following code keeps private data in properties whose names a marked via a prefixed underscore:`

```javascript
class Countdown {
    constructor(counter, action) {
        this._counter = counter;
        this._action = action;
    }
    dec() {
        if (this._counter < 1) return;
        this._counter--;
        if (this._counter === 0) {
            this._action();
        }
    }
}
```

장점 : `Pros:`

- 코드가 좋아 보인다. `Code looks nice.`
- 프로토 타입 방법을 사용할 수 있습니다. `We can use prototype methods.`

단점 : `Cons:`

- 안전하지 않다, 클라이언트 코드의 가이드라인일 뿐. `Not safe, only a guideline for client code.`
- private 속성의 이름이 충돌 할 수 있습니다. `The names of private properties can clash.`

### 15.3.3 WeakMaps를 통해 private 데이터 `Private data via WeakMaps`

(프로토 타입 메소드를 사용 수있는) 두 번째 접근법의 이점 첫 번째 방법 (안전성)의 장점을 결합한 WeakMap 관련된 깔끔한 기술이있다. 이 기술은 다음 코드에서 설명한다 : 우리는 private 데이터를 저장하기 위해 WeakMap _counter 및 _action를 사용합니다. `There is a neat technique involving WeakMaps that combines the advantage of the first approach (safety) with the advantage of the second approach (being able to use prototype methods). This technique is demonstrated in the following code: we use the WeakMaps _counter and _action to store private data.`

```javascript
const _counter = new WeakMap();
const _action = new WeakMap();
class Countdown {
    constructor(counter, action) {
        _counter.set(this, counter);
        _action.set(this, action);
    }
    dec() {
        let counter = _counter.get(this);
        if (counter < 1) return;
        counter--;
        _counter.set(this, counter);
        if (counter === 0) {
            _action.get(this)();
        }
    }
}
```

자신의 private 정보에 대한 두 WeakMap _counter 및 _action WeakMap 객체의 각. 방법에 의한 것은 가비지 수집에서 개체를 방지 할 수 없습니다 작업을 WeakMaps. 만큼 외부 세계에서 숨겨진 WeakMaps을 유지으로, private 데이터는 안전합니다. 안전 할하려는 경우, 임시 변수에 WeakMap.prototype.get 및 WeakMap.prototype.set를 저장하고 (동적 대신 방법의) 호출합니다. 악성 코드는 private 데이터 스누핑 그 방법을 대체하는 경우 그리고 코드는 영향을받지 않습니다. 그러나, 코드 뒤에 실행되는 코드에 대해 보호됩니다. 그것은 전에 실행하는 경우 할 수있는 것은 아무것도 없습니다. `Each of the two WeakMaps _counter and _action maps objects to their private data. Due to how WeakMaps work that won’t prevent objects from being garbage-collected. As long as you keep the WeakMaps hidden from the outside world, the private data is safe. If you want to be even safer, you can store WeakMap.prototype.get and WeakMap.prototype.set in temporary variables and invoke those (instead of the methods, dynamically). Then our code wouldn’t be affected if malicious code replaced those methods with ones that snoop on our private data. However, we are only protected against code that runs after our code. There is nothing we can do if it runs before ours.`

장점 : `Pros:`

- 프로토 타입 메소드를 사용할 수 있습니다. `We can use prototype methods.`
- 속성 키에 대한 명명 규칙을보다 안전. `Safer than a naming convention for property keys.`
- private 속성의 이름은 충돌 할 수 없습니다. `The names of private properties can’t clash.`
- 상대적으로 우아한. `Relatively elegant.`

단점 : `Con:`

- 코드 명명 규칙처럼 우아하지 않다. `Code is not as elegant as a naming convention.`

### 15.3.4 Symbol을 통해 private 데이터 `Private data via symbols`

private 정보에 대한 또 다른 저장 위치는 그 키 Symbol 프로퍼티입니다 : `Another storage location for private data are properties whose keys are symbols:`

```javascript
const _counter = Symbol('counter');
const _action = Symbol('action');

class Countdown {
    constructor(counter, action) {
        this[_counter] = counter;
        this[_action] = action;
    }
    dec() {
        if (this[_counter] < 1) return;
        this[_counter]--;
        if (this[_counter] === 0) {
            this[_action]();
        }
    }
}
```

각 Symbol은 기호 값 속성 키는 다른 속성 키와 충돌하지 않습니다 이유입니다, 유니크합니다. 또한, Symbol은 약간 바깥 세상에서 숨겨진 있지만 완전히 있습니다 : `Each symbol is unique, which is why a symbol-valued property key will never clash with any other property key. Additionally, symbols are somewhat hidden from the outside world, but not completely:`

```javascript
const c = new Countdown(2, () => console.log('DONE'));

console.log(Object.keys(c));
    // []
console.log(Reflect.ownKeys(c));
    // [ Symbol(counter), Symbol(action) ]
```

장점 : `Pros:`

- 프로토 타입 메서드를 사용할 수 있습니다. `We can use prototype methods.`
- private 속성의 이름은 충돌 할 수 없습니다. `The names of private properties can’t clash.`

단점 : `Cons:`

- 코드 명명 규칙처럼 우아하지 않다. `Code is not as elegant as a naming convention.`
- 안전하지 않다. : (Symbol을 포함!) Reflection.own() 키를 통해 개체의 모든 속성 키를 나열 할 수 있습니다. `Not safe: you can list all property keys (including symbols!) of an object via Reflect.ownKeys().`

### 15.3.5 더 읽기 `Further reading`

- 분파. "말하기 자바 스크립트"의 "데이터 private 유지"(ES5 기술을 포함) `Sect. “Keeping Data Private” in “Speaking JavaScript” (covers ES5 techniques)`

## 15.4 간단한 mixins `Simple mixins`

자바 스크립트에서 서브 클래 싱은 두 가지 이유로 사용됩니다 : `Subclassing in JavaScript is used for two reasons:`

- 인터페이스 상속 (instanceof를하여 시험 한대로) 서브 클래스의 인스턴스 인 모든 객체는 슈퍼 클래스의 인스턴스입니다. 기대는 서브 클래스의 인스턴스가 슈퍼 클래스 인스턴스처럼 행동하지만, 더 많은 일을 할 수 있다는 것이다. `Interface inheritance: Every object that is an instance of a subclass (as tested by instanceof) is also an instance of the superclass. The expectation is that subclass instances behave like superclass instances, but may do more.`
- 구현 상속 : 상위 클래스 자신의 서브 클래스에 기능에 전달합니다. `Implementation inheritance: Superclasses pass on functionality to their subclasses.`

그들은 단지 하나의 상속을 지원하기 때문에 구현 상속 클래스의 유용성은 (클래스가 최대 하나의 수퍼 클래스를 가질 수있다), 제한된다. 따라서, 여러 소스에서 공구 메소드를 상속하는 것은 불가능하다 - 모두 상위 클래스에서 와야합니다. `The usefulness of classes for implementation inheritance is limited, because they only support single inheritance (a class can have at most one superclass). Therefore, it is impossible to inherit tool methods from multiple sources – they must all come from the superclass.`

그래서 어떻게 이 문제를 해결할 수 있습니까? 예제를 통해 해결책을 알아 보자. 직원이 사람의 서브 클래스 기업에 대한 관리 시스템을 생각 해보자. `So how can we solve this problem? Let’s explore a solution via an example. Consider a management system for an enterprise where Employee is a subclass of Person.`

```javascript
class Person { ··· }
class Employee extends Person { ··· }
```

또한, 스토리지 및 데이터 검증을위한 도구 클래스가 있습니다 : `Additionally, there are tool classes for storage and for data validation:`

```javascript
class Storage {
    save(database) { ··· }
}
class Validation {
    validate(schema) { ··· }
}
```

우리는이 같은 도구 클래스를 포함 할 수 있다면 그것은 좋은 것입니다 : `It would be nice if we could include the tool classes like this:`

```javascript
// Invented ES6 syntax:
class Employee extends Storage, Validation, Person { ··· }
```

그것은 우리가 직원이 사람의 서브 클래스해야 검증의 서브 클래스해야 스토리지의 서브 클래스로 할 것이다. 직원과 사람은 클래스의 하나의 체인에 사용됩니다. 그러나 저장 및 검증은 여러 번 사용할 수 있습니다. 우리는 그들이 super 우리가 작성 클래스 템플릿되고 싶어요. 이러한 템플릿 추상 서브 클래스 또는 유지 mixin라고합니다. `That is, we want Employee to be a subclass of Storage which should be a subclass of Validation which should be a subclass of Person. Employee and Person will only be used in one such chain of classes. But Storage and Validation will be used multiple times. We want them to be templates for classes whose superclasses we fill in. Such templates are called abstract subclasses or mixins.`

ES6의 mixin을 구현하는 한 가지 방법은 그 입력 super 클래스이며, 출력이 그 super 클래스를 확장하는 서브 클래스의 함수로 볼 수 있습니다 : `One way of implementing a mixin in ES6 is to view it as a function whose input is a superclass and whose output is a subclass extending that superclass:`

```javascript
const Storage = Sup => class extends Sup {
    save(database) { ··· }
};
const Validation = Sup => class extends Sup {
    validate(schema) { ··· }
};
```

고정 된 식별자하지만, 임의의 표현되지 않는 확장의 여기, 우리는 피연산자에서 이익. 이러한 유지 mixin으로, 직원은 다음과 같이 생성된다 : `Here, we profit from the operand of the extends clause not being a fixed identifier, but an arbitrary expression. With these mixins, Employee is created like this:`

```javascript
class Employee extends Storage(Validation(Person)) { ··· }
```

승인. 제가 알고이 기술의 첫 번째 항목은 [a Gist by Sebastian Markbåge](https://gist.github.com/sebmarkbage/fac0830dbb13ccbff596) 이다 `Acknowledgement. The first occurrence of this technique that I’m aware of is a Gist by Sebastian Markbåge.`

## 15.5 클래스의 세부 사항 `The details of classes`

지금까지 본 것은 클래스의 필수 요소입니다. 당신이 후드 아래에 일이 관심이 있다면 읽어해야합니다. 의 클래스의 구문 시작하자. 다음은 [Sect. A.4 of the ECMAScript 6 specification.](http://www.ecma-international.org/ecma-262/6.0/#sec-functions-and-classes)의 약간 수정 된 버전입니다.  'What we have seen so far are the essentials of classes. You only need to read on if you are interested how things happen under the hood. Let’s start with the syntax of classes. The following is a slightly modified version of the syntax shown in [Sect. A.4 of the ECMAScript 6 specification.](http://www.ecma-international.org/ecma-262/6.0/#sec-functions-and-classes)'

```javascript
ClassDeclaration:
    "class" BindingIdentifier ClassTail
ClassExpression:
    "class" BindingIdentifier? ClassTail

ClassTail:
    ClassHeritage? "{" ClassBody? "}"
ClassHeritage:
    "extends" AssignmentExpression
ClassBody:
    ClassElement+
ClassElement:
    MethodDefinition
    "static" MethodDefinition
    ";"

MethodDefinition:
    PropName "(" FormalParams ")" "{" FuncBody "}"
    "*" PropName "(" FormalParams ")" "{" GeneratorBody "}"
    "get" PropName "(" ")" "{" FuncBody "}"
    "set" PropName "(" PropSetParams ")" "{" FuncBody "}"

PropertyName:
    LiteralPropertyName
    ComputedPropertyName
LiteralPropertyName:
    IdentifierName  /* foo */
    StringLiteral   /* "foo" */
    NumericLiteral  /* 123.45, 0xFF */
ComputedPropertyName:
    "[" Expression "]"
```
    
Two observations:

- The value to be extended can be produced by an arbitrary expression. Which means that you’ll be able to write code such as the following:
  ```javascript
  class Foo extends combine(MyMixin, MySuperClass) {}
  ```  
- Semicolons are allowed between methods.

### 15.5.1 Various checks

- Error checks: the class name cannot be eval or arguments; duplicate class element names are not allowed; the name constructor can only be used for a normal method, not for a getter, a setter or a generator method.
- Classes can’t be function-called. They throw a TypeException if they are.
- Prototype methods cannot be used as constructors:
  ```javascript
  class C {
      m() {}
  }
  new C.prototype.m(); // TypeError
  ```
  
### 15.5.2 Attributes of properties
Class declarations create (mutable) let bindings. For a given class Foo:

- Static methods Foo.* are writable and configurable, but not enumerable. Making them writable allows for dynamic patching.
- A constructor and the object in its property prototype have an immutable link:
-- Foo.prototype is non-writable, non-enumerable, non-configurable.
-- Foo.prototype.constructor is non-writable, non-enumerable, non-configurable.
- Prototype methods Foo.prototype.* are writable and configurable, but not enumerable.

Note that method definitions in object literals produce enumerable properties.

### 15.5.3 Classes have inner names
Classes have lexical inner names, just like named function expressions.

#### 15.5.3.1 The inner names of named function expressions
You may know that named function expressions have lexical inner names:

```javascript
const fac = function me(n) {
    if (n > 0) {
        // Use inner name `me` to
        // refer to function
        return n * me(n-1);
    } else {
        return 1;
    }
};
console.log(fac(3)); // 6
```

The name me of the named function expression becomes a lexically bound variable that is unaffected by which variable currently holds the function.

#### 15.5.3.2 The inner names of classes
Interestingly, ES6 classes also have lexical inner names that you can use in methods (constructor methods and regular methods):

```javascript
class C {
    constructor() {
        // Use inner name C to refer to class
        console.log(`constructor: ${C.prop}`);
    }
    logProp() {
        // Use inner name C to refer to class
        console.log(`logProp: ${C.prop}`);
    }
}
C.prop = 'Hi!';

const D = C;
C = null;

// C is not a class, anymore:
new C().logProp();
    // TypeError: C is not a function

// But inside the class, the identifier C
// still works
new D().logProp();
    // constructor: Hi!
    // logProp: Hi!
```

(In the ES6 spec the inner name is set up by [the dynamic semantics of ClassDefinitionEvaluation.](http://www.ecma-international.org/ecma-262/6.0/#sec-runtime-semantics-classdefinitionevaluation))

Acknowledgement: Thanks to Michael Ficarra for pointing out that classes have inner names.

## 15.6 The details of subclassing
In ECMAScript 6, subclassing looks as follows.

```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
    toString() {
        return `Person named ${this.name}`;
    }
    static logNames(persons) {
        for (const person of persons) {
            console.log(person.name);
        }
    }
}

class Employee extends Person {
    constructor(name, title) {
        super(name);
        this.title = title;
    }
    toString() {
        return `${super.toString()} (${this.title})`;
    }
}

const jane = new Employee('Jane', 'CTO');
console.log(jane.toString()); // Person named Jane (CTO)
```

The next section examines the structure of the objects that were created by the previous example. The section after that examines how jane is allocated and initialized.

### 15.6.1 Prototype chains
The previous example creates the following objects.

![classes----subclassing_es6.jpg](images/classes----subclassing_es6.jpg)

Prototype chains are objects linked via the [[Prototype]] relationship (which is an inheritance relationship). In the diagram, you can see two prototype chains:

#### 15.6.1.1 Left column: classes (functions)
The prototype of a derived class is the class it extends. The reason for this setup is that you want a subclass to inherit all properties of its superclass:

```javascript
> Employee.logNames === Person.logNames
true
```

The prototype of a base class is Function.prototype, which is also the prototype of functions:

```javascript
> const getProto = Object.getPrototypeOf.bind(Object);

> getProto(Person) === Function.prototype
true
> getProto(function () {}) === Function.prototype
true
```

That means that base classes and all their derived classes (their prototypees) are functions. Traditional ES5 functions are essentially base classes.

#### 15.6.1.2 Right column: the prototype chain of the instance
The main purpose of a class is to set up this prototype chain. The prototype chain ends with Object.prototype (whose prototype is null). That makes Object an implicit superclass of every base class (as far as instances and the instanceof operator are concerned).

The reason for this setup is that you want the instance prototype of a subclass to inherit all properties of the superclass instance prototype.

As an aside, objects created via object literals also have the prototype Object.prototype:

```javascript
> Object.getPrototypeOf({}) === Object.prototype
true
```

### 15.6.2 Allocating and initializing instances
The data flow between class constructors is different from the canonical way of subclassing in ES5. Under the hood, it roughly looks as follows.

```javascript
// Base class: this is where the instance is allocated
function Person(name) {
    // Performed before entering this constructor:
    this = Object.create(new.target.prototype);

    this.name = name;
}
···

function Employee(name, title) {
    // Performed before entering this constructor:
    this = uninitialized;

    this = Reflect.construct(Person, [name], new.target); // (A)
        // super(name);

    this.title = title;
}
Object.setPrototypeOf(Employee, Person);
···

const jane = Reflect.construct( // (B)
             Employee, ['Jane', 'CTO'],
             Employee);
    // const jane = new Employee('Jane', 'CTO')
```

The instance object is created in different locations in ES6 and ES5:

- In ES6, it is created in the base constructor, the last in a chain of constructor calls. The superconstructor is invoked via super(), which triggers a constructor call.
- In ES5, it is created in the operand of new, the first in a chain of constructor calls. The superconstructor is invoked via a function call.

The previous code uses two new ES6 features:

- new.target is an implicit parameter that all functions have. In a chain of constructor calls, its role is similar to this in a chain of supermethod calls.
If a constructor is directly invoked via new (as in line B), the value of new.target is that constructor.
If a constructor is called via super() (as in line A), the value of new.target is the new.target of the constructor that makes the call.
During a normal function call, it is undefined. That means that you can use new.target to determine whether a function was function-called or constructor-called (via new).
Inside an arrow function, new.target refers to the new.target of the surrounding non-arrow function.
- Reflect.construct() lets you make constructor calls while specifying new.target via the last parameter.

The advantage of this way of subclassing is that it enables normal code to subclass built-in constructors (such as Error and Array). A later section explains why a different approach was necessary.

As a reminder, here is how you do subclassing in ES5:

```javascript
function Person(name) {
    this.name = name;
}
···

function Employee(name, title) {
    Person.call(this, name);
    this.title = title;
}
Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;
···
```

#### 15.6.2.1 Safety checks

- this originally being uninitialized in derived constructors means that an error is thrown if they access this in any way before they have called super().
- Once this is initialized, calling super() produces a ReferenceError. This protects you against calling super() twice.
- If a constructor returns implicitly (without a return statement), the result is this. If this is uninitialized, a ReferenceError is thrown. This protects you against forgetting to call super().
- If a constructor explicitly returns a non-object (including undefined and null), the result is this (this behavior is required to remain compatible with ES5 and earlier). If this is uninitialized, a TypeError is thrown.
- If a constructor explicitly returns an object, it is used as its result. Then it doesn’t matter whether this is initialized or not.

#### 15.6.2.2 The extends clause
Let’s examine how the extends clause influences how a class is set up (Sect. 14.5.14 of the spec).

The value of an extends clause must be “constructible” (invocable via new). null is allowed, though.

```javascript
class C {
}
```

- Constructor kind: base
- Prototype of C: Function.prototype (like a normal function)
- Prototype of C.prototype: Object.prototype (which is also the prototype of objects created via object literals)

```javascript
class C extends B {
}
```

- Constructor kind: derived
- Prototype of C: B
- Prototype of C.prototype: B.prototype

```javascript
class C extends Object {
}
```

- Constructor kind: derived
- Prototype of C: Object
- Prototype of C.prototype: Object.prototype

Note the following subtle difference with the first case: If there is no extends clause, the class is a base class and allocates instances. If a class extends Object, it is a derived class and Object allocates the instances. The resulting instances (including their prototype chains) are the same, but you get there differently.

class C extends null {
}
Constructor kind: derived
Prototype of C: Function.prototype
Prototype of C.prototype: null
Such a class lets you avoid Object.prototype in the prototype chain. But that is rarely useful. Furthermore, you have to be careful: new-calling such a class leads to an error, because the default constructor makes a superconstructor call and Function.prototype (the superconstructor) can’t be constructor-called. The only way to make the error go away is by adding a constructor that returns an object:

```javascript
class C extends null {
    constructor() {
        const _this = Object.create(new.target.prototype);
        return _this;
    }
}
```

new.target ensures that C can be subclassed properly – the prototype of _this will always be the operand of new.

### 15.6.3 Why can’t you subclass built-in constructors in ES5?
In ECMAScript 5, most built-in constructors can’t be subclassed ([several work-arounds exist](http://speakingjs.com/es5/ch28.html)).

To understand why, let’s use the canonical ES5 pattern to subclass Array. As we shall soon find out, this doesn’t work.

```javascript
function MyArray(len) {
    Array.call(this, len); // (A)
}
MyArray.prototype = Object.create(Array.prototype);
```

Unfortunately, if we instantiate MyArray, we find out that it doesn’t work properly: The instance property length does not change in reaction to us adding Array elements:

```javascript
> var myArr = new MyArray(0);
> myArr.length
0
> myArr[0] = 'foo';
> myArr.length
0
```

There are two obstracles that prevent myArr from being a proper Array.

First obstacle: initialization. The this you hand to the constructor Array (in line A) is completely ignored. That means you can’t use Array to set up the instance that was created for MyArray.

```javascript
> var a = [];
> var b = Array.call(a, 3);
> a !== b  // a is ignored, b is a new object
true
> b.length // set up correctly
3
> a.length // unchanged
0
```

Second obstacle: allocation. The instance objects created by Array are exotic (a term used by the ECMAScript specification for objects that have features that normal objects don’t have): Their property length tracks and influences the management of Array elements. In general, exotic objects can be created from scratch, but you can’t convert an existing normal object into an exotic one. Unfortunately, that is what Array would have to do, when called in line A: It would have to turn the normal object created for MyArray into an exotic Array object.

#### 15.6.3.1 The solution: ES6 subclassing
In ECMAScript 6, subclassing Array looks as follows:

```javascript
class MyArray extends Array {
    constructor(len) {
        super(len);
    }
}
```

This works:

```javascript
> const myArr = new MyArray(0);
> myArr.length
0
> myArr[0] = 'foo';
> myArr.length
1
```

Let’s examine how the ES6 approach to subclassing removes the previously mentioned obstacles:

- The first obstacle, Array not being able to set up an instance, is removed by Array returning a fully configured instance. In contrast to ES5, this instance has the prototype of the subclass.
- The second obstacle, subconstructors not creating exotic instances, is removed by derived classes relying on base classes for allocating instances.

### 15.6.4 Referring to superproperties in methods
The following ES6 code makes a supermethod call in line B.

```javascript
class Person {
    constructor(name) {
        this.name = name;
    }
    toString() { // (A)
        return `Person named ${this.name}`;
    }
}

class Employee extends Person {
    constructor(name, title) {
        super(name);
        this.title = title;
    }
    toString() {
        return `${super.toString()} (${this.title})`; // (B)
    }
}

const jane = new Employee('Jane', 'CTO');
console.log(jane.toString()); // Person named Jane (CTO)
```

To understand how super-calls work, let’s look at the object diagram of jane:

![classes----supercalls.jpg](images/classes----supercalls.jpg)

In line B, Employee.prototype.toString makes a super-call (line B) to the method (starting in line A) that it has overridden. Let’s call the object, in which a method is stored, the home object of that method. For example, Employee.prototype is the home object of Employee.prototype.toString().

The super-call in line B involves three steps:

* Start your search in the prototype of the home object of the current method.
* Look for a method whose name is toString. That method may be found in the object where the search started or later in the prototype chain.
* Call that method with the current this. The reason for doing so is: the super-called method must be able to access the same instance properties (in our example, the own properties of jane).

Note that even if you are only getting (super.prop) or setting (super.prop = 123) a superproperty (versus making a method call), this may still (internally) play a role in step #3, because a getter or a setter may be invoked.

Let’s express these steps in three different – but equivalent – ways:

```javascript
// Variation 1: supermethod calls in ES5
var result = Person.prototype.toString.call(this) // steps 1,2,3

// Variation 2: ES5, refactored
var superObject = Person.prototype; // step 1
var superMethod = superObject.toString; // step 2
var result = superMethod.call(this) // step 3

// Variation 3: ES6
var homeObject = Employee.prototype;
var superObject = Object.getPrototypeOf(homeObject); // step 1
var superMethod = superObject.toString; // step 2
var result = superMethod.call(this) // step 3
Variation 3 is how ECMAScript 6 handles super-calls. This approach is supported by two internal bindings that the environments of functions have (environments provide storage space, so-called bindings, for the variables in a scope):
```

- [[thisValue]]: This internal binding also exists in ECMAScript 5 and stores the value of this.
- [[HomeObject]]: Refers to the home object of the environment’s function. Filled in via an internal property [[HomeObject]] that all methods have that use super. Both the binding and the property are new in ECMAScript 6.

> Methods are a special kind of function now
> In a class, a method definition that uses super creates a special kind of function: It is still a function, but it has the internal property [[HomeObject]]. That property is set up by the method definition and can’t be changed in JavaScript. Therefore, you can’t meaningfully move such a method to a different object. (But maybe it’ll be possible in a future version of ECMAScript.)

####15.6.4.1 Where can you use super?
Referring to superproperties is handy whenever prototype chains are involved, which is why you can use it in method definitions (incl. generator method definitions, getters and setters) inside object literals and class definitions. The class can be derived or not, the method can be static or not.

Using super to refer to a property is not allowed in function declarations, function expressions and generator functions.

#### 15.6.4.2 A method that uses super can’t be moved
You can’t move a method that uses super: Such a method has an internal property [[HomeObject]] that ties it to the object it was created in. If you move it via an assignment, it will continue to refer to the superproperties of the original object. In future ECMAScript versions, there may be a way to transfer such a method, too.

## 15.7 The species pattern
One more mechanism of built-in constructors has been made extensible in ECMAScript 6: Sometimes a method creates new instances of its class. If you create a subclass – should the method return an instance of its class or an instance of the subclass? A few built-in ES6 methods let you configure how they create instances via the so-called species pattern.

As an example, consider a subclass SortedArray of Array. If we invoke map() on instances of that class, we want it to return instances of Array, to avoid unnecessary sorting. By default, map() returns instances of the receiver (this), but the species patterns lets you change that.

### 15.7.1 Helper methods for examples
In the following three sections, I’ll use two helper functions in the examples:

```javascript
function isObject(value) {
    return (value !== null
       && (typeof value === 'object'
           || typeof value === 'function'));
}

/**
 * Spec-internal operation that determines whether `x`
 * can be used as a constructor.
 */
function isConstructor(x) {
    ···
}
```

### 15.7.2 The standard species pattern
The standard species pattern is used by Promise.prototype.then(), the filter() method of Typed Arrays and other operations. It works as follows:

- If this.constructor[Symbol.species] exists, use it as a constructor for the new instance.
- Otherwise, use a default constructor (e.g. Array for Arrays).

Implemented in JavaScript, the pattern would look like this:

```javascript
function SpeciesConstructor(O, defaultConstructor) {
    const C = O.constructor;
    if (C === undefined) {
        return defaultConstructor;
    }
    if (! isObject(C)) {
        throw new TypeError();
    }
    const S = C[Symbol.species];
    if (S === undefined || S === null) {
        return defaultConstructor;
    }
    if (! isConstructor(S)) {
        throw new TypeError();
    }
    return S;
}
```

> The standard species pattern is implemented in the spec via the operation [SpeciesConstructor()](http://www.ecma-international.org/ecma-262/6.0/#sec-speciesconstructor).

### 15.7.3 The species pattern for Arrays
Normal Arrays implement the species pattern slightly differently:

```javascript
function ArraySpeciesCreate(self, length) {
    let C = undefined;
    // If the receiver `self` is an Array,
    // we use the species pattern
    if (Array.isArray(self)) {
        C = self.constructor;
        if (isObject(C)) {
            C = C[Symbol.species];
        }
    }
    // Either `self` is not an Array or the species
    // pattern didn’t work out:
    // create and return an Array
    if (C === undefined || C === null) {
        return new Array(length);
    }
    if (! IsConstructor(C)) {
        throw new TypeError();
    }
    return new C(length);
}
```

Array.prototype.map() creates the Array it returns via ArraySpeciesCreate(this, this.length).

> The species pattern for Arrays is implemented in the spec via the operation [ArraySpeciesCreate()](http://www.ecma-international.org/ecma-262/6.0/#sec-arrayspeciescreate).

### 15.7.4 The species pattern in static methods
Promises use a variant of the species pattern for static methods such as Promise.all():

```javascript
let C = this; // default
if (! isObject(C)) {
    throw new TypeError();
}
// The default can be overridden via the property `C[Symbol.species]`
const S = C[Symbol.species];
if (S !== undefined && S !== null) {
    C = S;
}
if (!IsConstructor(C)) {
    throw new TypeError();
}
const instance = new C(···);
```

### 15.7.5 Overriding the default species in subclasses
This is the default getter for the property [Symbol.species]:

```javascript
static get [Symbol.species]() {
    return this;
}
```

This default getter is implemented by the built-in classes Array, ArrayBuffer, Map, Promise, RegExp, Set and %TypedArray%. It is automatically inherited by subclasses of these built-in classes.

There are two ways in which you can override the default species: with a constructor of your choosing or with null.

#### 15.7.5.1 Setting the species to a constructor of your choosing
You can override the default species via a static getter (line A):

```javascript
class MyArray1 extends Array {
    static get [Symbol.species]() { // (A)
        return Array;
    }
}
```

As a result, map() returns an instance of Array:

```javascript
const result1 = new MyArray1().map(x => x);
console.log(result1 instanceof Array); // true
```

If you don’t override the default species, map() returns an instance of the subclass:

```javascript
class MyArray2 extends Array { }

const result2 = new MyArray2().map(x => x);
console.log(result2 instanceof MyArray2); // true
```

##### 15.7.5.1.1 Specifying the species via a data property
If you don’t want to use a static getter, you need to use Object.defineProperty(). You can’t use assignment, as there is already a property with that key that only has a getter. That means that it is read-only and can’t be assigned to.

For example, here we set the species of MyArray1 to Array:

```javascript
Object.defineProperty(
    MyArray1, Symbol.species, {
        value: Array
    });
````

#### 15.7.5.2 Setting the species to null
If you set the species to null then the default constructor is used (which one that is depends on which variant of the species pattern is used, consult the previous sections for more information).

```javascript
class MyArray3 extends Array {
    static get [Symbol.species]() {
        return null;
    }
}

const result3 = new MyArray3().map(x => x);
console.log(result3 instanceof Array); // true
```

## 15.8 The pros and cons of classes
Classes are controversial within the JavaScript community: On one hand, people coming from class-based languages are happy that they don’t have to deal with JavaScript’s unconventional inheritance mechanisms, anymore. On the other hand, there are many JavaScript programmers who argue that what’s complicated about JavaScript is not prototypal inheritance, but constructors.

ES6 classes provide a few clear benefits:

- They are backwards compatible with much of the current code.
- Compared to constructors and constructor inheritance, classes make it easier for beginners to get started.
- Subclassing is supported within the language.
- Built-in constructors are subclassable.
- No library for inheritance is needed, anymore; code will become more portable between frameworks.
- They provide a foundation for advanced features in the future: traits (or mixins), immutable instances, etc.
- They help tools that statically analyze code (IDEs, type checkers, style checkers, etc.).

Let’s look at a few common complaints about ES6 classes. You will see me agree with most of them, but I also think that they benefits of classes much outweigh their disadvantages. I’m glad that they are in ES6 and I recommend to use them.

### 15.8.1 Complaint: ES6 classes obscure the true nature of JavaScript inheritance
Yes, ES6 classes do obscure the true nature of JavaScript inheritance. There is an unfortunate disconnect between what a class looks like (its syntax) and how it behaves (its semantics): It looks like an object, but it is a function. My preference would have been for classes to be constructor objects, not constructor functions. I explore that approach in [the Proto.js project](https://github.com/rauschma/proto-js), via a tiny library (which proves how good a fit this approach is).

However, backwards-compatibility matters, which is why classes being constructor functions also makes sense. That way, ES6 code and ES5 are more interoperable.

The disconnect between syntax and semantics will cause some friction in ES6 and later. But you can lead a comfortable life by simply taking ES6 classes at face value. I don’t think the illusion will ever bite you. Newcomers can get started more quickly and later read up on what goes on behind the scenes (after they are more comfortable with the language).

### 15.8.2 Complaint: Classes provide only single inheritance
Classes only give you single inheritance, which severely limits your freedom of expression w.r.t. object-oriented design. However, the plan has always been for them to be the foundation of a multiple-inheritance mechanism such as traits.

> traits.js: traits library for JavaScript
> Check out traits.js if you are interested in how traits work (they are similar to mixins, which you may be familiar with).

Then a class becomes an instantiable entity and a location where you assemble traits. Until that happens, you will need to resort to libraries if you want multiple inheritance.

### 15.8.3 Complaint: Classes lock you in, due to mandatory new
If you want to instantiate a class, you are forced to use new in ES6. That means that you can’t switch from a class to a factory function without changing the call sites. That is indeed a limitation, but there are two mitigating factors:

- You can override the default result returned by the new operator, by returning an object from the constructor method of a class.
- Due to its built-in modules and classes, ES6 makes it easier for IDEs to refactor code. Therefore, going from new to a function call will be simple. Obviously that doesn’t help you if you don’t control the code that calls your code, as is the case for libraries.

Therefore, classes do somewhat limit you syntactically, but, once JavaScript has traits, they won’t limit you conceptually (w.r.t. object-oriented design).

## 15.9 FAQ: classes
### 15.9.1 Why can’t classes be function-called?
Function-calling classes is currently forbidden. That was done to keep options open for the future, to eventually add a way to handle function calls via classes. One possibility is to do so via a special method (e.g. [Symbol.functionCall]).

### 15.9.2 How do I instantiate a class, given an Array of arguments?
What is the analog of Function.prototype.apply() for classes? That is, if I have a class TheClass and an Array args of arguments, how do I instantiate TheClass?

One way of doing so is via the spread operator (...):

```javascript
function instantiate(TheClass, args) {
    return new TheClass(...args);
}
```

Another option is to use Reflect.construct():

```javascript
function instantiate(TheClass, args) {
    return Reflect.construct(TheClass, args);
}
```

## 15.10 What is next for classes?
The design motto for classes was “maximally minimal”. Several advanced features were discussed, but ultimately discarded in order to get a design that would be unanimously accepted by TC39.

Upcoming versions of ECMAScript can now extend this minimal design – classes will provide a foundation for features such as traits (or mixins), value objects (where different objects are equal if they have the same content) and const classes (that produce immutable instances).

## 15.11 Further reading
The following document is an important source of this chapter:

[“Instantiation Reform: One last time”](https://github.com/rwaldron/tc39-notes/blob/master/es6/2015-01/jan2015-allen-slides.pdf), slides by Allen Wirfs-Brock.