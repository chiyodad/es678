# 24. 비동기 프로그래밍 (백그라운드) (Asynchronous programming (background))

이번 챕터는 자바스크립트의 비동기 프로그래밍의 기초를 알아본다. 이 지식은 다음 장의 ES6 Promise 에 도움이 될 것이다

<sub>This chapter explains foundations of asynchronous programming in JavaScript. It provides background knowledge for the next chapter on ES6 Promises.</sub>

## 24.1 자바스크립트의 호출 스택 (The JavaScript call stack)

함수 f 가 함수 g 를 호출할 때, g는 완료 후 (f 내부의) 돌아갈 곳을 알 필요가 있다. 이 정보는 보통 스택으로 관리되는, 호출 스택이다. 예제를 보자

<sub>When a function f calls a function g, g needs to know where to return to (inside f) after it is done. This information is usually managed with a stack, the call stack. Let’s look at an example.</sub>

```javascript
function h(z) {
    // 스택을 찍어보자
    console.log(new Error().stack); // (A)
}
function g(y) {
    h(y + 1); // (B)
}
function f(x) {
    g(x + 1); // (C)
}
f(3); // (D)
return; // (E)
```

처음에는 위의 프로그램이 시작될 때 호출 스택은 비었다. D 라인의 함수 호출 f(3) 이 된 후 스택은 하나의 엔트리를 가진다.

<sub>Initially, when the program above is started, the call stack is empty. After the function call f(3) in line D, the stack has one entry:</sub>

- 전역 스코프안의 위치 (Location in global scope)

C 라인의 g(x + 1) 이 호출된 뒤 스택은 두개의 엔트리를 가진다.

<sub>After the function call g(x + 1) in line C, the stack has two entries:</sub>

- f 안에 위치 (Location in f)
- 전역 스코프안의 위치 (Location in global scope)

B 라인의 h(y + 1) 이 호출된 뒤 스택은 세개의 엔트리를 가진다.

<sub>After the function call h(y + 1) in line B, the stack has three entries:</sub>

- g 안에 위치 (Location in g)
- f 안에 위치 (Location in f)
- 전역 스코프안의 위치 (Location in global scope)

A 라인에서 보여지는 호출 스택의 스택 트레이스는 다음과 같다.

<sub>The stack trace printed in line A shows you what the call stack looks like:</sub>

```javascript
Error
    at h (stack_trace.js:2:17)
    at g (stack_trace.js:6:5)
    at f (stack_trace.js:9:5)
    at <global> (stack_trace.js:11:1)
```

다음에 각각 함수의 종료마다, 스택의 탑 엔트리에서 지워진다. f 가 종료되면 우리는 다시 글로벌 스코프로 돌아가며, 호출 스택은 빈다. 라인 E 에서 리턴하고 스택은 빈다. 그것은 프로그램의 종료를 의미한다.

<sub>Next, each of the functions terminates and each time the top entry is removed from the stack. After function f is done, we are back in global scope and the call stack is empty. In line E we return and the stack is empty, which means that the program terminates.</sub>

## 24.2 브라우저 이벤트 루프 (The browser event loop)

심플하게도 각 브라우저는 탭마다 하나의 이벤트 루프 프로세스 로 돌아간다. 이 루프는 task 큐에 의해 공급되는 브라우저에 관련된 일(소위 task 라고 하는) 을 실행한다.

<sub>Simplifyingly, each browser tab runs (in) a single process: the event loop. This loop executes browser-related things (so-called tasks) that it is fed via a task queue. Examples of tasks are:</sub>

- HTML 파싱 <sub>(Parsing HTML)</sub>
- script 엘리먼트 안의 자바스크립트 코드 실행 <sub>(Executing JavaScript code in script elements)</sub>
- 사용자 입력에 반응(마우스 클릭, 키 입력 등) <sub>(Reacting to user input (mouse clicks, key presses, etc.))</sub>
- 비동기 네트워크 요청 결과 처리 <sub>(Processing the result of an asynchronous network request)</sub>
 
2-4 항목은 브라우저 안의 엔진에 의한 자바스크립트 코드 실행 task 이다. 그것들은 코드 종료시에 종료된다. 그뒤 queue 에서 다음 task 를 실행할 수 있다. 다음 다이어그램은 (필립 로버츠에게 영감을 얻은) 모든 연결된 메커니즘에 대한 개요를 제공한다.

<sub>Items 2–4 are tasks that run JavaScript code, via the engine built into the browser. They terminate when the code terminates. Then the next task from the queue can be executed. The following diagram (inspired by a slide by Philip Roberts [1]) gives an overview of how all these mechanisms are connected.</sub>

![async-stack](./async----event_loop.jpg)

이벤트 루프는 병렬로 실행되는 다른 프로세스로 둘러싸여 있다 (타이머, input 핸들링 등). 그 프로세스들은 그들의 큐에 task 를 삽입하는 것으로 커뮤니케이션한다.

<sub>The event loop is surrounded by other processes running in parallel to it (timers, input handling, etc.). These processes communicate with it by adding tasks to its queue.</sub>

### 24.2.1 타이머 (Timers)

브라우저는 타이머를 가진다. setTimeout() 은 타이터를 생성하며, 그게 수행될 때까지 기다린 뒤 큐에 task 를 추가한다. 그것은 이 시그니쳐를 가진다.

<sub>Browsers have timers. setTimeout() creates a timer, waits until it fires and then adds a task to the queue. It has the signature:</sub>

```javascript
setTimeout(callback, ms)
```

ms 밀리초 후 콜백함수는 task 큐에 추가된다. 중요한 점은 ms 는 단지 콜백이 추가되는 것을 나타낼 뿐이고, 실제 실행되는 때가 아니다. 즉, 이벤트 루프가 블럭되거나 해서 훨씬 나중에 일어날 수 있다.

<sub>After ms milliseconds, callback is added to the task queue. It is important to note that ms only specifies when the callback is added, not when it actually executed. That may happen much later, especially if the event loop is blocked (as demonstrated later in this chapter).</sub>

0으로 세팅한 setTimeout()은 보통 바로 다음의 task 큐에 무언가를 추가하는 방법으로 사용된다. 그러나 일부 브라우저는 최소값 이하의 ms 를 허용하지 않는다 (Firefox 는 4 ms); 만일 그렇다면 최소값으로 세팅된다.

<sub>setTimeout() with ms set to zero is a commonly used work-around to add something to the task queue right away. However, some browsers do not allow ms to be below a minimum (4 ms in Firefox); they set it to that minimum if it is.</sub>

### 24.2.2 표시중인 DOM의 변화 (Displaying DOM changes)

대부분의 DOM 변화 (특히 레이이웃 재설정)에서, 화면은 바로 업데이트 되지 않는다. "레이아웃은 매 16ms 의 틱마다 새 갱신 작업이 발생한다. " [@bz_moz](https://twitter.com/bz_moz) 그리고 반드시 이벤트 루프를 통해 그 실행 찬스가 주어진다

<sub>For most DOM changes (especially those involving a re-layout), the display isn’t updated right away. “Layout happens off a refresh tick every 16ms” ([@bz_moz](https://twitter.com/bz_moz)) and must be given a chance to run via the event loop.</sub>

브라우저에는 레이아웃 리듬(?)과의 충돌을 회피하는 빈번한 DOM 업데이트를 조정하는 방법이 있다. 자세한 것에 대해서는 requestAnimationFrame() 문서를 찾아봐라.

<sub>There are ways to coordinate frequent DOM updates with the browser, to avoid clashing with its layout rhythm. Consult the documentation on requestAnimationFrame() for details.</sub>

### 24.2.3 실행-완료 의미 (Run-to-completion semantics)

자바스크립트는 소위 실행 완료 의미를 가진다. 현재 task 는 항상  다음 task 가 실행되기 전에 종료된다. 그것은 각 task 는 현재의 모든 상태를 완벽하게 제어하고 동시 변경(concurrent modification)에 대해 걱정할 필요가 없다는걸 의미한다.

<sub>JavaScript has so-called run-to-completion semantics: The current task is always finished before the next task is executed. That means that each task has complete control over all current state and doesn’t have to worry about concurrent modification.</sub>

예제를 보자

<sub>Let’s look at an example:</sub>

```javascript
setTimeout(function () { // (A)
    console.log('Second');
}, 0);

console.log('First'); // (B)
```

A 라인의 함수 시작은 즉시 task 큐에 추가된다. 그러나 현재의 코드 조각의 완료(B 라인) 후에만 실행된다. 그것은 코드의 출력은 항상 다음이라는걸 의미한다:

<sub>The function starting in line A is added to the task queue immediately, but only executed after the current piece of code is done (in particular line B!). That means that this code’s output will always be:</sub>

```javascript
First
Second
```

### 24.2.4 이벤트 루프의 블럭 (Blocking the event loop)

봤던 대로, 각 탭(일부 완전한 브라우저)은는 싱글 프로세스로 관리된다. - 사용자 인터페이스와 다른 계산들 둘다. 그것은 당신이 사용자 인터페이스를 프로세스 안의 장시간의 계산을 수행함으로 프리징할 수 있다는걸 의미한다. 다음 코드는 그것을 보여준다.

<sub>As we have seen, each tab (in some browers, the complete browser) is managed by a single process – both the user interface and all other computations. That means that you can freeze the user interface by performing a long-running computation in that process. The following code demonstrates that.</sub>

```html
<a id="block" href="">Block for 5 seconds</a>
<p>
<button>This is a button</button>
<div id="statusMessage"></div>
<script>
    document.getElementById('block')
    .addEventListener('click', onClick);

    function onClick(event) {
        event.preventDefault();

        setStatusMessage('Blocking...');
        
        // setTimeout 호출, 브라우저가 표시(setStatusMessage)할 시간을 가진다. (Call setTimeout(), so that browser has time to display)
        // 상태 메시지 (status message)
        setTimeout(function () {
            sleep(5000);
            setStatusMessage('Done');
        }, 0);
    }
    function setStatusMessage(msg) {
        document.getElementById('statusMessage').textContent = msg;
    }
    function sleep(milliseconds) {
        var start = Date.now();
        while ((Date.now() - start) < milliseconds);
    }
</script>
```

<img src="./leanpub_github-alt.png" height="40"> 여기서 온라인으로 시도해볼 수 있다. (http://rauschma.github.io/async-examples/blocking.html)

<sub>You can try out the code online. (http://rauschma.github.io/async-examples/blocking.html)</sub>

처음 링크를 클릭할 때마다 onClick() 이 발생된다. 그것은 5초동안 이벤트 루프를 블럭하는 동기 - sleep() 함수를 사용한다. 그 초 동안 사용자 인터페이스는 동작하지 않는다. 예를 들면 "Simple 버튼" 을 클릭할 수 없다.

Whenever the link at the beginning is clicked, the function onClick() is triggered. It uses the – synchronous – sleep() function to block the event loop for five seconds. During those seconds, the user interface doesn’t work. For example, you can’t click the “Simple button”.

### 24.2.5 블럭 회피 (Avoiding blocking)

이벤트 루프에서 블럭을 회피하는 두가지 방법:

<sub>You avoid blocking the event loop in two ways:</sub>

첫째, 메인 프로세스에서 긴 실행시간이 걸리는 계산을 수행하지 않고, 다른 프로세스로 이동시킨다. 이것은 Worker API를 통해 달성될 수 있다.

<sub>First, you don’t perform long-running computations in the main process, you move them to a different process. This can be achieved via the Worker API.</sub>

둘째, 긴 실행시간이 걸리는 계산(당신의 작업 프로세스의 알고리즘, 네트워크 요청 등) 의 결과를 (동기적으로) 기다리지 말고, 이벤트 루프와 함께 계산을 실행시키고, 끝났을 때 알림을 받는다. 사실, 일반적으로 브라우저에서 이런 방법으로 일을 하는걸 선택할 필요가 없다. 예를 들면, 동기적인 내장된 기본 sleep 이 없다(이전에 구현된 sleep() 처럼). 대신 setTimeout 으로 sleep을 비동기로 시킬 수 있다.

<sub>Second, you don’t (synchronously) wait for the results of a long-running computation (your own algorithm in a Worker process, a network request, etc.), you carry on with the event loop and let the computation notify you when it is finished. In fact, you usually don’t even have a choice in browsers and have to do things this way. For example, there is no built-in way to sleep synchronously (like the previously implemented sleep()). Instead, setTimeout() lets you sleep asynchronously.</sub>

다음 섹션에서는 결과를 비동기적으로 기다리는 테크닉에 대해 알아본다

<sub>The next section explains techniques for waiting asynchronously for results.</sub>

## 24.3 비동기적으로 결과 받기 (Receiving results asynchronously)

비동기적으로 결과를 받는 두가지 일반적 패턴: 이벤트와 콜백이다.

<sub>Two common patterns for receiving results asynchronously are: events and callbacks.</sub>

### 24.3.1 이벤트를 통한 비동기 결과 (Asynchronous results via events)

이 비동기적을 결과를 받는 이 패턴에서는 요청 객체를 생성하고 각각의 성공적인 계산 및 에러 핸들링의 이벤트 핸들러를 등록한다. 다음 코드는 XMLHttpRequest API 와 함께 작동하는 방법을 보여준다.

<sub>In this pattern for asynchronously receiving results, you create an object for each request and register event handlers with it: one for a successful computation, another one for handling errors. The following code shows how that works with the XMLHttpRequest API:</sub>

```javascript
var req = new XMLHttpRequest();
req.open('GET', url);

req.onload = function () {
    if (req.status == 200) {
        processData(req.response);
    } else {
        console.log('ERROR', req.statusText);
    }
};

req.onerror = function () {
    console.log('Network Error');
};

req.send(); // 요청을 stack 큐에 추가한다. (Add request to task queue)
```

마지막 라인은 요청을 사실은 수행하지 않고, task 큐에 추가한다. 따라서 당신은 onload와 onerror 를 세팅 전 open 메서드를 나중에 호출할 수 있다. 동일하게 동작하며 자바스크립트의 실행 완료 표현에 알맞다.

<sub>Note that the last line doesn’t actually perform the request, it adds it to the task queue. Therefore, you could also call that method right after open(), before setting up onload and onerror. Things would work the same, due to JavaScript’s run-to-completion semantics.</sub>

#### 24.3.1.1 암묵적 요청 (Implicit requests)

브라우저 API IndexedDB 는 약간 괴상한 이벤트 핸들링을 가진다.

<sub>The browser API IndexedDB has a slightly peculiar style of event handling:</sub>

```javascript
var openRequest = indexedDB.open('test', 1);

openRequest.onsuccess = function (event) {
    console.log('Success!');
    var db = event.target.result;
};

openRequest.onerror = function (error) {
    console.log(error);
};
```

먼저 요청 객체를 만들고 결과 알림을 받기 위한 이벤트 리스너를 추가한다. 하지만 명시적으로 요청을 대기할 필요가 없고, open()으로 종료된다. 그것은 현재 task가 완료 된 뒤 실행된다. 당신은 open을 호출한 뒤 이벤트 핸들러를 등록할 수(실제로 해야 한다) 있다.

<sub>You first create a request object, to which you add event listeners that are notified of results. However, you don’t need to explicitly queue the request, that is done by open(). It is executed after the current task is finished. That is why you can (and in fact must) register event handlers after calling open().</sub>

만일 멀티 스레드 프로그래밍 언어를 사용하고 있다면, 이런 결과 핸들링은 race condition 경향으로 보이는 이상한 스타일로 보일 것이다. 하지만 실행과 완료에서 언제나 안전한 상황을 보장한다.

<sub>If you are used to multi-threaded programming languages, this style of handling requests probably looks strange, as if it may be prone to race conditions. But, due to run to completion, things are always safe.</sub>

#### 24.3.1.2 이벤트는 단일 결과로 작동하지 않는다 (Events don’t work well for single results)

만일 결과를 여러번 받을 때 이런 결과 비동기 결과 계산 스타일은 OK 다. 하지만 만일 단지 하나의 결과를 받을 경우 중복이 문제가 된다. 이런 경우에는 콜백이 인기가 있다.

<sub>This style of handling asynchronously computed results is OK if you receive results multiple times. If, however, there is only a single result then the verbosity becomes a problem. For that use case, callbacks have become popular.</sub>

### 24.3.2 콜백을 통한 비동기 결과 (Asynchronous results via callbacks)

만일 콜백을 통한 비동기 핸들링을 할 경우, 당신은 비동기 함수 혹은 메서드 호출에 콜백 함수 파라미터를 전달한다.

<sub>If you handle asynchronous results via callbacks, you pass callback functions as trailing parameters to asynchronous function or method calls.</sub>

다음은 Node.js 에 예저 코드이다. 비동기적인 fs.readFile 호출로 텍스트 파일의 컨텐트를 읽는다.

<sub>The following is an example in Node.js. We read the contents of a text file via an asynchronous call to fs.readFile():</sub>

```javascript
// Node.js
fs.readFile('myfile.txt', { encoding: 'utf8' },
    function (error, text) { // (A)
        if (error) {
            // ...
        }
        console.log(text);
    });
```

readFile()이 성공하면, 라인 A 콜백은 파라미터 텍스트를 통해 결과를 받는다. 그렇지 않으면 콜백은 첫번째 인자를 통해 에러(보통, Error 혹은 에러를 상속한 생성자의 인스턴스) 를 얻는다.

<sub>If readFile() is successful, the callback in line A receives a result via the parameter text. If it isn’t, the callback gets an error (often an instance of Error or a sub-constructor) via its first parameter.</sub>

고전적인 함수형 프로그래밍 스타일에서 동일한 코드는 다음과 같다.

<sub>The same code in classic functional programming style would look like this:</sub>

```javascript
// Functional
readFileFunctional('myfile.txt', { encoding: 'utf8' },
    function (text) { // success
        console.log(text);
    },
    function (error) { // failure
        // ...
    });
```

### 24.3.3 지속 - 통과 스타일 (Continuation-passing style)


The programming style of using callbacks (especially in the functional manner shown previously) is also called continuation-passing style (CPS), because the next step (the continuation) is explicitly passed as a parameter. This gives an invoked function more control over what happens next and when.

The following code illustrates CPS:

```javascript
console.log('A');
identity('B', function step2(result2) {
    console.log(result2);
    identity('C', function step3(result3) {
       console.log(result3);
    });
    console.log('D');
});
console.log('E');

// Output: A E B D C

function identity(input, callback) {
    setTimeout(function () {
        callback(input);
    }, 0);
}
```

For each step, the control flow of the program continues inside the callback. This leads to nested functions, which are sometimes referred to as callback hell. However, you can often avoid nesting, because JavaScript’s function declarations are hoisted (their definitions are evaluated at the beginning of their scope). That means that you can call ahead and invoke functions defined later in the program. The following code uses hoisting to flatten the previous example.

```javascript
console.log('A');
identity('B', step2);
function step2(result2) {
    // The program continues here
    console.log(result2);
    identity('C', step3);
    console.log('D');
}
function step3(result3) {
   console.log(result3);
}
console.log('E');
More information on CPS is given in [3].
```

### 24.3.4 Composing code in CPS
In normal JavaScript style, you compose pieces of code via:

Putting them one after another. This is blindingly obvious, but it’s good to remind ourselves that concatenating code in normal style is sequential composition.
Array methods such as map(), filter() and forEach()
Loops such as for and while
The library Async.js provides combinators to let you do similar things in CPS, with Node.js-style callbacks. It is used in the following example to load the contents of three files, whose names are stored in an Array.

```javascript
var async = require('async');

var fileNames = [ 'foo.txt', 'bar.txt', 'baz.txt' ];
async.map(fileNames,
    function (fileName, callback) {
        fs.readFile(fileName, { encoding: 'utf8' }, callback);
    },
    // Process the result
    function (error, textArray) {
        if (error) {
            console.log(error);
            return;
        }
        console.log('TEXTS:\n' + textArray.join('\n----\n'));
    });
```

### 24.3.5 Pros and cons of callbacks
Using callbacks results in a radically different programming style, CPS. The main advantage of CPS is that its basic mechanisms are easy to understand. But there are also disadvantages:

Error handling becomes more complicated: There are now two ways in which errors are reported – via callbacks and via exceptions. You have to be careful to combine both properly.
Less elegant signatures: In synchronous functions, there is a clear separation of concerns between input (parameters) and output (function result). In asynchronous functions that use callbacks, these concerns are mixed: the function result doesn’t matter and some parameters are used for input, others for output.
Composition is more complicated: Because the concern “output” shows up in the parameters, it is more complicated to compose code via combinators.
Callbacks in Node.js style have three disadvantages (compared to those in a functional style):

The if statement for error handling adds verbosity.
Reusing error handlers is harder.
Providing a default error handler is also harder. A default error handler is useful if you make a function call and don’t want to write your own handler. It could also be used by a function if a caller doesn’t specify a handler.

## 24.4 먼저 찾아보기 (Looking ahead)
The next chapter covers Promises and the ES6 Promise API. Promises are more complicated under the hood than callbacks. In exchange, they bring several significant advantages and eliminate most of the aforementioned cons of callbacks.

## 24.5 추가로 읽을 것 (Further reading)
[1] “Help, I’m stuck in an event-loop” by Philip Roberts (video).

[2] “Event loops” in the HTML Specification.

[3] “Asynchronous programming and continuation-passing style in JavaScript” by Axel Rauschmayer.
