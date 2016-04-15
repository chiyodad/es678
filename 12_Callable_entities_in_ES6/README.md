# 12. ECMAScript6 호출 할수 있는 개체들
이 챕터는 당신이 ES6안에서 호출 할 수 있는 개체(function calls, method calls, 등)를 어떻게 적절하게 사용하는 방법에 대해 알려 준다.

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

ES6안에서 더 특화되었다. 이 세개의 의무들은 아래처럼 처리되어있다. (클래스 정의는 클래스 생성을 위한 두 개의 생성자중 하나 입니다.- 클래스 선언 또는 클래스 표현식)

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

비록 그들의 동작이 다르나(뒤에 설명하겠음), 모든 개체는 함수다. 예를 들면:
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
모든 함수 호출들, 대부분의 ES6 코드는 모듈에 포함되고, 그 모듈 몸통은 암시적으로 stric mode이라는 것을 기억하는 것이 중요하다.

### 12.2.2 super에 의한 호출은 특별한 지역으로 제한되어 있다.
호출의 두가지 종류는 super키워드를 통해 만들어 진다; 그것들의 사용은 특정지역으로 제한된다.
* Super-메소드 호출: super.method('abc')
오직 객체 리터럴 이나 파생 클래스 정의 안에서의 메소드 정의에서만 가능하다.
* Super-생성자 호출: super(8)
오직 파생 클래스 정의 안의 특별 메소드인 constructor()에서만 가능하다.

### 12.2.3 비 메소드 함수 대 메소드
미 메소드 함수와 메소드의 차이는 ECMAScript 6 안에서 더욱더 명백하다. 지금 둘 다 그들만의 할 수 있는 특별한 개체가 있다.
* 애로우 함수는 비 메소드 함수를 만든다. 이것은 this(과 다른 변수들)을 둘러쌓인 스코프(어휘적 this)를 통해 가져온다. 
* 메소드 정의는 메소드를 만든다. 이것은 super, 부모 프로퍼티의 인용 그리고 부모 메소드 호출을 지원을 제공합니다.

## 12.3 호출 개체 사용을 위한 추천
이 섹션은 호출 개체 사용에 대한 팁을 준다: 언제 어느 개체 사용이 최선인가에 대하여: 등.

### 12.3.1 콜백으로 애로우 함수를 선호
콜백으로써, 애로우 함수는 전통적인 함수에 비해 두 가지 이점을 갖는다.:
* this는 어휘적 이므로 사용하기 안전하다.
* 이 신택스는 더 간결하다. 많은 고차 함수나 메서드가 사용된  함수형 프로그래밍에서 특히 중요하다. (파라메터가 함수인 함수와 메서드)

여러줄에 걸친 콜백 함 수때문에 나 역시 전통적인 함수 표현식을 수용하는 것을 발견할 수 있다. 그러나 당신은 this를 조심해야 한다.

#### 12.3.1.1 문제: 함축적인 파라미터로써 this
아아, 어떤 자바스크립트 API는 this를 콜백 함수의 함축적인 인자로 사용된다. 그것은 당신이 애로우 함수를 사용하는것을 막는다. 예를 들면: 이 B라인의 this는 A라인 안의 함수의 함축적 인자 이다.

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

### 12.3.2 독립적 함수로써, 함수 선언을 선호
독립적인 함수(대 콜백)로써, 나는 함수 선언을 선호한다:
```
function foo(arg1, arg2) {
    ···
}
```
장점:
* 주관전으로 나는 그것이 보기 더 좋다. 당신이 눈에 띄는 구조를 원할때 이 때, 함수 상세 키워드는 유리하다.
* 그것은 제너레이터 함수 선언과 비슷하고, 코드를 일관성 있게 보여진다.

하나의 경고가 있다. 일반적으로 당신은 독립적인 함수안에서 this는 필요 없다. 만일 당신 이것을 사용한다면, 당신은 둘러쌓인 스코프의 this를 접근하길 원한다.(예: 독립적인 함수를 포함한 메소드). 함수 선언은 당신에게 스코프에 둘러싸인 this를 감추고 그 자신의 this를 갖도록 한다. 그러므로 당신은 린터가 당신에게 함수 선언안에서 this를에 대한 경고를 하게 할 수 있다.

독립적 함수의 다른 옵션은 애로우 함수에 변수들은 할당 하는 것이다. 이것은 this에 대한 문제를 피한다. 왜냐하면 이것은 어휘적이기 때문이다.

```
const foo = (arg1, arg2) => {
    ···
};
```

### 12.3.3 메서드를 위한 메서드 정의 선호
메서드 정의는 super를 사용하는 메서드를 만드는 유일한 방법이다. 그것은 객체 리터럴과 클래스 안에서 명백한 선택이다(어디서나 그것들은 메서드를 정의하는 유일한 방법 이다.). 그러나 기존의 객체에 메소드를 어떻게 추가하지? 예를 들면

```
MyClass.prototype.foo = function (arg1, arg2) {
    ···
};
```
다음은 ES6에서 같은것을 하는 빠른 방법이다(주의: Object.assing()은 super 프로퍼티를 사용하는 메서드를 못 옮긴다.).
```
Object.assign(MyClass.prototype, {
    foo(arg1, arg2) {
        ···
    }
});
```
더 많은 정보와 주의를 위하여 Object.assign()섹션을 찾아라.

### 12.3.4 메서드 대 콜백
객체 메소드와 객체의 콜백은 세밀한 차이가 있다.

#### 12.3.4.1 프로퍼티가 메소드인 객체
메소드의 this는 메소드 콜의 수신자(receiver)이다. (예: 만약 메소드 콜이 obj.m(..)이라면 obj이다)

예를 들면, 당신은 WHATWG 스트림 API를 아래와 같이 사용할 수 있다.

```
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
```
obj는 start, pull, calls 메소드를 갖는 객체이다.
따라사, 이 메소드는 객체-지역 상태(줄 \*) 그리고 각각의 호출 (줄 \*\*)을 위하여 this를 사용 할 수 있다.

#### 12.3.4.2 프로퍼티가 콜백인 객체
애로우 함수의 this는 둘러쌓인 스코프의 this(어휘적 this)이다. 애로우 함수는 좋은 콜백을 만든다 왜냐하면 그것은 콜백을 위하여 당신이 일반적으로 원하는 행동(진짜, 비 메소드 함수들)이기 때문이다. 콜백은 그것 둘러쌓인 스코프의 this를 덮는 자신의 this가 없어야 한다.

만약 stat, pull, cancel 프로퍼티가 애로우 함수라면 그들은 surroundingMethod()의 this를 사용한다. (그들에 둘러싸인 스코프)
```
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
```
만일 줄 *에서 출력은 당신을 놀라게 하고 아래의 코드를 고려하게 된다.
```
const obj = {
    foo: 123,
    bar() {
        const f = () => console.log(this.foo); // 123
        const o = {
            p: () => console.log(this.foo), // 123
        };
    },
}
```
메소드 bar() 안에서, f, o.p 동작은 같다. 왜냐하면 둘다 애로우 함수는 같은 어휘적 스코프, bar()를 갖기 때문이다. 마지막 객체 리터럴로 둘러싸여 있는 애로우 함수는 바뀌지 않는다.  

### 12.3.5 ES6에서 IIFEs를 피해라
이 섹션은 ES6에서 IIFEs를 피하는 팁을 준다.

#### 12.3.5.1 IIFE를 블럭으로 치환
ES5에서는 당신이 지역 변수를 유지하기 원한다면 당신은 IIFE를 사용했다.
```
(function () {  // open IIFE
    var tmp = ···;
    ···
}());  // close IIFE

console.log(tmp); // ReferenceError
```
ES6 안에서 당신은 간단하게 블럭과 let또는 const 선언을 사용할 수 있다.
```
{  // open block
    let tmp = ···;
    ···
}  // close block

console.log(tmp); // ReferenceError
```
#### 12.3.5.2 IIFE를 모듈로 치환
ES5 안에서 라이브러리(RequireJS, browserify, webpack)를 통해 모듈을 사용하지 않는 코드, revealing module pattern는 인기 있고 IIFE에 기반을 둔다. 이것의 장점은 무엇이 public인지, private인지 명확하게 구분한다.
```
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
```
이 모듈 패턴은 전역변수 하나를 생성하고 아래처럼 사용한다.
```
my_module.myFunc(33);
```
ES6에서 모듈들은 내장되었고 그것이 그것을 사용하는 장벽이 낮은 이유이다.
```
// my_module.js

// Module-private variable:
let countInvocations = 0;

export function myFunc(x) {
    countInvocations++;
    ···
}
```
이 모듈은 전역 변수를 생성하지 않고 그것은 아래와 같이 사용된다.
```
import { myFunc } from 'my_module.js';

myFunc(33);
```
#### 12.3.5.3 애로우 함수 즉시 실행
당신이 여전히 ES6에서 즉시 실행 함수가 필요할 때 한 가지  사용 사례가 있다: 때때로 당신은 하나의 표현식이 아닌 문장의 순서를 통한 결과를 생성 할 수 있다. 만일 당신이 그 문장을 인라인을 원한다면, 당신은 함수를 즉시 실행해야 한다. ES6에서 당신은 당신이 원할 때 즉시 호출된 애로우 함수를 사용할 수 있다.
```
const SENTENCE = 'How are you?';
const REVERSED_SENTENCE = (() => {
    // Iteration over the string gives us code points
    // (better for reversal than characters)
    const arr = [...SENTENCE];
    arr.reverse();
    return arr.join('');
})();
```
당신은 반드시 보이는것(이 괄호는 애로우 함수를 감싼지만, 완전한 함수 호출을 감싸지 않는다.) 처럼 괄호로 묶어야 하는것을 주목하라. 애로우 함수에 대해서 13장에서 자세하게 설명한다.

### 12.3.6 생성자로써 클래스 사용
ES5에서 생성자함수는 객체를 위한 팩토리 만드는 주요 방법(그러나 많은 다른 테크닉, 틀림없이 더 우아한 방법이 있다.). ES6에서 클래스는 생성자 함수를 만드는 주요 방법이다. 몇몇의 프레임웍은 그것들은 그들의 커스톰 상속 API를 위한 대안처럼 제공한다.

## 12.4 ES6 호출가능한 개체 상세
각 ES6 호출가능한 개체에 대한 상세를 설명하기 전에 이 섹션은 컨닝지로 시작한다.

### 12.4.1 컨닝지 : 호출 개체들
#### 12.4.1.1 호출 가능한 개체의 동작과 구조
값:

| |Func decl/Func expr|Arrow|Class	Method|
|---|---|---|---|
|Function-callable|V|V|X|V|
|Constructor-callable|V|X|V|X|
|Prototype|F.p|F.p|SC|F.p|
|Property prototype|V|X|V|X|

완전한 생성자:

| |Func decl|Func expr|Arrow|Class|Method|
|---|---|---|---|---|---|
|Hoisted|V| |X| |
|Creates window prop. (1)|V| |X| |
|Inner name (2)|X|V|V|X|

바디:

| |Func decl|Func expr|Arrow|Class (3)|Method|
|---|---|---|---|---|---|
|this|V|V|lex|V|V|
|new.target|V|V|lex|V|V|
|super.prop|X|X|lex|V|V|
|super()|X|X|X|V|X|

셀 요약:
* V 존재함, 허용
* X 존재하지 않음 허용 X
* 빈셀: 해당사항 없음, 관련 없음
* lex: 어휘적, 둘러쌓인 어휘 스코프를 상속
* F.p: Function.prototype
* SC: 파생클래스를 위한 슈퍼 클래스, 기반클래스를 위한 Function.prototype. 이 상세는 15장에 설명되어 있다.

주석:
* (1) 선언은 전역 객체를 위한 프로퍼티 를 만드는것에 대한 규칙은 변수와 스코프 장에 설명되어 있다.
* (2) 기명식 함수 표현식과 기명 클래스의 내부 이름은 클래서 장에서 설명되어 있다.
* (3) 이 컬럼은 클래스 생성자의 바디에 대한 것이다.

제너레이터 함수나 메서든 무엇인가? 이것들은 그들의 비 제너레이터 반대 처럼 동작한다. 두가지 예외에서:
* 제너레이터 함수와 메서드는 (GeneratorFunction).prototype을 갖는다.((GeneratorFunction)은 내부 객제이다."Inheritance within the iteration API (including generators)"이 부분을 도표를 보면)
* 당신은 제너레이터 함수를 생성자 호출 할 수 없다.

#### 12.4.1.2 this의 규칙

| |FC strict|FC sloppy|MC|new|
|---|---|---|---|---|
|Traditional function|undefined|window|receiver|instance|
|Generator function|undefined|window|receiver|TypeError|
|Method|undefined|window|receiver|TypeError|
|Generator method|undefined|window|receiver|TypeError|
|Arrow function|lexical|lexical|lexical|TypeError|
|Class|TypeError|TypeError|TypeError|SC protocol|

컬럼 제목 요약:
* FC: 함수 호출
* MC: 메소드 호출
 
셀 요약:
* lexical: 둘러싸인 언휘적 스코프를 상속
* SC protocol: 서브 클래스 규약 (기저 클래스에서 신규 인스턴트, 파생 클래스안에서 슈퍼클래스로부터 받는다.)

### 12.4.2 전통적 함수
당신이 ES5부터 알고 있는 함수가 있다. 그들 생성에는 두 가지가 있다.
* 함수 표현식:
  ```
  const foo = function (x) { ··· };
  ```
* 함수 선언식:
  ```
  function foo(x) { ··· }
  ```

this에 대한 규칙
* 함수 호출: 엄격 모드에서는 this는 undefined이고 슬로피 모드에서는 전역 객체 이다.
* 메소드 호출: this는 메소드콜의 수신자이다. (또는 call/apply의 첫번째 인자)
* 생성자 호출: this는 새로 생성된 인스턴스 이다.

### 12.4.3 제너레이터 함수
제너레이터 함수는 제너레이터 장에서 설명되어 있다. 그들의 신택스는 전통적인 함수와 비슷하지만 그들은 추가적인 *를 갖는다.

* 제너레이터 함수 표현식:
  ```
  const foo = function* (x) { ··· };
  ```
* 제너레이터 함수 선언식:
  ```
  function* foo(x) { ··· }
  ```
아래와 같이 this에 대한 규칙. 그것은 결코 제너레이터 객체를 참조하지 않는다는 것을 주목해라
* 함수/메소드 호출: this는 전통적인 함수처럼 다뤄진다. 호출 결과는 제너레이터 객체 이다.
* 생성자 호출: 당신은 제너레이션 함수로 생성자 호출을 할 수 없다. 당신이 그렇게 한다면 TypeError를 던진다.

### 12.4.4 메소드 정의
메소드 정의는 객체 리터럴 안에서 나타날 수 있다.
```
const obj = {
    add(x, y) {
        return x + y;
    }, // comma is required
    sub(x, y) {
        return x - y;
    }, // comma is optional
};
```
그리고 클래스 정의 안에서는:
```
class AddSub {
    add(x, y) {
        return x + y;
    } // no comma
    sub(x, y) {
        return x - y;
    } // no comma
}
```
당신이 본것 처럼 당신은 반드시 객체리터럴에서 쉼표로 메소드 정의를 구분해야 한다. 그러나 클래스 정의에서는 메소드사이에 구분자가 없다. 이 양식은 특히 getter와 setter에 대해서 신택스를 유지하기 위해 필수다.

메소드 정의는 당신이 슈퍼-프로퍼티를 참조인 super를 사용 할 수 있는 유일한 곳이다. super를 사용하는 유일한 메소드 정의는 [[HomeObject]]프로퍼티를 갖는 함수를 생성한다. 이 기능을 위해 이것은 요구되어 진다. (자세한 내용은 클래스 장에서 설명한다.)

규칙:
* 함수 호출: 만약 당신이 메서드호출을 추출하고 함수처럼 호출한다면, 그것은 마치 전통적인 함수처럼 동작한다.
* 메소드 호출: 전통적인 함수처럼 동작하나 추가적으로 super를 허용한다.
* 생성자 호출: TypeError를 발생시킨다.
클래스 선언 내부에서, constructor이름의 메소드는 특별하다. 그 설명은 나중에 하겠다.

### 12.4.5 제너레이터 메소드 정의
제너레이터 메소드는 제너레이터 장에서 설명되어 있다. 그 신택스는 메소드 정의랑 유사하나 별표(*)를 가지고 있다.
```
const obj = {
    * generatorMethod(···) {
        ···
    },
};
```
```
class MyClass {
    * generatorMethod(···) {
        ···
    }
}
```
규칙:

* 제너레이터 메소드 호출은 제너레이터 객체를 반환한다.
* 당신은 마치 일반적인 메소드 정의 처럼 this와 super를 사용할 수 있습니다. 
 
### 12.4.6 애로우 함수
애로루 함수는 13장에서 설명하고 있다.
```
const squares = [1,2,3].map(x => x * x);
```
애로우 함수안에 다음 변수들은 어휘적이다(둘러싸인 스코프에서 얻음).

* arguments
* super
* this
* new.target

규칙:
* 함수 호출: 어휘적 this 등.
* 메소드 호출: 당신은 애로우 함수를 메소드 처럼 사용할 수 있으니, 그들의 this는 여전히 어휘적이고 메서드 호출의 수신자를 참조하지 않는다.
* 생성자 호출: TypeError를 발생시킨다.

### 12.4.7 클래스
클래스는 클래스 장에서 설명한다.

```
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
```

메소드 constructor는 특별하다 왜나하면 그것은 클래스가 되기 때문이다. 그것이 바로,  클래스는 생성자 함수와 아주 유사하다.
```
> Point.prototype.constructor === Point
true
```

규칙:
* 함수/메소드 호출: 클래스는 함수나 메서드 처럼 호출 될 수 없다(그 이유는 클래스 장에 있음).
* 생성자 호출: 하위 클래스 지원 규칙을 따라라. 기저 클래스안에서 인스턴스는 생성되고 this는 그것은 참조한다. 파생 클래스는 그것의 인스턴스를 슈퍼클래스로 부터 받는다. 그것이 그것이 this를 접근하기 전에 super를 호출이 필요한 이유이다.

## 12.5 ES5, ES6에서 전달과 직접 메소드 호출
자바스크립트에서 메서드 호출을 두가지 이다.

전달인 경우 예 
```
obj.somMethod(arg0, arg1)
```
직접 예
```
someFunc.call(thisValue, arg0, arg1)
```
이 섹션은 두가지 작동 방법과 당신이 ES6에서 직접 호출이 드문 이유에 대해서 설명한다. 당신이 시작하기 이전에 나는 당신의 프로토타입 체인에 대한 지식 w.r.t.를 새롭게 할 것 입니다.

### 12.5.1 배경: 프로토타입 체인
자바스크립트 안에서 각 객체는 실제로 하나나 그 이상의 객체 체인이라는  것을 기억해라. 첫 객체 마지막 객체들로 부터 상속 받는다. 예를 들면 배열['a','b']의 프로토타입체인은 아래와 같다.
1. 인스턴스, 엘리먼트 'a'와 'b'를 갖는다.
2. Array.prototype, 이 프로퍼티들은 Array 생성자로 부터 제공되어 진다.
3. Object.prototype, 이 프로퍼티들은 Object생성자로 부터 제공되어 진다.
4. null (체인의 마지막 부분, 실제 이것들의 멤버는 아니다.)

당신은 Object.getPrototypeOf()를 통해 체인을 검사할 수 있다.
```
> var arr = ['a', 'b'];
> var p = Object.getPrototypeOf;

> p(arr) === Array.prototype
true
> p(p(arr)) === Object.prototype
true
> p(p(p(arr)))
null
```
"이전의" 객체에 있는 프로퍼티들은 "나중의"객체 프로퍼티를 오버라이드 한다. 예를 들면 Array.prototype은 Object.prototype.toString()을 오버라이드 하여, toString() 메서드의 배열 특화 버전을 제공한다.
```
> var arr = ['a', 'b'];
> Object.getOwnPropertyNames(Array.prototype)
[ 'toString', 'join', 'pop', ··· ]
> arr.toString()
'a,b'
```
### 12.5.2 메서드 호출 전달
당신이 메서드 호출 arr.toString()를 봤다면, 당시는 실제 2단계로 수행되는것을 볼 수 있다.
1. 전달: arr의 프로토타입 체인에서 이름이 toString인 첫번째 프로퍼티의 값을 검색한다.
2. 호출: 그 값을 호출 하고 함축적 파라미터인 this에 메서드 호출의 수신자 arr를 설정한다.
당신은 함수의 call()메서드를 사용해서 두 단계를 명시적으로 만들수 있다.
```
> var func = arr.toString; // dispatch
> func.call(arr) // direct call, providing a value for `this`
'a,b'
```
### 12.5.3 직접 메소드 호출
자바스크립트에서 직접적으로 메소드 호출을 만드는 방법은 두 가지 이다.
* Function.prototype.call(thisValue, arg0?, arg1?, ···)
* Function.prototype.apply(thisValue, argArray)
두가지 call, apply메서드는 함수를 호출한다. 그것들은 당신이 this위해 값을 지정하는 일반적인 함수 호출과 다른다. call은 각각의 파라메터를 통해 메서드의 인자를 제공하고 apply를 배열을 통해 메서드의 인자를 제공한다.

전달된 메서드 호출에서 수신자는 두가지 역할을 한다. 그것은 메서드를 찾기 위해 사용되고, 그것은 함축적인 파라미터이다. 첫번째 역할의 문제는 만약에 당신이 메서드을 호출하기 원한다면, 메소드는 반드시 프로토타입 체인의 객체가 있어야 한다.
직접 메서드 호출에서 메서드는 어디서나 올 수 있다. 그것은 당신에서 다른 객체의 메서드를 빌리는 것을 허용한다. 예를 들면 당신은 Object.prototype.toString을 빌릴 수 있고, 따라서 배열 arr에 toString의 취소 오버라이드 구현을 적용한다.
```
> const arr = ['a','b','c'];
> Object.prototype.toString.call(arr)
'[object Array]'
```
toString()의 배열 버전은 다른 결과를 만든다.
```
> arr.toString() // dispatched
'a,b,c'
> Array.prototype.toString.call(arr); // direct
'a,b,c'
```
다양한 종류의 객체(단지 "그들의" 생성자의 인스턴스는 아님)와 함께 일하는 메서드는 제너릭으로 불린다. Speaking JavaScript는 제너릭인 모든 메서드의 리스트를 갖는다. 그 리스트는 대부분의 배열 메소드와 Object.prototype(이것은 모든 객체에서 동작어야 하고 따라서 함축적인 제너릭이다.)의 모든 메소드를 포함한다. 

### 12.5.4 직접 메소드 호출 사용 사례
이 섹션은 직접 메소드 호출의 사용사례를 다룬다. 때마다. 나는 ES5에서의 사용사례와 다음으로 ES6에서 어떻게 바뀔지 설명하겠다.(ES6에선 당신은 조금만 직접호출이 필요할 것이다.)

#### 12.5.4.1 ES5: 배열을 통한 메서드에게 파라미터 제공
어떤 함수는 다양한 값을 받아들이지만 파라미터당 하나의 값이다. 당신이 배열을 통해 값을 주길 원한다면?

예를 들면, push()는 당신에게 배열에 파괴적으로 몇몇 값을 추가시켜준다.
```
> var arr = ['a', 'b'];
> arr.push('c', 'd')
4
> arr
[ 'a', 'b', 'c', 'd' ]
```
그러나 당신은 전체의 배열에 파괴적으로 추가 할 수 없다. 당신은 apply()를 사용해 제한적으로 할 수 있다.
```
> var arr = ['a', 'b'];
> Array.prototype.push.apply(arr, ['c', 'd'])
4
> arr
[ 'a', 'b', 'c', 'd' ]
```
유사하게, Math.max()와 Math.min()는 오직 단일 값으로 동작한다.
```
> Math.max(-1, 7, 2)
7
```
apply()를 통해, 당신은 배열을 통해 그들을 사용할 수 있다.
```
> Math.max.apply(null, [-1, 7, 2])
7
```
#### 12.5.4.2 ES6: 펼침 연산자(...)는 대개 apply()를 바꾼다.
단지 당신이배열을 인자값으로 바꾸길 원해서 apply()통한 직접 메서드 호출을 만드는 것은 꼴사납기 때문에 ES6은 이것을 위해 펼침 연산자 (...) 가 있다. 이것은 심지어 전달 메소드 호출에서도 이 기능을 제공합니다.

```
> Math.max(...[-1, 7, 2])
7
```
다른 예:
```
> const arr = ['a', 'b'];
> arr.push(...['c', 'd'])
4
> arr
[ 'a', 'b', 'c', 'd' ]
```
추가적으로, 펼침은 또한 새로운 메소드에서도 동작한다.
```
> new Date(...[2011, 11, 24])
Sat Dec 24 2011 00:00:00 GMT+0100 (CET)
```
apply()는 new와 사용될 수 없다는점을 주목해라. 뛰어난 업적은 단지 ES5안에서 복잡한 해결 방법을 통해 달성된다.

#### 12.5.4.3 ES5: 유사 배열 객체를 배열로 변환
자바스크립트 안에서 어떤 객체는 유사 배열이다. 그것들은 거희 배열이지만, 배열 메소드에 어느것도 가지고 있지 않다. 두 예제를 보자.

첫째, 함수의 특수 변수 arguments는 유사 배열이다. 이것은 length와 원소를 접근하는 인덱스를 갖는다.
```
> var args = function () { return arguments }('a', 'b');
> args.length
2
> args[0]
'a'
```
그러나 arguments는 배열의 인스턴스가 아니고 forEach()메소드가 없다.
```
> args instanceof Array
false
> args.forEach
undefined
```
둘째로 DOM메소드인 document.querySelectorAll()은 NodeList의 인스턴스를 반환한다.
```
> document.querySelectorAll('a[href]') instanceof NodeList
true
> document.querySelectorAll('a[href]').forEach // no Array methods!
undefined
```
따라서, 많은 복잡한 연산자 때문에 우선 당신은 유사 배열 객체를 배열로 변경할 필요가 있다. Array.prototype.slice()를 통해 이루어 진다. 이 메소드는 그것의 수신자의 원소를 새로운 배열에 복사한다.
```
> var arr = ['a', 'b'];
> arr.slice()
[ 'a', 'b' ]
> arr.slice() === arr
false
```
만약 당신이 slice()를 직접 호출 한다면 당신은 NodeList를 배열로 바꿀 수 있다.
```
var domLinks = document.querySelectorAll('a[href]');
var links = Array.prototype.slice.call(domLinks);
links.forEach(function (link) {
    console.log(link);
});
```
그리고 당신은 arguments를 배열로 바꿀 수 있다.
```
function format(pattern) {
    // params start at arguments[1], skipping `pattern`
    var params = Array.prototype.slice.call(arguments, 1);
    return params;
}
console.log(format('a', 'b', 'c')); // ['b', 'c']
```
#### 12.5.4.4 ES6: 유사 배열은 더 적은 부담이다.
한편으로, ES6은 Array.from()을 가지고 있고, 이것은 유사배열 객체를 배열로 만드는 간단한 방법이다.
```
const domLinks = document.querySelectorAll('a[href]');
const links = Array.from(domLinks);
links.forEach(function (link) {
    console.log(link);
});
```
다른 한편, 당신은 유사배열 arguments를 필요하지 않다. 왜냐하면 ES6은 rest 파라미터를 가지고 있기 때문이다. (세개의 점으로 선언)
```
function format(pattern, ...params) {
    return params;
}

console.log(format('a', 'b', 'c')); // ['b', 'c']
```
#### 12.5.4.5 ES5: hasOwnProperty()를 안전하게 사용
obj.hasOwnProperty('prop')는 당신에게 obj가 자신의 프로퍼티(비 상속)에 prop가 있는지 여부를 말해준다.
```
> var obj = { prop: 123 };

> obj.hasOwnProperty('prop')
true

> 'toString' in obj // inherited
true
> obj.hasOwnProperty('toString') // own
false
```
그러나 만약 Object.prototype.hasOwnProperty가 오버라이드 되었다면, hasOwnProperty를 전달을 통해 호출하는 것은 적절하게 작동하는것을 막을 수 있다.
```
> var obj1 = { hasOwnProperty: 123 };
> obj1.hasOwnProperty('toString')
TypeError: Property 'hasOwnProperty' is not a function
```
만일 Object.prototype이 객체의 프로토타입 체인에 없다면, 전달을 통한 hasOwnProperty은 또한 아마도 이용할 수 없다.
```
> var obj2 = Object.create(null);
> obj2.hasOwnProperty('toString')
TypeError: Object has no method 'hasOwnProperty'
```
이 둘의 경우, 해결책은 hasOwnProperty를 직접 호출로 만드는 것이다.
```
> var obj1 = { hasOwnProperty: 123 };
> Object.prototype.hasOwnProperty.call(obj1, 'hasOwnProperty')
true

> var obj2 = Object.create(null);
> Object.prototype.hasOwnProperty.call(obj2, 'toString')
false
```
#### 12.5.4.6 ES6: hasOwnProperty() 적은 필요
hasOwnProperty()는 거희 객체를 통한 맵 구현을 위해 사용되었다. 감사하게도 ES6은 맵 자료구조가 지원된다. 그 말은 당신은 hasOwnProperty()를 적게 필요 한다는 것이다.

#### 12.5.4.7 ES5: 중간 객체를 피하기
join()과 같은 배열 메소드를 적용하여 일반적으로 문자열을 만드는건 두가지 단계를 포함한다.
```
var str = 'abc';
var arr = str.split(''); // step 1
var joined = arr.join('-'); // step 2
console.log(joined); // a-b-c
```
문자열은 유사배열이고 제너릭 배열 매소드의 this값이 될 수 있다. 그러므로 직접 호출은 당신이 1단계를 피하게 해준다.
```
var str = 'abc';
var joined = Array.prototype.join.call(str, '-');
```
유사하게, 당신이 그것을 분리한 후나 직접 메소드 호출을 후에 당신은 map()을 문자열로 적용 할 수 있다.

```
> function toUpper(x) { return x.toUpperCase() }
> 'abc'.split('').map(toUpper)
[ 'A', 'B', 'C' ]

> Array.prototype.map.call('abc', toUpper)
[ 'A', 'B', 'C' ]
```
직접 호출은 아마도 더 효과적 이지만, 그들은 덜 우와한것을 주목해라. 그들은 실제로 할 가치가 있는지 확인해라!


#### 12.5.4.8 ES6: 중간 객체 피하기
첫 단계에서 Array.from()은 변경하고 맵핑할 수 있다.  만일 당신이 두번째 인자값으로 콜백을 그것에게 제공한다면

```
> Array.from('abc', ch => ch.toUpperCase())
[ 'A', 'B', 'C' ]
```
참고로, 이 두단계 솔루션입니다.:

```
> 'abc'.split('').map(function (x) { return x.toUpperCase() })
[ 'A', 'B', 'C' ]
```

### 12.5.5 Object.prototype과 Array.prototype의 요약
당신은 빈객체 리터럴을 통해 Object.prototype(프로토타입이 Object.prototype인) 메소드에 접근할 수 있다. 예를 들면 다음 두 직접 호출 메소드는 동일하다.

```
Object.prototype.hasOwnProperty.call(obj, 'propKey')
{}.hasOwnProperty.call(obj, 'propKey')
```
이 같은 트릭은 Array.prototype에도 동작한다.

```
Array.prototype.slice.call(arguments)
[].slice.call(arguments)
```
이 패턴은 꽤 유명해졌다. 그것은 긴 비전처럼 명확하게 저자의 의도를 드러내지 않지만 그것은 훨신 덜 장확하다. 두 버전은 많은 차이가 없다.

## 12.6 함수의 name 프로퍼티
함수의 name 프로퍼티는 그것의 이름을 포함한다.
```
> function foo() {}
> foo.name
'foo'
```
이 프로퍼티는 디버깅과 (그것의 값은 스택 트레이스에서 보여진다.) 약간의 메타 프로그래밍 수행(이름등을 통해 함수를 선택) 할때 유용하다.

ES6이전에는, 이 프로퍼티는 이미 대부분의 엔진에서 지원한다. ES6에서 이것은 언어 표준의 한부분이 되었고 자동으로 자주 입력됩니다.

### 12.6.1 함수에게 name을 제공하는 생성자
다음 섹션에서 어떻게 다양한 프로그래밍 생성자를 통해 이름이 자동으로 설정되는지에 대해서 설명한다.

#### 12.6.1.1 변수 선언과 할당
만일 함수가 변수 선언에 의해 생성된다면 함수들은 그 이름을 선택한다.

```
let func1 = function () {};
console.log(func1.name); // func1

const func2 = function () {};
console.log(func2.name); // func2

var func3 = function () {};
console.log(func3.name); // func3
```
그러나 심지어 일반적인 할당에도 name은 정확하게 설정된다.
```
let func4;
func4 = function () {};
console.log(func4.name); // func4

var func5;
func5 = function () {};
console.log(func5.name); // func5
```
이름에 대해서는 애로우 함수들은 익명 함수 표현식과 유사하다.
```
const func = () => {};
console.log(func.name); // func
```
지금부터, 언제나 당신이 익명함수 표현식을 볼때, 당신은 애로우 함수도 같은 방법으로 동작하는것을 가정할 수 있다.

#### 12.6.1.2 디폴트 값
만약 함수가 디폴트 값이면 그것은 그것의 이름을 그것의 변수나 파라미터로 가져온다.

```
let [func1 = function () {}] = [];
console.log(func1.name); // func1

let { f2: func2 = function () {} } = {};
console.log(func2.name); // func2

function g(func3 = function () {}) {
    return func3.name;
}
console.log(g()); // func3
```
#### 12.6.1.3 기명 함수 정의
함수 선언과 함수 표현식은 함수 정의 이다. 이 시나리오는 오랜기간 제공되고 있다. 이름과 함께 함수 정의 그것을 name 프로퍼티로 전달한다.

예를 들어, 함수 선언:

```
function foo() {}
console.log(foo.name); // foo
```
기명 함수 표현식의 이름은 name 프로퍼티로 설정된다.

```
const bar = function baz() {};
console.log(bar.name); // baz
```
왜냐하면 이것은 처음 부분에서 왔기 때문에 이 함수 표현식의 이름 baz 다른 이름들을 앞선다. (예 이름 bar는 변수 선언을 통해 제공 된다.)

그러나 ES5에서는 함수 표현식 이름은 오직 함수 표현식 안에 변수이다.

```
const bar = function baz() {
    console.log(baz.name); // baz
};
bar();
console.log(baz); // ReferenceError
```

#### 12.6.1.4 객체 리터럴안에 메소드
만약 함수가 프로퍼티의 값이라면, 그것은 그것의 이름을 프로퍼티를 통해 얻는다. 메소드 정의(줄 A), 전통적인 프로퍼티 정의 (줄 B), 연산된 프로퍼티 키를 통한 프로퍼티 정의(줄 C) 또는 속기에 의한 프로퍼티 값(줄 D) 통해 발생되었다면, 
그것은 문제가 아닌다.
```
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
```
겟터들의 이름은 접두사 'get'을 가지고 있고 셋터들은 접두사로 'set'을 가진다.

```
let obj = {
    get foo() {},
    set bar(value) {},
};
let getter = Object.getOwnPropertyDescriptor(obj, 'foo').get;
console.log(getter.name); // 'get foo'

let setter = Object.getOwnPropertyDescriptor(obj, 'bar').set;
console.log(setter.name); // 'set bar'
```
### 12.6.1.5 클래스 정의 안에서 메소드
클래스 정의 안에서 메소드의 이름 짓기는 객체리터럴과 유사하다.

```
class C {
    m1() {}
    ['m' + '2']() {} // computed property key

    static classMethod() {}
}
console.log(C.prototype.m1.name); // m1
console.log(new C().m1.name); // m1

console.log(C.prototype.m2.name); // m2

console.log(C.classMethod.name); // classMethod
```
겟터와 셋터 또한 이름에 접두사로 각각 'get', 'set'을 같는다.

```
class C {
    get foo() {}
    set bar(value) {}
}
let getter = Object.getOwnPropertyDescriptor(C.prototype, 'foo').get;
console.log(getter.name); // 'get foo'

let setter = Object.getOwnPropertyDescriptor(C.prototype, 'bar').set;
console.log(setter.name); // 'set bar'
```

#### 12.6.1.6 키를 심볼로 갖는 메소드
ES6에서 메소드의 키가 심볼일 수 있다. 이 이 같은 메소드의 name 프로퍼티는 여전히 문자열이다.

* 만약 심볼이 디스크립션을 갖는다면, 그 메서드의 이름은 대괄호 안 디스크립션 이다.
* 반면에 이 메소드의 이름은 빈 문자열이다('').
```
const key1 = Symbol('description');
const key2 = Symbol();

let obj = {
    [key1]() {},
    [key2]() {},
};
console.log(obj[key1].name); // '[description]'
console.log(obj[key2].name); // ''
```

#### 12.6.1.7 클래스 정의
클래스 정의는 함수를 생성하는 것을 기억해라 이 함수들은 또한 name 프로퍼티의 정확한 설정를 갖는다.

```
class Foo {}
console.log(Foo.name); // Foo

const Bar = class {};
console.log(Bar.name); // Bar
```
#### 12.6.1.8 디폴트 exports
다음에 나온 모든 문장은 이름을 'default'로 설정한다.
```
export default function () {}
export default (function () {});

export default class {}
export default (class {});

export default () => {};
```

#### 12.6.1.9 다른 프로그래밍 생성자
* 제너레이터 함수와 제너레이터 메소드를 그들의 이름을 일반적인 함수와 메서드와 같은 방법으로 얻는다.
* new Function() 함수를 발생된 함수의 이름은 'anonymous'이다. 웹킷 버그는 왜 웹에서 필요한지에 대해서 설명한다.
* func.bind(...)으로 발생된 함수의 이름은 'bound'+func.name이다.
  ```
  function foo(x) {
      return x
  }
  
  const bound = foo.bind(undefined, 123);
  console.log(bound.name); // 'bound foo'
  ```

### 12.6.2 주의 사항
#### 12.6.2.1 주의 사항: 함수의 이름은 항상 생성때 할당된다.
함수 이름은 항상 생성하는 동안에 할당되고 그 후에 절때 변하지 않는다. 자바스크립트 엔진은 앞서 언급한 패턴에서 발견하고 처음부터 그것과 정확한 그것의 이름과 생성된다. 아래 코드는 functionFactory()에 의해서 생성된 함수의 이름이 줄A에서 할당되고 줄B에서 정의에 의해 변하지 않는다는 것을 증명한다.

```
function functionFactory() {
    return function () {}; // (A)
}
const foo = functionFactory(); // (B)
console.log(foo.name.length); // 0 (anonymous)
```
하나의 수, 이론적으로, 오른편에 함수를 평가하는지, 함수가 이름을 가지고 있지 않은지를 각 할당을 검사한다. 그러나 아마도 상당한 성능 패널티가 있다.

#### 12.6.2.2 주의: 압축
함수 이름은 압축된 이름 이다. 이 말은 그들은 대게 압축된 코드 안에서 변경된다. 당신이 원하는 무엇인가에 따라 당신은 문자열을 통해 함수 이름을 관리해야 하거나 당신은 당신의 압축에게 이름을 압축하지 말라고 이야기 해야 한다.

### 12.6.3 함수의 이름 변경
name 프로퍼티의 속성이다.
```
> let func = function () {}
> Object.getOwnPropertyDescriptor(func, 'name')
{ value: 'func',
  writable: false,
  enumerable: false,
  configurable: true }
```
이 속성에 쓰기 속성이 없다는건 당신인 할당을 통해 이것을 바꿀 수 없다.
```
> func.name = 'foo';
> func.name
'func'
```
그러나 이 프로퍼티는 configurable이다. 이 말은 당신은 그것을 재정의를 통해 바꿀 수 있다.
```
> Object.defineProperty(func, 'name', {value: 'foo', configurable: true});
> func.name
'foo'
```
만얀 name 프로퍼티가 이미 존재한다면, 당신은 디스크립터 프로퍼티인 configurable을 뺄 수 있다. 왜냐하면 빠진 디스트럽터 프로퍼티는 일치하는 속성은 변하지 않는것을 의미하기 때문이다.

만일 name 프로퍼티가 아직 존재하지 않는다면 디스크럽터 프로퍼티인 configurable은 name은 configurable이 유지하는것을 보장한다(디폴트 속성 값은 모두 false 또는 undefined이다).

### 12.6.4 스팩에서 함수 name 프로퍼티
* 스팩 연산자인 SetFunctionName()는 name프로퍼티를 설정한다. 어디서 발생됐는지 발견하기 위해서 스팩에서 그것의 이름을 찾는다 
  * 연산자중 3번째 파라미터는 이름 접두사를 지정한다. 
    * 이것은 겟터와 셋터(접두사 'get', 'set')인 경우 사용된다.
    * Function.prototype.bind() (접두사 'bound')
* name프로퍼티를 가지고 있지 않은 익명 함수 표현식은 그들의 런타입 시맨틱을 보는것을 통해 보여진다.
  * 기명 함수의 이름은 setFunction()을 통해 설졍된다. 그 연산자는 익명함수 표현식인 경우 호출되지 않는다.
  * 함수 선언의 이름은 스코프에 들어갈때 (호이스트) 설정된다.
  * 애로우 함수는 생성될때 이름 없음으로 설정된다. (setFunctionName() 은 호출되지 않는다.)

## 12.7 FAQ: 호출 가능 개체
### 12.7.1 왜 ES6에서는 "날씬한" 애로우 함수가 아닌 "뚱뚱한" 애로우 함수 (=>)이냐?
ES6은 애로우 함수로 불리는 어휘적 this를 갖는 함수 신택스를 갖는다. 그러나 이것은 동적 this를 갖는 애로우 신택스는 갖지 않는다. 이 생략은 신중하다. 메서드 정의는 대부분의 날씬한 사용 사례를 다룬다. 만약 당신이 진정으로 동적 this가 필요하다면 당신은 여전히 전통적인 함수표현식을 사용 할 수 있다.

### 12.7.2 어떻게 내가 함수가 new를 통해 호출되었는지 아냐?
ES6은 서브클래싱에 대한 새로운 프로토콜을 갖는다. 이것은 15장에서 다룬다. 이 프로토콜의 일부는 meta-property new.target이다. 이것은 생성자 호출 체인의 처음을 가르킨다(슈퍼 메서드 호출에 대한 체인 this와 유사하다.).  만약 생성자 호출을 하지 않았다면,이것은 undefined 이다. 우리는 함수가 new를 통해 호출되는것나 호출 되지 않게 억지로 사용 할 수 있다. 이 예제는 후자다.
```
function realFunction() {
    if (new.target !== undefined) {
        throw new Error('Can’t be invoked via `new`');
    }
    ···
}
```
ES5에서 이것은 대개 흔한 방식이다.
```
function realFunction() {
    "use strict";
    if (this !== undefined) {
        throw new Error('Can’t be invoked via `new`');
    }
    ···
}
```
