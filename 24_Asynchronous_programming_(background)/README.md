# 24. 비동기 프로그래밍 (백그라운드)

이번 챕터는 자바스크립트의 비동기 프로그래밍의 기초를 알아본다. 이 지식은 다음 장의 ES6 Promise 에 도움이 될 것이다

## 24.1 자바스크립트의 호출 스택

함수 f 가 함수 g 를 호출할 때, g는 완료 후 (g를 실행시킨 f 내부로) 되돌아갈 곳을 알아야 한다. 이 정보는 보통 스택으로 관리되는 호출 스택 (Call Stack) 이다. 

예제를 보자

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

위의 첫 프로그램이 시작될 때 호출 스택은 비워져 있다. D 라인의 함수 호출 f(3) 이 호출 된 뒤 스택은 하나의 엔트리를 가진다.

- 전역 스코프안의 위치

C 라인의 g(x + 1) 이 호출된 뒤 스택은 두개의 엔트리를 가진다.

- f 안에 위치
- 전역 스코프안의 위치

B 라인의 h(y + 1) 이 호출된 뒤 스택은 세개의 엔트리를 가진다.

- g 안에 위치
- f 안에 위치
- 전역 스코프안의 위치

A 라인에서 보여지는 호출 스택의 스택 트레이스는 다음과 같다.

```javascript
Error
    at h (stack_trace.js:2:17)
    at g (stack_trace.js:6:5)
    at f (stack_trace.js:9:5)
    at <global> (stack_trace.js:11:1)
```

각각의 함수 종료마다 스택은 탑 엔트리에서 지워진다. f 가 종료되면 다시 글로벌 스코프로 돌아가며 라인 E 에서 리턴을 하게 되면 모든 스택은 비워진다. 그것은 프로그램의 종료를 의미한다.

## 24.2 브라우저 이벤트 루프

심플하게도 각 브라우저는 탭마다 하나의 프로세스:이벤트 루프 로 돌아간다. 이 루프는 작업 큐에서 주어지는 브라우저에 관련된 일(소위 'task' 라고 하는) 을 실행한다.

- HTML 파싱
- script 엘리먼트 안의 자바스크립트 코드 실행
- 사용자 입력에 반응(마우스 클릭, 키 입력 등)
- 비동기 네트워크 요청 결과 처리
 
2-4 항목은 브라우저 안의 엔진에 의한 자바스크립트 코드 실행 task 이다. 그것들은 코드 수행 종료시에 끝난다. 그뒤 큐에서 다음 task 를 실행할 수 있다. 다음 다이어그램은 (필립 로버츠에게 영감을 얻은) 모든 연결된 메커니즘에 대한 개요를 제공한다.

![async-stack](./async----event_loop.jpg)

이벤트 루프는 병렬로 실행되는 다른 프로세스로 둘러싸여 있다 (타이머, input 핸들링 등). 그 프로세스들은 자신이 가진 큐에 task 를 삽입하는 것으로 커뮤니케이션한다.

### 24.2.1 타이머

브라우저는 타이머를 가지고 있다. setTimeout() 은 타이머를 생성하며, 수행될 때까지 기다린 뒤 큐에 task 를 추가한다. 그것은 이러한 시그니쳐를 가진다.

```javascript
setTimeout(callback, ms)
```

ms 밀리초 후 콜백함수는 task 큐에 추가된다. 중요한 점은 ms 는 단지 콜백이 추가되는 것을 나타낼 뿐이고, 실제 실행되는 때가 아니다. 즉, 이벤트 루프가 블럭되거나 해서 훨씬 나중에 일어날 수 있다.

0으로 세팅한 setTimeout()은 보통 바로 다음의 task 큐에 무언가를 추가하는 방법으로 사용된다. 그러나 일부 브라우저는 최소값 이하의 ms 를 허용하지 않는다 (Firefox 는 4 ms); 만일 그렇다면 최소값으로 세팅된다.

### 24.2.2 표시중인 DOM의 변화

대부분의 DOM 변화 (특히 레이이웃 재설정)에서, 화면은 바로 업데이트 되지 않는다. "레이아웃은 매 16ms 의 틱마다 새 갱신 작업이 발생한다. " - [@bz_moz](https://twitter.com/bz_moz)) 그리고 반드시 이벤트 루프를 통해 그 실행 찬스가 주어진다

브라우저에는 레이아웃 리듬(?)과의 충돌을 회피하는 빈번한 DOM 업데이트를 조정하는 방법이 있다. 자세한 것에 대해서는 requestAnimationFrame() 문서가 도움이 될 것이다.

### 24.2.3 실행-완료 의미

자바스크립트는 소위 실행 완료 의미를 가진다. 현재 task 는 항상  다음 task 가 실행되기 전에 종료된다. 그것은 각 task 는 현재의 모든 상태를 완벽하게 제어하고 동시 변경(concurrent modification)에 대해 걱정할 필요가 없다는걸 의미한다.

예제를 보자

```javascript
setTimeout(function () { // (A)
    console.log('Second');
}, 0);

console.log('First'); // (B)
```

A 라인의 함수 시작은 즉시 task 큐에 추가된다. 그러나 현재의 코드 조각의 완료(B 라인) 후에만 실행된다. 그것은 코드의 출력은 항상 다음이라는걸 의미한다:

```javascript
First
Second
```

### 24.2.4 이벤트 루프의 블럭

봤던 대로 각 탭(일부 완전한 브라우저)은는 싱글 프로세스로 관리된다. 사용자 인터페이스 및 다른 계산등을 함께 관리한다. 그것은 당신이 사용자 인터페이스를 프로세스 안의 장시간의 계산을 수행함으로 프리징할 수 있다는걸 의미한다. 다음 코드는 그것을 보여준다.

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
        
        // Call setTimeout(), so that browser has time to display)
        // status message
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

처음 링크를 클릭할 때마다 onClick() 이 발생된다. 이벤트 발생 시 5초동안 이벤트 루프를 블럭하는 동기적 sleep() 함수를 사용한다. 그 초 동안 사용자 인터페이스는 동작하지 않는다. 

예를 들면, "Simple 버튼" 을 클릭할 수 없다.

### 24.2.5 블럭 회피

이벤트 루프에서 블럭을 회피하는 두가지 방법:

첫째, 메인 프로세스에서 긴 실행시간이 걸리는 계산을 수행하지 않고, 다른 프로세스로 이동시킨다. 이것은 Worker API 를 써서 적용힐 수 있다.

둘째, 긴 실행시간이 걸리는 계산(당신의 작업 프로세스의 알고리즘, 네트워크 요청 등) 의 결과를 (동기적으로) 기다리지 말고, 이벤트 루프와 함께 계산을 실행시키고, 끝났을 때 알림을 받는다. 사실, 일반적으로는 브라우저에서 이런 방법으로 일을 처리할 필요가 없다. 예를 들어 동기적인 내장된 기본 sleep 이 기본 함수로 없다(이전에 구현된 sleep() 처럼). 대신 setTimeout 으로 sleep을 비동기로 실행시킬 수 있다.

다음 섹션에서는 결과를 비동기적으로 기다리는 테크닉에 대해 알아본다

## 24.3 비동기적으로 결과 받기

비동기적으로 결과를 받는 두가지 일반적 패턴: 이벤트와 콜백이다.

### 24.3.1 이벤트를 통한 비동기 결과

이 비동기적을 결과를 받는 이 패턴에서는 요청 객체를 생성하고 각각의 성공적인 계산 및 에러 핸들링의 이벤트 핸들러를 등록한다. 다음 코드는 XMLHttpRequest API 와 함께 작동하는 방법을 보여준다.

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

#### 24.3.1.1 암묵적 요청

브라우저 API IndexedDB 는 약간 괴상한 이벤트 핸들링을 가진다.

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

만일 멀티 스레드 프로그래밍 언어를 사용하고 있다면, 이런 결과 핸들링은 race condition 경향으로 보이는 이상한 스타일로 보일 것이다. 하지만 실행과 완료에서 언제나 안전한 상황을 보장한다.

#### 24.3.1.2 이벤트는 단일 결과로 작동하지 않는다

만일 결과를 여러번 받을 때 이런 결과 비동기 결과 계산 스타일은 OK 다. 하지만 만일 단지 하나의 결과를 받을 경우 중복이 문제가 된다. 이런 경우에는 콜백이 인기가 있다.

### 24.3.2 콜백을 통한 비동기 결과

만일 콜백을 통한 비동기 핸들링을 할 경우, 당신은 비동기 함수 혹은 메서드 호출에 콜백 함수 파라미터를 전달한다.

다음은 Node.js 에 예저 코드이다. 비동기적인 fs.readFile 호출로 텍스트 파일의 컨텐트를 읽는다.

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

고전적인 함수형 프로그래밍 스타일에서 동일한 코드는 다음과 같다.

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

### 24.3.3 지속 - 통과 스타일

콜백을 사용하는 스타일(특히 전에 보여준 기능적인 벙식)은 continuation-passing style (CPS) 이라고 하는데 다음 단계를 명시적으로 파라미터로 보내기 때문이다. 이것은 next 와 when 의 발생을 통해 함수 호출에 더 많은 제어를 제공한다.

다음 코드는 CPS 를 보여준다.

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

각 단계동안 프로그램 제어의 흐름은 콜백 안에서 계속된다. 이건 중첩 함수에 리드되는데 때때로 이는 때때로 콜백 헬로 불려진다. 하지만 자바스크립트의 함수 선언은 호이스트(함수 정의는 반드시 스코프의 처음에 평가된다) 이기에 중첩을 회피할 수 있다. 이 뜻은 프로그램에서 호출 전 함수 정의를 해둔다는걸 의미한다. 다음은 이전 예제를 호이스팅을 사용하여 펼쳐본 코드이다.

```javascript
console.log('A');
identity('B', step2);
function step2(result2) {
    // 프로그램은 여기서 계속된다. (The program continues here)
    console.log(result2);
    identity('C', step3);
    console.log('D');
}
function step3(result3) {
   console.log(result3);
}
console.log('E');
```

CPS에 대한 더 많은 정보는 [3](http://www.2ality.com/2012/06/continuation-passing-style.html) 에 있다

### 24.3.4 CPS 구성 코드

보통의 자바스크립트 스타일에서 코드 조각은 다음에 의해 구성된다.

- 차례로 놓는다. 너무나도 분명하지만, 순차적 구성인 보통 스타일 안의 연결을 떠올리게 하는것이 좋다.
- 배열 메서드는 map, filter, forEach 등
- 루프는 for 와 while 등

라이브러리 Async.js 는 CPS Node.js 스타일 콜백과 비슷한 CPS 방식의 콤비네이터를 제공한다. 배열에 저장된 이름들을 가진 세개의 파일에서 컨텐츠를 로딩하는 예제에서 사용된다.

```javascript
var async = require('async');

var fileNames = [ 'foo.txt', 'bar.txt', 'baz.txt' ];
async.map(fileNames,
    function (fileName, callback) {
        fs.readFile(fileName, { encoding: 'utf8' }, callback);
    },
    // 결과 처리 (Process the result)
    function (error, textArray) {
        if (error) {
            console.log(error);
            return;
        }
        console.log('TEXTS:\n' + textArray.join('\n----\n'));
    });
```

### 24.3.5 콜백의 장점과 단점

콜백의 결과를 사용하는 CPS는 근본적으로 다른 프로그래밍 스타일이다. CPS의 주요 장점은 기본 메커니즘이 이해하기 쉽다는 것이다. 하지만 단점도 있다.

- 에러 핸들링이 더욱 복잡해진다: 지금 에러 보고서 두개 - 콜백과 예외를 통해 - 가 있다. 두개의 장점을 조합하는것에 주의해야 한다.
- 덜 우아한 식별자: 동기 함수에서는 입력(파라미터)과 출력(함수 결과) 의 명확한 구분이 있다. 콜백을 사용하는 비동기 함수에서는, 이런 구분이 혼재되어 있다. 함수의 결과는 중요하지 않고 일부 파라미터는 입력 혹은 출력을 위해 사용된다.
- 구성이 더욱 복잡해진다: 출력을 파라미터로 표현하는 관심사 때문에 연계를 통한 코드 구성이 더욱 복잡해진다.

Node.js 스타일의 콜백은 세개의 단점을 가진다.(함수형 스타일에 비해))

- 에러를 위한 if 문이 중복으로 추가된다
- 재사용되는 에러 핸들러는 어렵다.
- 제공되는 기본 에러 핸들러를 제공하는 것도 어렵다. 함수 호출을 하면서 자신만의 핸들러를 작성하지 않을 때 기본 핸들러는 유용하다. 호출자가 특정 핸들러를 하지 않을 경우 함수로서 사용될 수 있다.

## 24.4 앞서 찾아보기

다음 장에서는 Promise 와 ES6 Promise API 를 포함한다. Promise 는 콜백보다 더 복잡하다. 그 대신 여러 중요한 이점을 가져다주고, 콜백의 단점을 대부분 제거할 수 있다.


## 24.5 추가로 읽을 것

[1] [“Help, I’m stuck in an event-loop” by Philip Roberts](video)(https://vimeo.com/96425312).

[2] [“Event loops” in the HTML Specification.](https://html.spec.whatwg.org/multipage/webappapis.html#event-loops)

[3] [“Asynchronous programming and continuation-passing style in JavaScript” by Axel Rauschmayer.](http://www.2ality.com/2012/06/continuation-passing-style.html)
