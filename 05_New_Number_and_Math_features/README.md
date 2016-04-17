# 5. 새로운 수 와 Math 기능

이 챕터는 ECMAScript6의 새로운 수와 Math 기능을 설명한다.

## 5.1 개요
당신은 이제 수를 2진수과 8진수으로 표기 할 수 있다:

```javascript
> 0xFF // ES5: hexadecimal
255
> 0b11 // ES6: binary
3
> 0o10 // ES6: octal
8
```

### 5.1.2 새로운 Number 프로퍼티

이 전역 객체 Number는 몇 가지 새로운 프로퍼티를 얻었다. 다른 것들 사이에서:

* 반올림 오류에 대한 허용 오차와 부동 소수점 숫자와의 비교를 위한 Number.EPSILON.
* 자바스크립트 정수형이 안전한지 아닌지를 결정하는 메소드와 상수(정밀도 손실이 없는 부호를 포함한 53bit 범위):
  * Number.isSafeInteger(number)
  * Number.MIN_SAFE_INTEGER
  * Number.MAX_SAFE_INTEGER
 
### 5.1.3 새로운 Math 메소드
전역 객체 Math는 수치, 삼각 함수와 비트 연산에 대한 새로운 매소두를 가진다. 4가지 예제를 보자.

Max.sign()는 수의 부호를 반환한다.:

```javascript
> Math.sign(-8)
-1
> Math.sign(0)
0
> Math.sign(3)
1
```

Math.trunc() 는 수의 소수를 제거한다.:

```javascript
> Math.trunc(3.1)
3
> Math.trunc(3.9)
3
> Math.trunc(-3.1)
-3
> Math.trunc(-3.9)
-3
```

Math.log10() 기수가 10인 로그를 계산한다.

```javascript
> Math.log10(100)
2
```

Math.hypot() 인자값의 제곱의 합의 루트 값을 계산한다.

```javascript
> Math.hypot(3, 4)
5   
```


## 5.2 새로운 정수 리터럴

ECMAScript 5는 이미 16진수 정수에 대한 리터럴을 가진다.:

```javascript
> 0x9
9
> 0xA
10
> 0x10
16
> 0xFF
255
```

ECMAScript 6 두개의 새로운 종류의 리터럴을 갖는다.:

2진수 리터럴은 0b또는 0B의 접두사를 갖는다:

```javascript
  > 0b11
  3
  > 0b100
  4
```

8진수 리터럴은 0o또는 0O의 접두사를 갖는다. (그래, 0뒤에 대문자 O이다. ; 처음 변형을 사용하는 경우에도 괜찬다.)

```javascript
  > 0o7
  7
  > 0o10
  8
```

Number.prototype.toString(기수)메소드가 수를 다시 변환하는데 사용될 수 있다는것을 기억하라.:

```javascript
> (255).toString(16)
'ff'
> (4).toString(2)
'100'
> (8).toString(8)
'10'
```

### 5.2.1 8진법 리터럴 사용 예: 유닉스 스타일 파일 퍼미션
Node.js 파일 시스템 모듈에서 몇몇 함수는 모드에 대한 파라미터를 갖는다. 유닉스로 부터 남은 인코딩을 통해 이 값은 파일 퍼미션을 지정하는데 사용된다.:
* 퍼미션은 세가지 사용자 카테고리를 통해 지정된다.:
  * User: 파일의 소유자
  * Group: 파일에 연관된 그룹의 맴버
  * All: 모든 사람
* 각 카테고리는 아래의 퍼미션을 부여할 수 있다:
  * r (read): 이 카테고리에 있는 사용자는 파일을 읽는 것을 허용한다.
  * w (write): 이 카테고리에 있는 사용자는 파일을 변경하는 것을 허용한다.
  * x (execute): 이 카테고리에 있는 사용자는 파일을 실행하는 것을 허용한다.

이것은 퍼미션이 9bits로 표현된다는것을 의미한다 (3 카테고리 마다 3개의 퍼미션).:

| |User|Group|All|
|---|---|---|---|
|Permissions|r, w, x|r, w, x|r, w, x|
|Bit|8, 7, 6|5, 4, 3|2, 1, 0|

사용자의 단일 카테고리의 퍼미션의 3비트로 저장된다.:

|Bits|Permissions|Octal digit|
|---|---|---|
|000|–––|0|
|001|––x|1|
|010|–w–|2|
|011|–wx|3|
|100|r––|4|
|101|r–x|5|
|110|rw–|6|
|111|rwx|7|

8진수 수가 모든 퍼미션을 간단하게 3자리 수, 하나당 사용하의 카테고리로 표현할수 있다는것을 의미한다. 두 예를 보면:

* 755 = 111,101,101: 나는 바꾸고 읽고 실행하고 있다 모든 사람들은 오직 읽기와 실행만 할 수 있다.
* 640 = 110,100,000: 나는 읽고 쓰는게 가능하고 그룹의 맴버는 읽을 수만 있다. 모든 사람은 접근할 수 없다.

### 5.2.2 parseInt() 와 새로운 integer 리터럴

parseInt()는 아래 같은 시그니쳐를 갖는다.:

```javascript
parseInt(string, radix?)
```

이것은 16진수 리터럴 표기에 대한 특별한 지원을 제공한다. 

- string의 접두사 0x (또는 0X)를 제거 하면: 
* radix가 없거나 또는 0이면 16으로 설정한다.
* radix가 이미 16이다.

예를 들면:

```javascript
> parseInt('0xFF')
255
> parseInt('0xFF', 0)
255
> parseInt('0xFF', 16)
255
```

모든 경우에서 숫자는 숫자만 아닌 첫번째 자리까지 파싱되어 있다.:

```javascript
> parseInt('0xFF', 10)
0
> parseInt('0xFF', 17)
0
```

그러나 parseInt()는 2진수나 8진수에 대한 특별한 지원은 없다.

```javascript
> parseInt('0b111')
0
> parseInt('0b111', 2)
0
> parseInt('111', 2)
7

> parseInt('0o10')
0
> parseInt('0o10', 8)
0
> parseInt('10', 8)
8
```

만약 이런 종류의 리터럴을 파싱하기 원한다면 Number()의 사용이 필요하다:

```javascript
> Number('0b111')
7
> Number('0o10')
8
```

대안적으로 접두사를 제거하고 적당한 radix와 합께 parseInt()를 사용할 수 있다.
Alternatively, you can also remove the prefix and use parseInt() with the appropriate radix:

```javascript
> parseInt('111', 2)
7
> parseInt('10', 8)
8
```

## 5.3 새로운 Number 스태틱 프로퍼티

이 섹션은 ECMAScript6에서 생성자 Number에 나온 새로운 프로퍼티를 설명한다.

### 5.3.1 이전 전역 함수들

수와 관련된 4개의 함수들은 여전히 전역 함수에 있고, Number에 메소드로 추가 되었다. :isFinite와 isNaN, parseFloat와 parseInt. 이 함수들은 모두 전역에 짝지어 진것과 같은 일을 하지만 isFinite와 isNaN은 그 함수의 인자값을 수로 더 이상 강제 변환 하지 않는다. 특히 isNaN이 중요하다. 이어진 절에서 자세한 설명을 한다.

### 5.3.1.1 Number.isFinite(number)

number가 정확한 수인지(또는 Infinity, -Infinity, NaN이 아닌지) 여부?

```javascript
> Number.isFinite(Infinity)
false
> Number.isFinite(-Infinity)
false
> Number.isFinite(NaN)
false
> Number.isFinite(123)
true
```
이 메소드의 이점은 이것은 강제로 파라메터를 변환하지 않는다는 것이다(반면에 전역 함수를 변환한다.):

```javascript
> Number.isFinite('123')
false
> isFinite('123')
true
```

#### 5.3.1.2 Number.isNaN(number)

number가 NaN값인지 여부. ===을 통한 확인을 하는것은 핵스럽다. NaN은 자기 자신과 같지 않는 유일한 값이다.:

```javascript
> const x = NaN;
> x === NaN
false
```

그러므로, 이 표현식은 이것을 확인하는데 사용된다.

```javascript
> x !== x
true
```

Number.isNaN()을 사용하는것은 더 자기 설명 적이다.:

```javascript
> Number.isNaN(x)
true
```

Number.isNaN()은 파라메터를 수로 강제 변환하지 않는 것에 대한 장점(전역 함수 행동과 반하여)이 있다.:

```javascript
> Number.isNaN('???')
false
> isNaN('???')
true
```

#### 5.3.1.3 Number.parseFloat와 Number.parseInt

이 두 메소드는 이름이 같은 전역함수와 정확히 동작이 같다. 이것들은 완전성을 위하여 Number에 추가 되었다; 이제 모든 수와 관련된 함수는 Number로 사용가능 하다.

```javascript
Number.parseFloat(string)
Number.parseInt(string, radix)
```

### 5.3.2 Number.EPSILON

특히 분수에서 반올림 오류는 Javascript3에서 문제가 된다. 예를 들어 0.1와 0.2을 더하고 그것과 0.3(이것 또한 정확하게 표현되지 않는다.)을 비교 한다면 정확하게 표현되지 않을 수 있다는 것을 알아야 한다.

```javascript
> 0.1 + 0.2 === 0.3
false
```

부동 소수점 수를 비교할 때 Number.EPSILON은 이유있는 오류 마진을 지정한다. 이것은 부동 소수점 수를 비교할때 더 나은 방법을 제공한다. 아래 함수로 보여주겠다.

```javascript
function epsEqu(x, y) {
    return Math.abs(x - y) < Number.EPSILON;
}
console.log(epsEqu(0.1+0.2, 0.3)); // true
```

### 5.3.3 Number.isInteger(number)

Javascript는 오로지 부동소수점 수(doubles)만 가진다. 따라서 정수는 간단하게 소수 부분이 없는 부동 소수점 수 이다.

만약 number에 소수가 없다면 Number.isInteger(number)는 true를 반환한다.

```javascript
> Number.isInteger(-17)
true
> Number.isInteger(33)
true
> Number.isInteger(33.1)
false
> Number.isInteger('33')
false
> Number.isInteger(NaN)
false
> Number.isInteger(Infinity)
false
```

### 5.3.4 안전 정수

Javascript 수는 오직 53bit의 부호있는 정수를 표현하는 저장 공간이 있다. -2^53 < i < 2^53은 정수 i는 안전 하다. 정확히 무엇을 말하려는지 지금 설명 하겠다. 아래 프로퍼티는 자바스크립트 정수가 안전한지를 결정하는것을 돕는다.:
* Number.isSafeInteger(number)
* Number.MIN_SAFE_INTEGER
* Number.MAX_SAFE_INTEGER

안전 정수에 대한 개념은 자바스크립트에서 어떻게 정확한 정수를 표현하는가에 대해서 중점적으로 둔다. (-2^53, 2^53)범위에서 (경계면을 제외하고), 자바스크립트 정수는 안전하다: 이것들 사이에서 1대1 맴핑되고, 이것은 수학상의 정수로 표현된다.

범위를 넘어서, 자바스크립트 정수는 안전하지 않다: 둘 이상 수학상 정수를 자바스크립트에서 같은 정수로 표현된다. 예를 들면 2^53 이상에서 자바스크립트는 두번째 수학적 정수만 표현될 수 있다.:

```javascript
> Math.pow(2, 53)
9007199254740992

> 9007199254740992
9007199254740992
> 9007199254740993
9007199254740992
> 9007199254740994
9007199254740994
> 9007199254740995
9007199254740996
> 9007199254740996
9007199254740996
> 9007199254740997
9007199254740996
```

그러므로 안전 자바스크립트 정수는 명확하게 하나의 수학적 정수만 표현되는 것 이다.

#### 5.3.4.1 안전 정수에 관한 스태틱 Number 프로퍼티

안전 정수에 상계와 하계를 지정하는 두개의 Number 프로퍼티는 아래와 같이 정의되어 있다.:

```javascript
Number.MAX_SAFE_INTEGER = Math.pow(2, 53)-1;
Number.MIN_SAFE_INTEGER = -Number.MAX_SAFE_INTEGER;
```

Number.isSafeInteger()는 자바스크립트 수가 안전한지 여부를 결정하고 아래와 같이 구현할 수 있다.:

```javascript
Number.isSafeInteger = function (n) {
    return (typeof n === 'number' &&
        Math.round(n) === n &&
        Number.MIN_SAFE_INTEGER <= n &&
        n <= Number.MAX_SAFE_INTEGER);
}
```

값 n이 주어질때, 이 함수는 처음으로 n이 수와 정수인지 확인한다. 만약 둘다 성공한다면 만약 MIN_SAFE_INTEGER보다 크거나 같고 MAX_SAFE_INTEGER보다 작거나 같다면, n은 안전하다.

#### 5.3.4.2 언제 정수의 계산결과가 정확한가?

어떻게 우리는 정수의 계산 결과가 정확하다고 확신 할 수 있는가? 예를 들면, 아래 결과는 명백하게 틀리다.:

```javascript
> 9007199254740990 + 3
9007199254740992
```

두가지 피연산자가 안전하지만 결과는 안전하지 않다.:

```javascript
> Number.isSafeInteger(9007199254740990)
true
> Number.isSafeInteger(3)
true
> Number.isSafeInteger(9007199254740992)
false
```

다음 결과 또한 올바르지 않다.:

```javascript
> 9007199254740995 - 10
9007199254740986
```

이 때는 결과는 안전하지만 하나의 피연산자는 그렇지 않다.:

```javascript
> Number.isSafeInteger(9007199254740995)
false
> Number.isSafeInteger(10)
true
> Number.isSafeInteger(9007199254740986)
true
```

그러므로 만약 모든 피연산자와 결과가 안전하다면, 정수 연산자 op를 적용한 결과가 올바르다고 보장된다. 더 공식적으로 :

```javascript
isSafeInteger(a) && isSafeInteger(b) && isSafeInteger(a op b)
```
a op b는 정확한 결과라는것을 의미한다.

이 섹션의 소스
“Clarify integer and safe integer resolution”, email by Mark S. Miller to the es-discuss mailing list.

# 5.4 Math

ECMAScript 6에서 이 전역 객체 Math는 몇몇의 새로운 메소드를 갖는다.

### 5.4.1 다양한 수학적 함수

#### 5.4.1.1 Math.sign(x)
-1또는 +1로 x의 부호를 반환한다. 만약 x가 NaN또는 0이면 x를 반환한다.

```javascript
> Math.sign(-8)
-1
> Math.sign(3)
1

> Math.sign(0)
0
> Math.sign(NaN)
NaN

> Math.sign(-Infinity)
-1
> Math.sign(Infinity)
1
```

#### 5.4.1.2 Math.trunc(x)

x의 소수부를 제거한다. 다른 반올림 메소드 Math.floor(), Math.ceil(), Math.round()를 보완한다.

```javascript
> Math.trunc(3.1)
3
> Math.trunc(3.9)
3
> Math.trunc(-3.1)
-3
> Math.trunc(-3.9)
-3
```

Math.trunc()를 이것처럼 구현할 수 있다.:

```javascript
function trunc(x) {
    return Math.sign(x) * Math.floor(Math.abs(x));
}
```

#### 5.4.1.3 Math.cbrt(x)

x의 세제곱근을 반환한다. (∛x).

```javascript
> Math.cbrt(8)
2
```

### 5.4.2 지수와 로그에서 0대신 1을 사용

0뒤로 부터 오는 작은 소수는 더 정확하게 표현될 수 있다. 나는 이것을 작은 소수와 함께 증명하겠다. (자바스크립트의 수는 내부적으로 밑수가 2로 저장되지만 동일한 이유가 적용된다.).

밑수가 10인 부동소수점수는 내부적으로 가수 * 10^지수로 표현된다. 이 가수는 소수점 이전 하나의 숫자를 갖고 지수는 필요에 따라 점을 "이동" 한다. 만약 작은 정수를 내부적 표현으로 변환한다면 소수점 이전의 0은 소수점 이전의 1보다 더 작은 가수로 이끈다. 예를 들면:

* (A) 0.000000234 = 2.34 * 10^−7. 유효숫자: 234
* (b) 1.000000234 = 1.000000234 * 10^0. 유효숫자: 1000000234

정밀하게 보자면, 여기에서 중요한 양은 유효 숫자가 측정된 가수의 용량이다. 이것이 A가 B보다 높은 정밀도를 제공하는 이유이다.

다음과 같은 상호작용을 볼수 있다: 처음 수는 (1 * 10^-16) 0과 다르게 저장되지만 같은 수에 1을 더하면 1로 저장된다.

```javascript
> 1e-16 === 0
false
> 1 + 1e-16 === 1
true
```

#### 5.4.2.1 Math.expm1(x)

Math.exp(x)-1을 반환한다. Math.log1p()의 역이다.

그러므로, 이 메소드는 Math.exp()의 결과가 1과 근접하게 될때 높은 정밀도를 제공한다. 아래 상호작용에 대해서 둘에 대한 차이를 볼 수 있다.:

```javascript
> Math.expm1(1e-10)
1.00000000005e-10
> Math.exp(1e-10)-1
1.000000082740371e-10
```

전자가 더 좋은 결과이다. 임의 정밀도("bigfloats")와 함께 부동 소수점 수에 대한 라이브러리(decimal.js같은)을 사용하여 증명할 수 있다.:

```javascript
> var Decimal = require('decimal.js').config({precision:50});
> new Decimal(1e-10).exp().minus(1).toString()
'1.000000000050000000001666666666708333333e-10'
```

#### 5.4.2.2 Math.log1p(x)

Math.log(1+x)를 반환한다. Math.expm1의 역이다.

그러므로 이 메소드는 높은 정밀도를 1근처의 파라미터에 대해 지정 할 수 있다.

우리는 이미 1 + 1e-16 === 1을 확인했다. 그러므로, 다음 두개의 log() 호출이 같은 결과를 내는것은 놀랄일이 아니다.:

```javascript
> Math.log(1 + 1e-16)
0
> Math.log(1 + 0)
0
```
반면에, log1p()는 다른 결과를 생산한다:

```javascript
> Math.log1p(1e-16)
1e-16
> Math.log1p(0)
0
```

### 5.4.3 밑수가 2와 10인 로그
#### 5.4.3.1 Math.log2(x)

밑수가 2인 로그를 계산한다.

```javascript
> Math.log2(8)
3
```

#### 5.4.3.2 Math.log10(x)

밑수가 10인 로그를 계산한다.

```javascript
> Math.log10(100)
2
```

### 5.4.4 자바스크립트로의 컴파일링 제공

Emscripten은 asm.js에 의해 나중에 선택된 코딩 스타일을 개척했다. 가상머신(생각하기론 바이트코드)의 동작은 자바스크립트의 정적 부분집합으로 표현된다. 이 부분집합은 자바스크립트 엔진에서 효과적으로 실행된다. c++로 부터 컴파일된 결과는 원래 속도에 대략 70% 로 실행된다.

다음 Math 메소드는 asm.js와 컴파일 전략의 지원을 위해 주로 추가 되었고, 다른 애플리케이션을 위해 유용하진 않다.

#### 5.4.4.1 Math.fround(x)

x를 32 비트 부동소수점 값(float)으로 반환한다. asm.js를 통해 엔진에게 내부적으로 float 값으로 사용한다고 말하기 위해 사용된다.

#### 5.4.4.2 Math.imul(x, y)

두 32비트 정수 x, y를 곱하고 결과의 하위 32비트를 반환한다. 이것은 자바스크립트 연산자를 사용해서 따라 할 수 없고, 결과값을 32비트로 강제 변환 할 수 없는 오직 32 비트 기본 수학 연산이다. 예를들면 idiv는 아래처럼 구현될 수 있다.:

```javascript
function idiv(x, y) {
    return (x / y) | 0;
}
```
반면에 두 32비트 정수 이상의 곱셉은 값이 커서 하위 비트 잃어버린 double을 생성한다.

### 5.4.5 비트 연산자

* Math.clz32(x)
  32비트 정수 x에서 앞에 0의 갯수
```javascript
  > Math.clz32(0b01000000000000000000000000000000)
  1
  > Math.clz32(0b00100000000000000000000000000000)
  2
  > Math.clz32(2)
  30
  > Math.clz32(1)
  31
```

왜 이것이 흥미로운가? "Fast, Deterministic, and Portable Counting Leading Zeros" by Miro Samek을 인용하면:

정수의 앞에 0의 갯수는 소리나 비디오 프로세싱에서 샘플의 노말리제이션과 같은 많은 DSP 알고리즘의 중요한 연산이고 또한 실시간 스케줄러에서 빠르게 높은 우선순위 테스크를 발견하는데 좋다.

### 5.4.6 삼각 함수

* Math.sinh(x)
  x의 쌍곡선 싸인을 계산한다.
* Math.cosh(x)
  x의 쌍곡선 코싸인을 계산한다.
* Math.tanh(x)
  x의 쌍곡선 탄젠트를 계산한다.
* Math.asinh(x)
  x의 쌍곡선 싸인의 역을 계산한다.
* Math.acosh(x)
  x의 쌍곡선 코사인의 역을 계산한다.
* Math.atanh(x)
  x의 쌍곡선 탄젠트의 역을 계산한다.
* Math.hypot(...values)
  인자들의 제곱의 덧셈의 제곱근을 계산한다.

##5.5 FAQ: numbers

### 5.5.1 어떻게 내가 53bit 범위 넘는 정수를 사용할 수 있는가?

자바스크립트의 정수는 53비트의 범위를 갖는다. 이것은 64 비트 정수가 필요할때 문제가 된다. 예를 들면: JSON API에서 트위터는 트위터 ID가 매우 큰 경우 정수를 문자열로 변환을 갖는다.

그때에 제한하는 한가지 방법은 높은 정밀도 수를 위한 라이브러리(bigints 또는 bitfloats)를 사용하는 것이다. 이 라이브러리중 하나는 decimal.js이다.

자바스크립트에서 큰 정수를 지원하는 계획은 존재하나 시간이 걸릴 수 있다.
