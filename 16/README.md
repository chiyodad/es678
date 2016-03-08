#  16. Modules

이 챕터에서는 ECMAScript 6 의 내장 모듈 작동 방법에 대해 설명하고있다.

##  16.1 Overview

ECMAScript 6 에서의, modules 은 파일에 저장되어있다. 하나의 module 당 파일과 하나의 파일당 module 이 명확히 존재한다.
당신은 module 을 exporting 하는 두 가지 방법을 가지고 있다. 이 두 방법을 혼합할 수 있지만, 개별적으로 사용하는것이 일반적으로 더 좋다.

###  16.1.1 Multiple named exports

다양한 named exports 를 할 수 있다.

```javascript
//------ lib.js ------
export const sqrt = Math.sqrt;
export function square(x) {
    return x * x;
}
export function diag(x, y) {
    return sqrt(square(x) + square(y));
}

//------ main.js ------
import { square, diag } from 'lib';
console.log(square(11)); // 121
console.log(diag(4, 3)); // 5
```

당신은 또한 전체 모듈을 가져올 수 있다.

```javascript
//------ main.js ------
import * as lib from 'lib';
console.log(lib.square(11)); // 121
console.log(lib.diag(4, 3)); // 5
```

###  16.1.2 Single default export

단일 default export 를 할 수 있다. 함수 예제:

```javascript
//------ myFunc.js ------
export default function () { ··· } // no semicolon!

//------ main1.js ------
import myFunc from 'myFunc';
myFunc();
```

또는 클래스

```javascript
//------ MyClass.js ------
export default class { ··· } // no semicolon!

//------ main2.js ------
import MyClass from 'MyClass';
const inst = new MyClass();
```

함수 또는 클래스의 default-export 일 경우에는 문장의 끝에 semicolon 이 존재하지 않는것을 주의하라.


###  16.1.3 Browsers: scripts versus modules

|   | Scripts | Modules |
| -------- | ----- | ------- |
| HTML element | script | script type="module" |
| Top-level variables are | global | local to module |
| Value of this at top level | window | undefined |
| Executed | synchronously | asynchronously |
| Import declaratively (import statement) | no | yes |
| Import programmatically (Promise-based API) | yes | yes-global |
| File extension | .js | .js |

###  16.2 Modules in JavaScript

비록 JavaScript 에는 내장된 모듈이 없었지만, 커뮤니티는 ES5(and earlier) 라이브러리에 의해 지원된 모듈의 간소한 스타일을
수렴하였다. 이 스타일은 ES6 에서도 채용되었다.


###  16.2.1 ECMAScript 5 module systems


###  16.2.2 ECMAScript 6 modules


###  16.3 The basics of ES6 modules


