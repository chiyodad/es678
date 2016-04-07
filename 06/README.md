#6. New string features

##6.1 개요
새로 추가된 string method들은 다음과 같다.
```js
'hello'.startsWith('hell');    // true
'hello'.endsWith('ello');      // true
'doo '.repeat(3);              // 'doo doo doo '
```

ES6에는 또 'template literal'이라고 하는 전혀 새로운 string literal이 도입되었다.

```js
// template literal - backtick(`)을 통한 문자열 보간(string interpolation)
const first = 'Jane';
const last = 'Doe';
console.log(`Hello ${first} ${last}!`);    // Hello Jane Doe!

// Template literal은 여러줄의 문자열을 만들 수 있도록 해준다.
const multiLine = `
This is
a string
with multiple
lines`;
```


##6.2 Unicode code point escapes
ES6에서는 (16비트를 초과한 유니코드의 경우까지도 포함한) 모든 code point를 특정할 수 있는 새로운 Unicode escape가 도입되었다.
> In ECMAScript 6, there is a new kind of Unicode escape that lets you specify any code point (even those beyond 16 bits):

```js
console.log('\u{1F680}'); 
    // ES6: single code point (16비트를 초과한 단일유니코드)
console.log('\uD83D\uDE80');
    // ES5: two code units (16비트단위의 유니코드 두 개로 이루어진 집합)
```
Unicode 챕터에서 escape에 관해 더 자세히 살펴볼 것이다.


##6.3 String interpolation, multi-line string literals and raw string literals
Template literal은 세 가지 흥미로운 기능을 제공한다. 각 챕터에서 더 깊이있게 다룰 예정이다.

#####1) template literal은 문자열 ㄴ(interpolation)을 지원한다.
```js
const first = 'Jane';
const last = 'Doe';
console.log(`Hello ${first} ${last}!`);    // Hello Jane Doe!
```

#####2) template literal은 여러줄로 이루어질 수 있다.
```js
const multiLine = `
This is
a string
with multiple
lines`;
```

#####3) template literal은 String.raw 태그함수를 이용하면 \n과 같은 escape 문자를 해석하지 않은 채 날것 그대로의 문자열로 만들 수 있다.
> template literals are “raw” if you prefix them with the tag String.raw – the backslash is not a special character and escapes such as \n are not interpreted:

```js
const str = String.raw `Not a newline: \n`;
console.log(str === 'Not a newline: \\n');    // true
```


##6.4 Iterating over strings
문자열은 이터러블하다. 즉 for-of 구문을 통해 문자열 내의 문자들을 각각 순회하며 처리할 수 있다.
```js
for (const ch of 'abc') {
    console.log(ch);
}
// a
// b
// c
```

또한 spread operator(`...`)을 이용하여 문자열을 배열로 치환할 수도 있다.
```js
const chars = [...'abc'];    // ['a', 'b', 'c']
```


###6.4.1 Iteration honors Unicode code points

The string iterator splits strings along code point boundaries, which means that the strings it returns comprise one or two JavaScript characters:

for (const ch of 'x\uD83D\uDE80y') {
    console.log(ch.length);
}
// Output:
// 1
// 2
// 1
6.4.2 Counting code points
Iteration gives you a quick way to count the Unicode code points in a string:

> [...'x\uD83D\uDE80y'].length
3
6.4.3 Reversing strings with non-BMP code points
Iteration also helps with reversing strings that contain non-BMP code points (which are larger than 16 bit and encoded as two JavaScript characters):

const str = 'x\uD83D\uDE80y';

// ES5: \uD83D\uDE80 are (incorrectly) reversed
console.log(str.split('').reverse().join(''));
    // 'y\uDE80\uD83Dx'

// ES6: order of \uD83D\uDE80 is preserved
console.log([...str].reverse().join(''));
    // 'y\uD83D\uDE80x'
The two reversed strings in the Firefox console.
The two reversed strings in the Firefox console.
Remaining problem: combining marks
A combining mark is a sequence of two Unicode code points that is displayed as single symbol. The ES6 approach to reversing a string that I have presented here works for non-BMP code points, but not for combining marks. For those, you need a library, e.g. Mathias Bynens’ Esrever.

6.5 Numeric values of code points
The new method codePointAt() returns the numeric value of a code point at a given index in a string:

const str = 'x\uD83D\uDE80y';
console.log(str.codePointAt(0).toString(16)); // 78
console.log(str.codePointAt(1).toString(16)); // 1f680
console.log(str.codePointAt(3).toString(16)); // 79
This method works well when combined with iteration over strings:

for (const ch of 'x\uD83D\uDE80y') {
    console.log(ch.codePointAt(0).toString(16));
}
// Output:
// 78
// 1f680
// 79
The opposite of codePointAt() is String.fromCodePoint():

> String.fromCodePoint(0x78, 0x1f680, 0x79) === 'x\uD83D\uDE80y'
true
6.6 Checking for inclusion
Three new methods check whether a string exists within another string:

> 'hello'.startsWith('hell')
true
> 'hello'.endsWith('ello')
true
> 'hello'.includes('ell')
true
Each of these methods has a position as an optional second parameter, which specifies where the string to be searched starts or ends:

> 'hello'.startsWith('ello', 1)
true
> 'hello'.endsWith('hell', 4)
true

> 'hello'.includes('ell', 1)
true
> 'hello'.includes('ell', 2)
false
6.7 Repeating strings
The repeat() method repeats strings:

> 'doo '.repeat(3)
'doo doo doo '
6.8 String methods that delegate regular expression work to their parameters
In ES6, the four string methods that accept regular expression parameters do relatively little. They mainly call methods of their parameters:

String.prototype.match(regexp) calls regexp[Symbol.match](this).
String.prototype.replace(searchValue, replaceValue) calls searchValue[Symbol.replace](this, replaceValue).
String.prototype.search(regexp) calls regexp[Symbol.search](this).
String.prototype.split(separator, limit) calls separator[Symbol.split](this, limit).
The parameters don’t have to be regular expressions, anymore. Any objects with appropriate methods will do.

6.9 Cheat sheet: the new string methods
Tagged templates:

String.raw(callSite, ...substitutions) : string
Template tag for “raw” content (backslashes are not interpreted):
  > String.raw`\` === '\\'
  true
Consult the chapter on template literals for more information.

Unicode and code points:

String.fromCodePoint(...codePoints : number[]) : string
Turns numbers denoting Unicode code points into a string.
String.prototype.codePointAt(pos) : number
Returns the number of the code point starting at position pos (comprising one or two JavaScript characters).
String.prototype.normalize(form? : string) : string
Different combinations of code points may look the same. Unicode normalization changes them all to the same value(s), their so-called canonical representation. That helps with comparing and searching for strings. The 'NFC' form is recommended for general text.
Finding strings:

String.prototype.startsWith(searchString, position=0) : boolean
Does the receiver start with searchString? position lets you specify where the string to be checked starts.
String.prototype.endsWith(searchString, endPosition=searchString.length) : boolean
Does the receiver end with searchString? endPosition lets you specify where the string to be checked ends.
String.prototype.includes(searchString, position=0) : boolean
Does the receiver contain searchString? position lets you specify where the string to be searched starts.
Repeating strings:

String.prototype.repeat(count) : string
Returns the receiver, concatenated count times.
