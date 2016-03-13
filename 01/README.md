# 1. ECMAScript 6에 대하여 (ES6)
1. About ECMAScript 6 (ES6)

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

1.2.1 The design process after ES6
Starting with ECMAScript 7 (whose official name is ECMAScript 2016), TC39 will time-box releases. The plan is to release a new version of ECMAScript every year, with whatever features are ready at that time. That means that from now on, ECMAScript versions will be relatively small upgrades.

Work on ECMAScript 2016 (and later) has already begun, current proposals are listed on GitHub. The process has changed, too and is described in a TC39 process document.

1.3 JavaScript versus ECMAScript
JavaScript is what everyone calls the language, but that name is trademarked (by Oracle, which inherited the trademark from Sun). Therefore, the official name of JavaScript is ECMAScript. That name comes from the standards organization Ecma, which manages the language standard. Since ECMAScript’s inception, the name of the organization has changed from the acronym “ECMA” to the proper name “Ecma”.

Versions of JavaScript are defined by specifications that carry the official name of the language. Hence, the first standard version of JavaScript is ECMAScript 1 which is short for “ECMAScript Language Specification, Edition 1”. ECMAScript x is often abbreviated ESx.

1.4 Upgrading to ES6
The stake holders on the web are:

Implementors of JavaScript engines
Developers of web applications
Users
These groups have remarkably little control over each other. That’s why upgrading a web language is so challenging.

On one hand, upgrading engines is challenging, because they are confronted with all kinds of code on the web, sometimes very old one. You also want engine upgrades to be automatic and unnoticeable for users. Therefore, ES6 is a superset of ES5, nothing is removed1. ES6 upgrades the language without introducing versions or modes. It even manages to make strict mode the de-facto default (via modules), without increasing the rift between it and sloppy mode. The approach that was taken is called “One JavaScript” and explained in a separate chapter.

On the other hand, upgrading code is challenging, because your code must run on all JavaScript engines that are used by your target audience. Therefore, if you want to use ES6 in your code, you only have two choices: You can either wait until no one in your target audience uses a non-ES6 engine, anymore. That will take years; mainstream audiences were at that point w.r.t. ES5 when ES6 became a standard in June 2015. And ES5 was standardized in December 2009! Or you can compile ES6 to ES5 and use it now. More information on how to do that is given in the book “Setting up ES6”, which is free to read online.

Goals and requirements clash in the design of ES6:

Goals are fixing JavaScript’s pitfalls and adding new features.
Requirements are that both need to be done without breaking existing code and without changing the lightweight nature of the language.
1.5 Goals for ES6
The original project page for Harmony/ES6 includes several goals. In the following subsections, I’m taking a look at some of them.

1.5.1 Goal: Be a better language
The goal is: Be a better language for writing:

complex applications;
libraries (possibly including the DOM) shared by those applications;
code generators targeting the new edition.
Sub-goal (i) acknowledges that applications written in JavaScript have grown huge. A key ES6 feature fulfilling this goal is built-in modules.

Modules are also an answer to goal (ii). As an aside, the DOM is notoriously difficult to implement in JavaScript. ES6 Proxies should help here (as described in a separate chapter).

Several features were specifically added not to improve JavaScript, but to make it easier to compile to JavaScript. Two examples are:

Math.fround() – rounding Numbers to 32 bit floats
Math.imul() – multiplying two 32 bit ints
They are both useful for, e.g., compiling C/C++ to JavaScript via Emscripten.

1.5.2 Goal: Improve interoperation
The goal is: Improve interoperation, adopting de facto standards where possible.

Examples are:

Classes: are based on how constructor functions are currently used.
Modules: picked up design ideas from the CommonJS module format.
Arrow functions: have syntax that is borrowed from CoffeeScript.
Named function parameters: There is no built-in support for named parameters. Instead, the existing practice of naming parameters via object literals is supported via destructuring in parameter definitions.
1.5.3 Goal: Versioning
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
