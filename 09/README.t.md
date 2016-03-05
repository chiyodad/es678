#  9. Variables and scoping
변수와 스코핑

이번 장은 변수와 스코핑이 ECMAScript 6 에서 어떻게 핸들링 되는지 고찰해본다

##  9.1 개요
ES6에서는 두가지의 새로운 변수 선언을 제공한다. let 과 const이다. var를 사용하는 ES5의 변수 선언법을 거의 대체한다.

###  9.1.1 let
let 은 var 와 비슷하게 동작하지만, 그것이 선언하는 변수는 블럭 스코프이다. 그것은 단지 현재 블럭에서만 존재한다. var 는 함수 스코프이다.
다음 코드에 따르면 당신은 let 선언 변수인 tmp 가 단지 A 라인에서 시작한 블럭에만 존재하는걸 볼 수 있다


```javascript
function order(x, y) {
    if (x > y) { // (A)
        let tmp = x;
        x = y;
        y = tmp;
    }
    console.log(tmp===x); // ReferenceError: tmp is not defined
    return [x, y];
}
```

### 9.1.2 const
const 는 let과 비슷하게 동작하지만 당신은 나중에 값을 변경할 수 없으며, 즉시 초기화로만 선언해야하는 변수이다.