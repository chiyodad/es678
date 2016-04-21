## 3. One JavaScript: avoiding versioning in ECMAScript 6

언어에 새로운 features를 추가하기위한 가장 좋은 방법은 무엇인가? 이 챕터에서는 ECMAScript6 에 대한 접근을 설명한다. versioning 을 피해야하기 때문에 이것은 One JavaScript로 불린다. 
What is the best way to add new features to a language? This chapter describes the approach taken by ECMAScript 6. It is called One JavaScript, because it avoids versioning.

### 3.1 Versioning

원론적으로, 언어의 새로운 버전은 오래된 특징을 제거하거나 동작하는 방법을 바꾸기 위한 찬스다. 
In principle, a new version of a language is a chance to clean it up, by removing outdated features or by changing how features work. 
이 말은 곧 새로운 코드가 그 언어의 더 오래된 구현물에서 동작하지 않거나 오래된 코드가 새로운 구현에서 동작하지 않는다는 것을 의미한다.
That means that new code doesn’t work in older implementations of the language and that old code doesn’t work in a new implementation.
각각의 코드 조각들은 언어의 특정 버전에 연결되어 있다. 다른 버전을 다루기 위한 두 가지 접근법이 흔히 사용된다.
Each piece of code is linked to a specific version of the language. Two approaches are common for dealing with versions being different.

첫 째, 코드 베이스가 새로운 버전을 사용길 원한다면 "all or nothing" 접근을 선택 할 수 있다. 이것은 완전히 업그레이드 되어야만 한다. 파이썬은 2에서 3버전으로 올라갈 때 이러한 접근법을 적용했다. 이 방법의 한 가지 문제점은 존재하는 코드베이스의 모든것을 통합하는 것이 실현 불가능 할 수도 있다. 특히 규모가 크다면 더더욱 그렇다. 더군다나 이 방법은 오래된 코드가 항상 있고 자바스크립트 엔진이 자동으로 업데이트되는 웹에서 선택 가능한 방법은 아니다. 
First, you can take an “all or nothing” approach and demand that, if a code base wants to use the new version, it must be upgraded completely. Python took that approach when upgrading from Python 2 to Python 3. A problem with it is that it may not be feasible to migrate all of an existing code base at once, especially if it is large. Furthermore, the approach is not an option for the web, where you’ll always have old code and where JavaScript engines are updated automatically.

둘 째, 코드에 버전을 명시하는 것으로써 여러가지 버전에서의 코드를 포함하는 코드베이스를 허용 할 수도 있다. 웹에서는, 전용 인터넷 미디어 타입을 통한 ECMAScript 6 코드를 태그할 수도 있다. 미디어 타입은 HTTP header 를 통한 파일과 연관된 것일 수 있다.
Second, you can permit a code base to contain code in multiple versions, by tagging code with versions. On the web, you could tag ECMAScript 6 code via a dedicated Internet media type. Such a media type can be associated with a file via an HTTP header:

Content-Type: application/ecmascript;version=6

이것은 스크립트 태그의 타입 속성을 통해 관련될수도 있다. (기본값은 text/javascript) :
It can also be associated via the type attribute of the <script> element (whose default value is text/javascript):

<script type="application/ecmascript;version=6">
    ···
</script>

이것은 외부적으로 실제 컨텐츠에 the version out of band를 명시한다. 또 다른 옵션은 컨텐츠 내부에 버전을 명시하는 것이다.(in-band). 예를 들어 다음의 예처럼 파일을 시작하는것에 의해 :
This specifies the version out of band, externally to the actual content. Another option is to specify the version inside the content (in-band). For example, by starting a file with the following line:

use version 6;

태깅하는 두 가지 방법 모두 문제점이 있다 : out-of-band versions는 잘 깨지고 잃어버리기 쉽고 in-band 버전은 코드를 지저분하게 한다. 
Both ways of tagging are problematic: out-of-band versions are brittle and can get lost, in-band versions add clutter to code.

더욱 근본적인 이슈는, 코드 베이스 마다 언어를 효과적으로 fork하   여러 버전을 허용하는 것 .....후..... 동시에 유지보수되어야만 하는 서브 랭귀지  후... 나의 번역은 여기까지 인가...
A more fundamental issue is that allowing multiple versions per code base effectively forks a language into sub-languages that have to be maintained in parallel. This causes problems:

엔진들은 모든 버전의 의미해석을 구현해야하기 때문에 비대해진다. 언어분석 툴에도 똑같이 적용된다.
Engines become bloated, because they need to implement the semantics of all versions. The same applies to tools analyzing the language (예: JSLint) (e.g. style checkers such as JSLint).
프로그래머는 버전들이 어떻게 다른지 기억해야한다.
Programmers need to remember how the versions differ.
코드는 더욱 리팩토링하기 어려워진다. 왜냐하면 
Code becomes harder to refactor, because you need to take versions into consideration when you move pieces of code.

그러므로 versioning 은 피해야하는 것이다. 특히 JavaScript와 웹을 위해서 말이다. 
Therefore, versioning is something to avoid, especially for JavaScript and the web.

### 3.1.1 versioning이 없는 점진적 발전 
### 3.1.1 Evolution without versioning 
그러나 어떻게 versioning을 제거 할 수 있을까? 항상 이전 버전과 호환이 되어야하는데. 우리는 자바스크립트를 정화하는 그런 야망을 포기해야만 한다. 우리는 breaking changes를 소개 할 수 없다. 이전 버전과 호환이 된다는 것은 이전 버전의 특성을 바꾸거나 지우지 않는 것을 의미한다. 이 원칙에 의거한 슬로건이 바로 "dont' break the wb" 이다.

But how can we get rid of versioning? By always being backwards-compatible. That means we must give up some of our ambitions w.r.t. cleaning up JavaScript: We can’t introduce breaking changes. Being backwards-compatible means not removing features and not changing features. The slogan for this principle is: “don’t break the web”.

We can, however, add new features and make existing features more powerful.

이러한 결과로써, 새로운 엔진을 위한 버전은 필요가 없다. 엔진은 여전히 오래된 모든 코드를 실행할 수 있기 때문에 말이다. David Herman 은 versioning을 피하기 위한 접근을 One JavaScript (1JS) 을 부른다. 그렇게 부르는 이유는 자바스크립트가 다른 버전이나 다른 모드로 분열되는 것을 피할 수 있기 때문이다. 나중에 보게 되겠지만, 1JS는 stric mode 로 인해 이미 분열된 것들을 다시 원상태로 돌리기도 한다.
As a consequence, no versions are needed for new engines, because they can still run all old code. David Herman calls this approach to avoiding versioning One JavaScript (1JS) [1], because it avoids splitting up JavaScript into different versions or modes. As we shall see later, 1JS even undoes some of a split that already exists, due to strict mode.

One JavaScript 는 우리가 언어를 cleaning up 하는 걸 완전히 포기해야만 한다는 의미는 아니다. 이미 존재하는 특징들을 없애는 대신에 새롭고 클린한 특징을 접하게 된다. 그러한 예의 한 가지로써 let 을 들 수 있다. 이것은 var 의 향상된 버전이며 블록 스코프 변수를 선언한다. 그러나 이것이 var를 대체하는 것은 아니다. let 은 더 상위 옵션으로써 var 와 나란히 존재한다.
One JavaScript does not mean that you have to completely give up on cleaning up the language. Instead of cleaning up existing features, you introduce new, clean, features. One example for that is let, which declares block-scoped variables and is an improved version of var. It does not, however, replace var, it exists alongside it, as the superior option.

더 이상 아무도 사용하지 않는 특징들은 언젠가 제거 될 수도 있다. ES6 특징의 몇 가지는 웹에서의 자바스크립트 코드의 측정에 의해 고안되었다. 다음의 두 가지 예제가 있다.
One day, it may even be possible to eliminate features that nobody uses, anymore. Some of the ES6 features were designed by surveying JavaScript code on the web. Two examples are:

let 선언은 non-strict 모드에 추가하기는 어렵다. 왜냐하면 non-strict 모드에서 let은 에약어가 아니기 때문이다. ES5 코드로 유효한 let의 변형은 다음과 같다.
let-declarations are difficult to add to non-strict mode, because let is not a reserved word in that mode. The only variant of let that looks like valid ES5 code is:
```javascript
  let[x] = arr;
```

Research yielded that no code on the web uses a variable let in non-strict mode in this manner. That enabled TC39 to add let to non-strict mode. 자세한 내용은 이 챕터 후반부에 설명한다. Details of how this was done are described later in this chapter.

함수 선언은 때때로 non-strict blocks에서 나타난다. 
Function declarations do occasionally appear in non-strict blocks, which is why the ES6 specification describes measures that web browsers can take to ensure that such code doesn’t break. Details are explained later.

###3.2 Strict 모드 그리고 ECMAScript6
###3.2 Strict mode and ECMAScript 6
Strict 모드는 ECMAScript 5에서 언어를 더욱 클린업 시키기 위해 소개되었다. 파일이나 함수의 첫 번째 줄에 아래와 같이 추가하면 strict 모드로 전환된다.
Strict mode was introduced in ECMAScript 5 to clean up the language. It is switched on by putting the following line first in a file or in a function:

```javascript
'use strict';
```
Strict 모드는 3가지 braking cahnges를 소개한다.
Strict mode introduces three kinds of breaking changes:

문법적 변화 : 이전의 몇가지 유효한 문법이 스트릭트 모드에서는 금지된다. 예:
Syntactic changes: some previously legal syntax is forbidden in strict mode. For example:

with 문은 금지된다. with문은 사용자가 임의의 객체를 변수 스코프 체인에 추가하게 한다. 이것은 실행을 느리게 하고 변수 참조를 알아보기 어렵게 만든다. 
The with statement is forbidden. It lets users add arbitrary objects to the chain of variable scopes, which slows down execution and makes it tricky to figure out what a variable refers to.
자격이 없는 식별자(객체의 속성이 아닌 변수)를 지우는것은 금지된다.
Deleting an unqualified identifier (a variable, not a property) is forbidden.
함수는 오직 스코프의 최상단에만 선언될 수 있다.
Functions can only be declared at the top level of a scope.
더 많은 식별자들이 생겼다 : implements interface let package private protected public static yield
More identifiers are reserved: implements interface let package private protected public static yield
더 많은 오류. 예 :
More errors. For example:
선언되지 않는 변수에 할당하는 것은 ReferenceError 를 발생시킨다. non-strict mode 에서는 이와 같은 경웨 전역 변수가 생성된다.
Assigning to an undeclared variable causes a ReferenceError. In non-strict mode, a global variable is created in this case.
읽기전용 속성을 변경하는 것(string 의 length 변경 같은 것) 은 typeError 을 유발한다. non-strict mode 에서는 아무런 영향을 미치지 않는다.
Changing read-only properties (such as the length of a string) causes a TypeError. In non-strict mode, it simply has no effect.
다른 의미들: 어떤 것들은 스트릭트 모드에서 다르게 동작한다. 예 :
Different semantics: Some constructs behave differently in strict mode. For example:
arguments 는 더 이상 현재 매개변수의 값들을 추적하지 않는다.
arguments doesn’t track the current values of parameters, anymore.
메소드가 아닌 함수에서 arguments는 undefined 이다. non-strict모드에서 이것은 전역변수를 참조한다. 
this is undefined in non-method functions. In non-strict mode, it refers to the global object (window), which meant that global variables were created if you called a constructor without new.
Strict mode is a good example of why versioning is tricky: Even though it enables a cleaner version of JavaScript, its adoption is still relatively low. The main reasons are that it breaks some existing code, can slow down execution and is a hassle to add to files (let alone interactive command lines). I love the idea of strict mode and don’t nearly use it often enough.

### 3.2.1 Supporting sloppy (non-strict) mode
One JavaScript means that we can’t give up on sloppy mode: it will continue to be around (e.g. in HTML attributes). Therefore, we can’t build ECMAScript 6 on top of strict mode, we must add its features to both strict mode and non-strict mode (a.k.a. sloppy mode). Otherwise, strict mode would be a different version of the language and we’d be back to versioning. Unfortunately, two ECMAScript 6 features are difficult to add to sloppy mode: let declarations and block-level function declarations. Let’s examine why that is and how to add them, anyway.

### 3.2.2 let declarations in sloppy mode
let enables you to declare block-scoped variables. It is difficult to add to sloppy mode, because let is only a reserved word in strict mode. That is, the following two statements are legal ES5 sloppy code:

var let = [];
let[x] = 'abc';
In strict ECMAScript 6, you get an exception in line 1, because you are using the reserved word let as a variable name. And the statement in line 2 is interpreted as a let variable declaration (that uses destructuring).

In sloppy ECMAScript 6, the first line does not cause an exception, but the second line is still interpreted as a let declaration. This way of using the identifier let is so rare on the web that ES6 can afford to make this interpretation. Other ways of writing let declarations can’t be mistaken for sloppy ES5 syntax:

let foo = 123;
let {x,y} = computeCoordinates();
3.2.3 Block-level function declarations in sloppy mode
ECMAScript 5 strict mode forbids function declarations in blocks. The specification allowed them in sloppy mode, but didn’t specify how they should behave. Hence, various implementations of JavaScript support them, but handle them differently.

ECMAScript 6 wants a function declaration in a block to be local to that block. That is OK as an extension of ES5 strict mode, but breaks some sloppy code. Therefore, ES6 provides “web legacy compatibility semantics” for browsers that lets function declarations in blocks exist at function scope.

### 3.2.4 Other keywords
The identifiers yield and static are only reserved in ES5 strict mode. ECMAScript 6 uses context-specific syntax rules to make them work in sloppy mode:

In sloppy mode, yield is only a reserved word inside a generator function.
static is currently only used inside class literals, which are implicitly strict (see below).
3.2.5 Implicit strict mode
The bodies of modules and classes are implicitly in strict mode in ECMAScript 6 – there is no need for the 'use strict' marker. Given that virtually all of our code will live in modules in the future, ECMAScript 6 effectively upgrades the whole language to strict mode.

The bodies of other constructs (such as arrow functions and generator functions) could have been made implicitly strict, too. But given how small these constructs usually are, using them in sloppy mode would have resulted in code that is fragmented between the two modes. Classes and especially modules are large enough to make fragmentation less of an issue.

### 3.2.6 Things that can’t be fixed
The downside of One JavaScript is that you can’t fix existing quirks, especially the following two.

First, typeof null should return the string 'null' and not 'object'. TC39 tried fixing it, but it broke existing code. On the other hand, adding new results for new kinds of operands is OK, because current JavaScript engines already occasionally return custom values for host objects. One example are ECMAScript 6’s symbols:

> typeof Symbol.iterator
'symbol'
Second, the global object (window in browsers) shouldn’t be in the scope chain of variables. But it is also much too late to change that now. At least, one won’t be in global scope in modules and let never creates properties of the global object, not even when used in global scope.

### 3.3 Breaking changes in ES6
ECMAScript 6 does introduce a few minor breaking changes (nothing you’re likely to encounter). They are listed in two annexes:

Annex D: Corrections and Clarifications in ECMAScript 2015 with Possible Compatibility Impact
Annex E: Additions and Changes That Introduce Incompatibilities with Prior Editions
### 3.4 Conclusion
One JavaScript means making ECMAScript 6 completely backwards compatible. It is great that that succeeded. Especially appreciated is that modules (and thus most of our code) are implicitly in strict mode.

In the short term, adding ES6 constructs to both strict mode and sloppy mode is more work when it comes to writing the language specification and to implementing it in engines. In the long term, both the spec and engines profit from the language not being forked (less bloat etc.). Programmers profit immediately from One JavaScript, because it makes it easier to get started with ECMAScript 6.

### 3.5 Further reading
[1] The original 1JS proposal (warning: out of date): “ES6 doesn’t need opt-in” by David Herman.

