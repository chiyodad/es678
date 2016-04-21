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

더욱 근본적인 이슈는, 코드 베이스 마다 언어를 효과적으로 fork하   여러 버전을 허용하는 것 .....후..... 동시에 유지보수되어야만 하는 서브 랭귀지 
A more fundamental issue is that allowing multiple versions per code base effectively forks a language into sub-languages that have to be maintained in parallel. This causes problems:

Engines become bloated, because they need to implement the semantics of all versions. The same applies to tools analyzing the language (e.g. style checkers such as JSLint).
Programmers need to remember how the versions differ.
Code becomes harder to refactor, because you need to take versions into consideration when you move pieces of code.
Therefore, versioning is something to avoid, especially for JavaScript and the web.

### 3.1.1 Evolution without versioning
But how can we get rid of versioning? By always being backwards-compatible. That means we must give up some of our ambitions w.r.t. cleaning up JavaScript: We can’t introduce breaking changes. Being backwards-compatible means not removing features and not changing features. The slogan for this principle is: “don’t break the web”.

We can, however, add new features and make existing features more powerful.

As a consequence, no versions are needed for new engines, because they can still run all old code. David Herman calls this approach to avoiding versioning One JavaScript (1JS) [1], because it avoids splitting up JavaScript into different versions or modes. As we shall see later, 1JS even undoes some of a split that already exists, due to strict mode.

One JavaScript does not mean that you have to completely give up on cleaning up the language. Instead of cleaning up existing features, you introduce new, clean, features. One example for that is let, which declares block-scoped variables and is an improved version of var. It does not, however, replace var, it exists alongside it, as the superior option.

One day, it may even be possible to eliminate features that nobody uses, anymore. Some of the ES6 features were designed by surveying JavaScript code on the web. Two examples are:

let-declarations are difficult to add to non-strict mode, because let is not a reserved word in that mode. The only variant of let that looks like valid ES5 code is:
  let[x] = arr;
Research yielded that no code on the web uses a variable let in non-strict mode in this manner. That enabled TC39 to add let to non-strict mode. Details of how this was done are described later in this chapter.

Function declarations do occasionally appear in non-strict blocks, which is why the ES6 specification describes measures that web browsers can take to ensure that such code doesn’t break. Details are explained later.
3.2 Strict mode and ECMAScript 6
Strict mode was introduced in ECMAScript 5 to clean up the language. It is switched on by putting the following line first in a file or in a function:

'use strict';
Strict mode introduces three kinds of breaking changes:

Syntactic changes: some previously legal syntax is forbidden in strict mode. For example:
The with statement is forbidden. It lets users add arbitrary objects to the chain of variable scopes, which slows down execution and makes it tricky to figure out what a variable refers to.
Deleting an unqualified identifier (a variable, not a property) is forbidden.
Functions can only be declared at the top level of a scope.
More identifiers are reserved: implements interface let package private protected public static yield
More errors. For example:
Assigning to an undeclared variable causes a ReferenceError. In non-strict mode, a global variable is created in this case.
Changing read-only properties (such as the length of a string) causes a TypeError. In non-strict mode, it simply has no effect.
Different semantics: Some constructs behave differently in strict mode. For example:
arguments doesn’t track the current values of parameters, anymore.
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

