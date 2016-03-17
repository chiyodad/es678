# 1. ECMAScript 6에 대하여 (ES6)

그것 끝내기 위해서 오랜 시간이 걸렸지만 자바스크립트 다음 버전인 ECMAScript 6은 마침내 현실이 되었다.:

* 이것은 2015년 6월 17일 표준이 되었다.
* 그 기능은 천천히 자바스크립트 엔진에서 나타나고 있다. (kangax의 ES6 호환 테이블 문서 에서)
* 트렌스파일러(바벨, Traceur)은 당신의 ES5코드를 ES6으로 컴파일 해준다.
다음 섹션은 ES6의 세계에서 중요한 개념을 설명한다.

## 1.1 TC39 (Ecma Technical Committee 39)
TC39 (Ecma Technical Committee 39)은 자바스크립트를 발달시키는 위원회이다. 그 회원은 회사(주요 브라우져 벤더들)들이다. TC39는 정기적으로 모이고 그 회의는 회원들이 보낸 대리인들과 초대된 전문가들로 참석되어 진다. 그 회의의 의사록은 온라인에서 볼 수 있고, 당신의 TC39 작동하는 방식의 생각을 당신에게 제공한다.

## 1.2 ECMAScript 6은 어떻게 설계 되었나
ECMAScript 6 설계는 기능을 위한 제안의 센터로 저장된다. 제안들은 개발자 커뮤니티로 부터 요청에 의해 종종 유발되어졌다. 의원회에 의한 설계를 피하기 위해서 제안은 챔피언(1~2명 의원회 대표)들을 통해 유지되어진다.

제안은 표준안이 되기 위에 다음과 같은 과정을 겪는다.
* 초고 (비공식적: "strawman 제안"): 제안된 기능을 처음 기술
* 제안: 만약 TC39가 기능이 중요하다고 동의하면, 그것은 공식 제안 상태로 오른다. 이것이 표준이 된다는것을 보장하지 않지만 이 제안한 상당히 큰 가능성이 있다. ES6 제안의 마지막 기한은 2011년 5월 이였다. 그 이후에 더이상의 주요 제안은 고려되어 지지 않는다.
* 구현: 이상적으로 두 자바스크립트 엔진에서 제안된 기능은 반드시 구현되어진다. 구현과 커뮤티니로부터으이 피드백은 그 제안을 진화 처럼 구체화 한다.
* 표준: 만약 제안이 계속적으로 스스로를 증명하고 TC39에 의해서 받아드려진다면, 그것은 ECMAScript 표준의 판안에 결국 추가된다. 그 시점에서 제안은 표준기능이다.

[Source of this section: “The Harmony Process” by David Herman.]

### 1.2.1 ES6이후 설계 과정
ESMAScript 7은 시작되었고(공식적인 이름은 ECMAScript 2016), TC39는 타입박스를 공개할 것이다. 이 계획은 ECMAScript의 그때 준비된 기능과 함께 매년 새로운 버전은 공개되는 것이다. 이것은 지금부터 ECMAScript 버전들은 비교적 작은 업데이트라는것을 의미 한다.

ECMAScript 2016(그 이후) 작업은 이미 시작되었고, 현재 제안은 깃헙에 리스트 되어 있다. 그 과정도 변경 되고 있으며, TC39 프로세서 문서에 기술되고 있다.

## 1.3 자바스크립트 대 ECMAScript
자바스크립트는 누구에게나 프로그램 언어로 불리지만 그 이름은 트레이드마크(오라클에 의한,sun으로 부터 받은 트레이드마크)다. 그러므로, 공식적인 자바스크립트 이름은 ECMAscript이다. 이 이름은 언어의 표준을 관리하는 표준 조직인 Ecma로 부터 왔다. ECMAscript 시작 이래로 조직의 이름은 약어 "ECMA"로 부터 고유명사인 "Ecma"로 변경되었다.

자바스크립트 버전은 언어의 공식적 이름을 수행하는 명세에 의해서 정의 된다. 따라서 처음 자바스크립트 표준 버전은 "ECMAScript Language Specification, Edition 1"를 줄여서 ECMAScript 1이다. ECMAScript X는 종종 ESX로 줄여 부른다.

## 1.4 ES6으로 업그레이드
웹 이해 관계자들:
* 자바스크립트 엔진 구현자
* 웹 애플리케이션 개발자
* 사용자

이 그룹들은 서로 매우 작은 제어를 가진다. 그것이 왜 웹언어 업그레이드가 매우 도전적인 이유이다.

한편 엔진을 업그레이드하는 것은 도전이다. 왜냐하면 그들은 모든 종류의 웹코드와 때때로 매우 오래된 코드에 직면해 있기 때문이다. 당신 역시 자연스럽고 사용자가 인지하지 못하게 엔진 업그레이드되길 원한다. 그러므로 ES6은 ES5의 상위 집합으로 아무것도 제거 된 것이 없다. ES6는 언어를 버전이나 모드를 도입하지 않고 업그래이드 한다. 이것은 심지어 슬리피 모드와의 틈 없이 엄격모드를 사실상 기본값으로 관리 한다. 이 "하나의 자바스크립트"로 불리는 접근는 별도에 장에서 설명한다.

반면에 코드를 업그레이드 하는것은 도전이다. 왜냐하면 당신의 코드는 반드시 당신이 타겟으로 하는 사용자가 사용하는 모든 자바스크립트 엔진에서 돌아야 하기 때문이다. 그러므로 만약 당신이 ES6을 당신의 코드에서 사용하길 원한다면 당신은 단지 두가지 선택이 있다. 당신은 당신의 타겟 사용자가 ES6를 지원하지 않는 엔진을 더이상 사용하지 않을 때까지 기다릴 수 있다. 그것은 몇 해 걸릴 것이다. 주요 ES5 2009년 11월 표준화 되었다.! 또는 당신은 ES6을 ES5로 컴파일 할 수 있다. 이 작업을 수행하는 자세한 방법은 "Setting up ES6"에 나와 있고 이것은 온라인에서 공짜다.

ES6 설계에서 목표 와 요구사항의 충돌

* 목표는 자바스크립트의 함정을 수정하고 새로운 기능을 추가
* 요구 사항은 존재하는 코드의 깨짐이나 언어의 경량특성을 변경 없이 되는 것. 

## 1.5 ES6의 목표
하모니/ES6의 원래 프로젝트 페이지는 몇몇개의 목표를 가진다. 다음 소단원에서 나는 목표의 몇개를 설명하겠다.

### 1.5.1 목표: 더나은 언어

목표는 쓰기에 더 나은 언어:

1. 복잡한 어플리케이션;
2. 애플리케이션에 의해 공유된 라이브러리(아마도 DOM을 포함);
3. 새 판을 겨냥한 코드 생성기.

소 목적 (1) 자바스크립트로 쓰여진 애플리케이션 사례는 크게 증가되었다. 목표를 실현하는 ES6 기능의 키는 내장 모듈이다.

모듈들은 또한 목표(2)의 해답이다. 반면에 DOM은 자바스크립트에서 구현하기 어렵기로 악명이 높다. ES6 프록시는 이것을 도와준다(이 쳅터에서 설명).

몇몇의 기능들은 자바스크립트 개선이 아닌 특별하게 추가 되었다. 그러나 이 기능은 자바스크립트로 컴파일을 쉽게 해준다. 두가지 예를 보면:

```
Math.fround() – rounding Numbers to 32 bit floats
Math.imul() – multiplying two 32 bit ints
```

이것들은 예를 들면 C/C++을 Emscripten을 통해 자바스크립트로 컴파일 할때 유용하다. 

### 1.5.2 목표: 상호 개선 
이 목표는 상호 개선, 가능한 사실상의 표준을 채택이다.
예를 들면:

* Classes: 는 생성자 함수들이 현재 사용되는 방법을 기반한다.
* Modules: 은 CommonJS 모델 포멧으로 부터 설계 아이디어를 가져왔다.
* Arrow functions: 은 CoffeeScript로 부터 빌려온 문법을 가진다.
* Named function parameters: 기명 파라미터에 대한 지원이 내장되어 있지 않다. 대신에 객체 리터럴를 통한 기명 파라미터의 기존 관행은 파라미터 정의 시 해체를 통해 제공된다.
### 1.5.3 목표: 버져닝
The goal is: Keep versioning as simple and linear as possible.

As mentioned previously, ES6 avoids versioning via “One JavaScript”: In an ES6 code base, everything is ES6, there are no parts that are ES5-specific.

1.6 An overview of ES6 features
Quoting the introduction of the ECMAScript 6 specification:

Some of [ECMAScript 6’s] major enhancements include modules, class declarations, lexical block scoping, iterators and generators, promises for asynchronous programming, destructuring patterns, and proper tail calls. The ECMAScript library of built-ins has been expanded to support additional data abstractions including maps, sets, and arrays of binary numeric values as well as additional support for Unicode supplemental characters in strings and regular expressions. The built-ins are now extensible via subclassing.

There are three major groups of features:

Better syntax for features that already exist (e.g. via libraries). For example:
Classes
Modules
New functionality in the standard library. For example:
New methods for strings and Arrays
Promises
Maps, Sets
Completely new features. For example:
Generators
Proxies
WeakMaps
1.7 A brief history of ECMAScript
This section describes what happened on the road to ECMAScript 6.

1.7.1 The early years: ECMAScript 1–3
ECMAScript 1 (June 1997) was the first version of the JavaScript language standard.
ECMAScript 2 (June 1998) contained minor changes, to keep the spec in sync with a separate ISO standard for JavaScript.
ECMAScript 3 (December 1999) introduced many features that have become popular parts of the language2:
1.7.2 ECMAScript 4 (abandoned in July 2008)
Work on ES4 started after the release of ES3 in 1999. In 2003, an interim report was released after which work on ES4 paused. Subsets of the language described in the interim report were implemented by Adobe (in ActionScript) and by Microsoft (in JScript.NET).

In February 2005, Jesse James Garrett observed that new techniques had become popular for implementing dynamic frontend apps in JavaScript. He called those techniques Ajax. Ajax enabled a completely new class of web apps and led to a surge of interest in JavaScript.

That may have contributed to TC39 resuming work on ES4 in fall 2005. They based ES4 on ES3, the interim ES4 report and experiences with ActionScript and JScript.NET.

There were now two groups working on future ECMAScript versions:

ECMAScript 4 was designed by Adobe, Mozilla, Opera, and Google and was a massive upgrade. Its planned feature sets included:
Programming in the large (classes, interfaces, namespaces, packages, program units, optional type annotations, and optional static type checking and verification)
Evolutionary programming and scripting (structural types, duck typing, type definitions, and multimethods)
Data structure construction (parameterized types, getters and setters, and meta-level methods)
Control abstractions (proper tail calls, iterators, and generators)
Introspection (type meta-objects and stack marks)
ECMAScript 3.1 was designed by Microsoft and Yahoo. It was planned as a subset of ES4 and an incremental upgrade of ECMAScript 3, with bug fixes and minor new features. ECMAScript 3.1 eventually became ECMAScript 5.
The two groups disagreed on the future of JavaScript and tensions between them continued to increase.

Sources of this section:
“Proposed ECMAScript 4th Edition – Language Overview”. 2007-10-23
“ECMAScript Harmony” by John Resig. 2008-08-13
1.7.3 ECMAScript Harmony
At the end of July 2008, there was a TC39 meeting in Oslo, whose outcome was described as follows by Brendan Eich:

It’s no secret that the JavaScript standards body, Ecma’s Technical Committee 39, has been split for over a year, with some members favoring ES4 […] and others advocating ES3.1 […]. Now, I’m happy to report, the split is over.

The agreement that was worked out at the meeting consisted of four points:

Develop an incremental update of ECMAScript (which became ECMAScript 5).
Develop a major new release, which was to be more modest than ECMAScript 4, but much larger in scope than the version after ECMAScript 3. This version was code-named Harmony, due to the nature of the meeting in which it was conceived.
Features from ECMAScript 4 that would be dropped: packages, namespaces, early binding.
Other ideas were to be developed in consensus with all of TC39.
Thus: The ES4 group agreed to make Harmony less radical than ES4, the rest of TC39 agreed to keep moving things forward.

The next versions of ECMAScript are:

ECMAScript 5 (December 2009). This is the version of ECMAScript that most browsers support today. It brings several enhancements to the standard library and updated language semantics via a strict mode.
ECMAScript 5.1 (June 2011). ES5 was submitted as an ISO standard. In the process, minor corrections were made. ES5.1 contains those corrections, it is the same text as ISO/IEC 16262:2011.
ECMAScript 6 (June 2015). This version went through several name changes:
ECMAScript Harmony: was the initial code name for JavaScript improvements after ECMAScript 5.
ECMAScript.next: It became apparent that the plans for Harmony were too ambitious for a single version, so its features were split into two groups: The first group of features had highest priority and was to become the next version after ES5. The code name of that version was ECMAScript.next, to avoid prematurely comitting to a version number, which proved problematic with ES4. The second group of features had time until after ECMAScript.next.
ECMAScript 6: As ECMAScript.next matured, its code name was dropped and everybody started to call it ECMAScript 6.
ECMAScript 2015: In late 2014, TC39 decided to change the official name of ECMAScript 6 to ECMAScript 2015, in light of upcoming yearly spec releases. However, given how established the name “ECMAScript 6” already is and how late TC39 changed their minds, I expect that that’s how everybody will continue to refer to that version.
ECMAScript 2016 was previously called ECMAScript 7. Starting with ES2016, the language standard will see smaller yearly releases.
