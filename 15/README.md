----

15. Class

이 장에서는 ES6 클래스가 어떻게 작동하는지에 대해 설명합니다.

15.1 Overview

클래스와 하위 클래스

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

클래스 사용하기

```javascript
> const cp = new ColorPoint(25, 8, 'green');

> cp.toString();
'(25, 8) in green'

> cp instanceof ColorPoint
true
> cp instanceof Point
true
```

ES6의 클래스는 근본적으로 새로운 것은 아닙니다: 오래된 방식의 생성자 함수를 작성하기 위한 더 편리한 구문을 제공하는 것입니다. typeof 연산자를 사용하면 알 수 있습니다.

```javascript
> typeof Point
'function'
```

15.2 The essentials

15.2.1 Base classes

ECMAScript 6의 클래스는 이렇게 정의합니다.

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
ES5의 생성자 함수처럼 이 클래스를 사용할 수 있습니다.

```javascript
> var p = new Point(25, 8);
> p.toString()
'(25, 8)'
```
사실, 클래스 정의 결과는 function입니다.

```javascript
> typeof Point
'function'
```

하지만, function과는 다르게 new를 사용해서 호출해야만 합니다.  ( 이론적인 근거는 뒤에서 다룹니다. )

```javascript
> Point()
TypeError: Classes can’t be function-called
```

> 스펙상, 클래스를 함수처럼 호출하는 것은 function 객체의 내부 메서드 [[Call]]에서 거부됩니다.

15.2.1.1 클래스 선언은 호이스팅되지 않습니다.

함수 선언은 호이스팅됩니다: 스코프 내에 선언된 함수들은 - 선언된 위치에 상관없이 - 즉시 사용 가능합니다. 즉, 나중에 선언된 함수를 호출할 수 있습니다.

```javascript
foo(); // 가능. foo가 호이스팅되므로.

function foo() {}
```

반면 클래스 선언은 호이스팅되지 않습니다. 따라서, 클래스는 오직 클래스가 정의된 후에만 존재하며 평가됩니다. 이전에 억세스하면 ReferenceError가 발생합니다.

```javascript
new Foo(); // ReferenceError

class Foo {}
```

이런 제한의 이유는 클래스는 extends라는 독특한 표현식을 가질 수 있기 때문입니다. 이 표현식은 적절한 "위치"에서 평가되어야 하며, 평가는 호이스팅될 수 없습니다. 

호이스팅 되지 않는다는 것은 생각보다 심각한 제한은 아닙니다. 예를 들어, 클래스 선언 이전에 위치한 함수는 여전히 클래스를 참조할 수 있습니다. 하지만 함수를 호출하기 전에 클래스 정의가 평가될 때까지 기다려야 합니다. 

```javascript
function functionThatUsesBar() {
    new Bar();
}

functionThatUsesBar(); // ReferenceError
class Bar {}
functionThatUsesBar(); // OK
```

15.2.1.2 Class expressions

function과 비슷하게, 선언식과 표현식 두가지 방법으로 클래스를 정의할 수 있습니다. 


함수 표현식과 유사하게 클래스 표현식도 익명으로 할 수 있습니다.

```javascript
const MyClass = class {
    ···
};
const inst = new MyClass();
```

또한 함수 표현식처럼 클래스 표현식도 내부에서만 참조 가능한 이름을 가질 수 있습니다.

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

예제의 마지막 두줄에서 보이듯이 Me는 클래스 외부에서는 참조할 수 없지만 내부에서는 가능합니다.

15.2.2 Inside the body of a class definition

클래스 body는 데이터 프로퍼티가 아닌 메소드만을 포함할 수 있습니다. 프로토타입에 데이터 프로퍼티를 포함하는 것은 일반적으로 안티 패턴으로 간주됩니다. 그래서 이런 방식이 권장됩니다.

15.2.2.1 constructor, static methods, prototype methods

클래스 정의에서 흔하게 볼 수 있는 메소드의 세가지 방법을 살펴보겠습니다.

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

