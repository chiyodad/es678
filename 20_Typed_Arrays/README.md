# 20. 타입화 배열(Typed Arrays)
## 20.1 개요
타입화 배열은 이진 데이터를 다루기 위한 ES6 API 이다.

예제:

```javascript
const typedArray = new Uint8Array([0,1,2]);
console.log(typedArray.length); // 3
typedArray[0] = 5;
const normalArray = [...typedArray]; // [5,1,2]
// 배열의 요소들은 typedArray.buffer 에 저장 된다.
// 동일한 데이터를 다른 뷰를 통해서도 볼 수 있다. 
const dataView = new DataView(typedArray.buffer);
console.log(dataView.getUint8(0)); // 5
```

Instances of `ArrayBuffer` store the binary data to be processed. Two kinds of views are used to access the data:
`ArrayBuffer` 인스턴스는 처리를 위해 이진 데이터를 저장한다. 데이터에 접근할 수 있는 View에는 다음과 같이 2가지가 있다. 

* Typed Arrays (`Uint8Array, Int16Array, Float32Array,` etc.) interpret the ArrayBuffer as an indexed sequence of elements of a single type.
* 타입화 배열(`Uint8Array`, `Int16Array`, `Float32Array` 등등)은 ArrayBuffer 를 싱글 타입의 요소의 인덱스 시퀀스로 번역한다.
* Instances of DataView let you access data as elements of several types (`Uint8, Int16, Float32,` etc.), at any byte offset inside an ArrayBuffer.
* DataView 인스턴스는 ArrayBuffer 안에 있는 어떤 바이트 오프셋에서도 몇몇 타입(`Uint8`, `Int16`, `Float32` 등등)의 요소처럼 데이터에 접근 가능하게 해준다. 

The following browser APIs support Typed Arrays ([details are mentioned in a dedicated section](http://exploringjs.com/es6/ch_typed-arrays.html#sec_browser-apis-supporting-typed-arrays)):
아래의 브라우저 API는 타입화 배열을 지원한다. ([자세한 내용은 각 섹션에서 살펴 볼 것.](http://exploringjs.com/es6/ch_typed-arrays.html#sec_browser-apis-supporting-typed-arrays)):

*   File API
*   XMLHttpRequest
*   Fetch API
*   Canvas
*   WebSockets
*   기타 등등

## 20.2 소개

Much data one encounters on the web is text: JSON files, HTML files, CSS files, JavaScript code, etc. For handling such data, JavaScript’s built-in string data type works well. However, until a few years ago, JavaScript was ill-equipped to handle binary data. On 8 February 2011, [the Typed Array Specification 1.0](https://www.khronos.org/registry/typedarray/specs/1.0/) standardized facilities for handling binary data. By now, Typed Arrays are [well supported](http://caniuse.com/#feat=typedarrays) by various engines. With ECMAScript 6, they became part of the core language and gained many methods in the process that were previously only available for Arrays (`map()`, `filter()`, etc.).

웹에서 마주치는 많은 데이터(JSON 파일, HTML 파일, CSS 파일, 자바스크립트 코드 등)는 텍스트이다. 이런 데이터를 다루는데 자바스크립트 내장(built-in) 스트링 데이터 타입은 충분히 쓸만하다. 그러나 몇 년 전만 해도 자바스크립트로 이진 데이터를 다루기는 쉽지 않았다. 2011년 8월, 이진 데이터를 위한 [타입화 배열 스펙 1.0](https://www.khronos.org/registry/typedarray/specs/1.0/)이 표준화 되었다. 이제는 타입화 배열를 다양 엔진에서 [제대로 지원한다](http://caniuse.com/#feat=typedarrays). ES6에 이르러 타입화 배열은 언어 중심(core)의 한 부분이 되었고, 이전에는 배열에서만 사용 가능했던 많은 메소드(`map()`, `filter()` 등등)도 얻게 되었다.

The main uses cases for Typed Arrays are:
다음은 타입화 배열의 주 사용 예시이다.
*   Processing binary data: manipulating image data in HTML Canvas elements, parsing binary files, handling binary network protocols, etc.
*   이진 데이터 처리 : HTML Canvas 요소에 쓰이는 이미지 데이터 조작, 이진 데이터 파싱, 이진 네트워크 프로토콜 처리 등.
*   Interacting with native APIs: Native APIs often receive and return data in a binary format, which you could neither store nor manipulate well in traditional JavaScript. That meant that whenever you were communicating with such an API, data had to be converted from JavaScript to binary and back, for every call. Typed Arrays eliminate this bottleneck. One example of communicating with native APIs is WebGL, for which Typed Arrays were initially created. Section “[History of Typed Arrays](http://www.html5rocks.com/en/tutorials/webgl/typed_arrays/#toc-history)” of the article “[Typed Arrays: Binary Data in the Browser](http://www.html5rocks.com/en/tutorials/webgl/typed_arrays/#toc-history)” (by Ilmari Heikkinen for HTML5 Rocks) has more information.
*   네이티브 API와의 통신 : 네이티브 API는 종종 이진 포맷으로 데이터를 주고 받는데, 기존의 자바스크립트에서는 이를 저장은 물론 조작도 할 수 없었다. 이는 어떤 API와 통신을 하든 호출 시마다 데이터를 자바스크립트에서 바이너리로, 그리고 그 반대로 변환해야 했다는 의미이다. 타입화 배열은 이런 병목 현상을 없애준다. 네이티브 API 중 WebGL은 타입화 배열이 만들어진 한 이유이다. 더 많은 정보는 “[타입화 배열:브라우저에서의 이진 데이터](http://www.html5rocks.com/en/tutorials/webgl/typed_arrays/#toc-history)”(Ilmari Heikkinen, HTML5 Rocks)라는 글의 “[타입화 배열의 역사](http://www.html5rocks.com/en/tutorials/webgl/typed_arrays/#toc-history)” 섹션에서 찾아 볼 수 있다. 

Two kinds of objects work together in the Typed Array API:
타입화 배열 API에서는 다음 두 가지 오브젝트가 함께 동작한다.
*   Buffers: Instances of `ArrayBuffer` hold the binary data.
*   버퍼(Buffers) : 이진 데이터를 담고 있는 `ArrayBuffer`의 인스턴스
*   Views: provide the methods for accessing the binary data. There are two kinds of views:
    *   An instance of a Typed Array constructor (`Uint8Array`, `Float64Array`, etc.) works much like a normal Array, but only allows a single type for its elements and doesn’t have holes.
    *   An instance of `DataView` lets you access data at any byte offset in the buffer, and interprets that data as one of several types (`Uint8`, `Float64`, etc.).
*  뷰(Views): 이진 데이터 접근 메소드를 제공한다. 뷰에는 두 가지가 있다.
    *   타입화 배열 생성자(`Uint8Array`, `Float64Array` 등등)의 인스턴스는 일반 배열과 매우 흡사하게 동작하지만, 오직 한 가지 타입의 요소만 허용하고 빈 요소는 갖지 않는다.
    *   `DataView` 버퍼 내 어느 바이트 오프셋에 있는 데이터에도 접근 가능하게 해주고, 몇몇 데이터 타입(`Uint8`, `Float64` 등)으로 변환해 준다.

This is a diagram of the structure of the Typed Array API (notable: all Typed Arrays have a common superclass):
다음은 타입화 배열 API 구조를 도식화한 다이어그램이다.(주의: 모든 타입화 배열은 공통 superclass 를 갖는다.)

![TypedArray](http://exploringjs.com/es6/images/typed-arrays----typed_arrays_class_diagram.jpg)

### 20.2.1Element types

The following element types are supported by the API:

| Element type | Bytes | Description | C type |
| --- | --- | --- | --- |
| Int8 | 1 | 8-bit signed integer | signed char |
| Uint8 | 1 | 8-bit unsigned integer | unsigned char |
| Uint8C | 1 | 8-bit unsigned integer (clamped conversion) | unsigned char |
| Int16 | 2 | 16-bit signed integer | short |
| Uint16 | 2 | 16-bit unsigned integer | unsigned short |
| Int32 | 4 | 32-bit signed integer | int |
| Uint32 | 4 | 32-bit unsigned integer | unsigned int |
| Float32 | 4 | 32-bit floating point | float |
| Float64 | 8 | 64-bit floating point | double |

The element type `Uint8C` is special: it is not supported by `DataView` and only exists to enable `Uint8ClampedArray`. This Typed Array is used by the `canvas` element (where it replaces `CanvasPixelArray`). The only difference between `Uint8C` and `Uint8` is how overflow and underflow are handled (as explained in the next section). It is recommended to avoid the former – [quoting Brendan Eich](https://mail.mozilla.org/pipermail/es-discuss/2015-August/043902.html):

> Just to be super-clear (and I was around when it was born), `Uint8ClampedArray` is _totally_ a historical artifact (of the HTML5 canvas element). Avoid unless you really are doing canvas-y things.

### 20.2.2 Handling overflow and underflow

Normally, when a value is out of the range of the element type, modulo arithmetic is used to convert it to a value within range. For signed and unsigned integers that means that:

*   The highest value plus one is converted to the lowest value (0 for unsigned integers).
*   The lowest value minus one is converted to the highest value.

Modulo conversion for unsigned 8-bit integers:

```javascript
> const uint8 = new Uint8Array(1);
> uint8[0] = 255; uint8[0] // highest value within range
255
> uint8[0] = 256; uint8[0] // overflow
0
> uint8[0] = 0; uint8[0] // lowest value within range
0
> uint8[0] = -1; uint8[0] // underflow
255
```
Modulo conversion for signed 8-bit integers:

```javascript
> const int8 = new Int8Array(1);
> int8[0] = 127; int8[0] // highest value within range
127
> int8[0] = 128; int8[0] // overflow
-128
> int8[0] = -128; int8[0] // lowest value within range
-128
> int8[0] = -129; int8[0] // underflow
127
```

Clamped conversion is different:

*   All underflowing values are converted to the lowest value.
*   All overflowing values are converted to the highest value.

```javascript
> const uint8c = new Uint8ClampedArray(1);
> uint8c[0] = 255; uint8c[0] // highest value within range
255
> uint8c[0] = 256; uint8c[0] // overflow
255
> uint8c[0] = 0; uint8c[0] // lowest value within range
0
> uint8c[0] = -1; uint8c[0] // underflow
0
```
### 20.2.3 Endianness

Whenever a type (such as `Uint16`) is stored as multiple bytes, _endianness_ matters:

*   Big endian: the most significant byte comes first. For example, the `Uint16` value 0xABCD is stored as two bytes – first 0xAB, then 0xCD.
*   Little endian: the least significant byte comes first. For example, the `Uint16` value 0xABCD is stored as two bytes – first 0xCD, then 0xAB.

Endianness tends to be fixed per CPU architecture and consistent across native APIs. Typed Arrays are used to communicate with those APIs, which is why their endianness follows the endianness of the platform and can’t be changed.

On the other hand, the endianness of protocols and binary files varies and is fixed across platforms. Therefore, we must be able to access data with either endianness. DataViews serve this use case and let you specify endianness when you get or set a value.

[Quoting Wikipedia on Endianness](https://en.wikipedia.org/wiki/Endianness):

*   Big-endian representation is the most common convention in data networking; fields in the protocols of the Internet protocol suite, such as IPv4, IPv6, TCP, and UDP, are transmitted in big-endian order. For this reason, big-endian byte order is also referred to as network byte order.
*   Little-endian storage is popular for microprocessors in part due to significant historical influence on microprocessor designs by Intel Corporation.

You can use the following function to determine the endianness of a platform.
```javascript
const BIG_ENDIAN = Symbol('BIG_ENDIAN');
const LITTLE_ENDIAN = Symbol('LITTLE_ENDIAN');
function getPlatformEndianness() {
    const arr32 = Uint32Array.of(0x12345678);
    const arr8 = new Uint8Array(arr32.buffer);
    switch ((arr8[0]*0x1000000) + (arr8[1]*0x10000) + (arr8[2]*0x100) + (arr8\[3])) {
        case 0x12345678:
            return BIG_ENDIAN;
        case 0x78563412:
            return LITTLE_ENDIAN;
        default:
            throw new Error('Unknown endianness');
    }
}
```

There are also platforms that arrange _words_ (pairs of bytes) with a different endianness than bytes inside words. That is called mixed endianness. Should you want to support such a platform then it is easy to extend the previous code.

### 20.2.4 Negative indices

With the bracket operator `[ ]`, you can only use non-negative indices (starting at 0). The methods of ArrayBuffers, Typed Arrays and DataViews work differently: every index can be negative. If it is, it counts backwards from the length. In other words, it is added to the length to produce a normal index. Therefore `-1` refers to the last element, `-2` to the second-last, etc. Methods of normal Arrays work the same way.

```javascript
> const ui8 = Uint8Array.of(0, 1, 2);
> ui8.slice(-1)
Uint8Array [ 2 ]
```

Offsets, on the other hand, must be non-negative. If, for example, you pass `-1` to:

```javascript
DataView.prototype.getInt8(byteOffset)
```
then you get a `RangeError`.

## 20.3 ArrayBuffers

ArrayBuffers store the data, _views_ (Typed Arrays and DataViews) let you read and change it. In order to create a DataView, you need to provide its constructor with an ArrayBuffer. Typed Array constructors can optionally create an ArrayBuffer for you.

### 20.3.1 `ArrayBuffer` constructor

The signature of the constructor is:
```javascript
ArrayBuffer(length : number)
```
Invoking this constructor via `new` creates an instance whose capacity is `length` bytes. Each of those bytes is initially 0.

### 20.3.2 Static `ArrayBuffer` methods

*   `ArrayBuffer.isView(arg)`
    Returns `true` if `arg` is an object and a view for an ArrayBuffer. Only Typed Arrays and DataViews have the required internal property `[[ViewedArrayBuffer]]`. That means that this check is roughly equivalent to checking whether `arg` is an instance of a Typed Array or of `DataView`.

### 20.3.3 `ArrayBuffer.prototype` properties

*   `get ArrayBuffer.prototype.byteLength`
    Returns the capacity of this ArrayBuffer in bytes.
*   `ArrayBuffer.prototype.slice(start, end)`
    Creates a new ArrayBuffer that contains the bytes of this ArrayBuffer whose indices are greater than or equal to `start` and less than `end`. `start` and `end` can be negative (see Sect. “[Negative indices](ch_typed-arrays.html#sec_negative-typed-array-indices)”).

## 20.4 Typed Arrays

The various kinds of Typed Array are only different w.r.t. to the type of their elements:

*   Typed Arrays whose elements are integers: `Int8Array`, `Uint8Array`, `Uint8ClampedArray`, `Int16Array`, `Uint16Array`, `Int32Array`, `Uint32Array`
*   Typed Arrays whose elements are floats: `Float32Array`, `Float64Array`

### 20.4.1 Typed Arrays versus normal Arrays

Typed Arrays are much like normal Arrays: they have a `length`, elements can be accessed via the bracket operator `[ ]` and they have all of the standard Array methods. They differ from Arrays in the following ways:

*   All of their elements have the same type, setting elements converts values to that type.
*   They are contiguous. Normal Arrays can have _holes_ (indices in the range [0, `arr.length`) that have no associated element), Typed Arrays can’t.
*   Initialized with zeros. This is a consequence of the previous item:
    *   `new Array(10)` creates a normal Array without any elements (it only has holes).
    *   `new Uint8Array(10)` creates a Typed Array whose 10 elements are all 0.
*   An associated buffer. The elements of a Typed Array `ta` are not stored in `ta`, they are stored in an associated ArrayBuffer that can be accessed via `ta.buffer`.

### 20.4.2 Typed Arrays are iterable

Typed Arrays implement a method whose key is `Symbol.iterator` and are therefore iterable (consult chapter “[Iterables and iterators](http://exploringjs.com/es6/ch_iteration.html#ch_iteration)” for more information). That means that you can use the `for-of` loop and similar mechanisms in ES6:

```javascript
const ui8 = Uint8Array.of(0,1,2);
for (const byte of ui8) {
    console.log(byte);
}
// Output:
// 0
// 1
// 2
```
ArrayBuffers and DataViews are not iterable.

### 20.4.3 Converting Typed Arrays to and from normal Arrays

To convert a normal Array to a Typed Array, you make it the parameter of a Typed Array constructor. For example:

```javascript
> const tarr = new Uint8Array([0,1,2]);
```

The classic way to convert a Typed Array to an Array is to invoke `Array.prototype.slice` on it. This trick works for all Array-like objects (such as `arguments`) and Typed Arrays are Array-like.

```javascript
> Array.prototype.slice.call(tarr)
[ 0, 1, 2 ]
```

In ES6, you can use the spread operator (`...`), because Typed Arrays are iterable:

```javascript
> [...tarr]
[ 0, 1, 2 ]
```

Another ES6 alternative is `Array.from()`, which works with either iterables or Array-like objects:
```javascript
> Array.from(tarr)
[ 0, 1, 2 ]
```

### <span class="section-number">20.4.4</span> The Species pattern for Typed Arrays

Some methods create new instances that are similar to `this`. The species pattern lets you configure what constructor should be used to do so. For example, if you create a subclass `MyArray` of `Array` then the default is that `map()` creates instances of `MyArray`. If you want it to create instances of `Array`, you can use the species pattern to make that happen. Details are explained in Sect “[The species pattern](http://exploringjs.com/es6/ch_classes.html#sec_species-pattern)” in the chapter on classes.

ArrayBuffers use the species pattern in the following locations:

*   `ArrayBuffer.prototype.slice()`
*   Whenever an ArrayBuffer is cloned inside a Typed Array or DataView.

Typed Arrays use the species pattern in the following locations:

*   `TypedArray<T>.prototype.filter()`
*   `TypedArray<T>.prototype.map()`
*   `TypedArray<T>.prototype.slice()`
*   `TypedArray<T>.prototype.subarray()`

DataViews don’t use the species pattern.

### 20.4.5 The inheritance hierarchy of Typed Arrays

As you could see in the diagram at the beginning of this chapter, all Typed Array classes (`Uint8Array` etc.) have a common superclass. I’m calling that superclass `TypedArray`, but it is not directly accessible from JavaScript (the ES6 specification calls it _the intrinsic object `%TypedArray%`_). `TypedArray.prototype` houses all methods of Typed Arrays.

### 20.4.6 Static `TypedArray` methods

Both static `TypedArray` methods are inherited by its subclasses (`Uint8Array` etc.).

#### 20.4.6.1 `TypedArray.of()`

This method has the signature:
```javascript
TypedArray.of(...items)
```

It creates a new Typed Array that is an instance of `this` (the class on which `of()` was invoked). The elements of that instance are the parameters of `of()`.

You can think of `of()` as a custom literal for Typed Arrays:

```javascript
> Float32Array.of(0.151, -8, 3.7)
Float32Array [ 0.151, -8, 3.7 ]
```
#### 20.4.6.2 `TypedArray.from()`

This method has the signature:

```javascript
TypedArray<U>.from(source : Iterable<T>, mapfn? : T => U, thisArg?)
```

It converts the iterable `source` into an instance of `this` (a Typed Array).

For example, normal Arrays are iterable and can be converted with this method:

```javascript
> Uint16Array.from([0, 1, 2])
Uint16Array [ 0, 1, 2 ]
```

Typed Arrays are iterable, too:

```javascript
> const ui16 = Uint16Array.from(Uint8Array.of(0, 1, 2));
> ui16 instanceof Uint16Array
true
```

The optional `mapfn` lets you transform the elements of `source` before they become elements of the result. Why perform the two steps _mapping_ and _conversion_ in one go? Compared to performing the first step separately, via `source.map()`, there are two advantages:

1.  No intermediate Array or Typed Array is needed.
2.  When converting a Typed Array to a Typed Array whose elements have a higher precision, the mapping step can make use of that higher precision.

To illustrate the second advantage, let’s use `map()` to double the elements of a Typed Array:

```javascript
> Int8Array.of(127, 126, 125).map(x => 2 * x)
Int8Array [ -2, -4, -6 ]
```

As you can see, the values overflow and are coerced into the `Int8` range of values. If map via `from()`, you can choose the type of the result so that values don’t overflow:

```javascript
> Int16Array.from(Int8Array.of(127, 126, 125), x => 2 * x)
Int16Array [ 254, 252, 250 ]
```

[According to Allen Wirfs-Brock](https://twitter.com/awbjs/status/585199958661472257), mapping between Typed Arrays was what motivated the `mapfn` parameter of `from()`.

### 20.4.7 `TypedArray.prototype` properties

Indices accepted by Typed Array methods can be negative (they work like traditional Array methods that way). Offsets must be non-negative. For details, see Sect. “[Negative indices](ch_typed-arrays.html#sec_negative-typed-array-indices)”.

##### 20.4.7.1 Methods specific to Typed Arrays

The following properties are specific to Typed Arrays, normal Arrays don’t have them:

*   `get TypedArray<T>.prototype.buffer : ArrayBuffer`
    Returns the buffer backing this Typed Array.
*   `get TypedArray<T>.prototype.byteLength : number`
    Returns the size in bytes of this Typed Array’s buffer.
*   `get TypedArray<T>.prototype.byteOffset : number`
    Returns the offset where this Typed Array “starts” inside its ArrayBuffer.
*   `TypedArray<T>.prototype.set(arrayOrTypedArray, offset=0) : void`
    Copies all elements of `arrayOrTypedArray` to this Typed Array. The element at index 0 of `arrayOrTypedArray` is written to index `offset` of this Typed Array (etc.).
    *   If `arrayOrTypedArray` is a normal Array, its elements are converted to numbers who are then converted to the element type `T` of this Typed Array.
    *   If `arrayOrTypedArray` is a Typed Array then each of its elements is converted directly to the appropriate type for this Typed Array. If both Typed Arrays have the same element type then faster, byte-wise copying is used.
*   `TypedArray<T>.prototype.subarray(begin=0, end=this.length) : TypedArray<T>`
    Returns a new Typed Array that has the same buffer as this Typed Array, but a (generally) smaller range. If `begin` is non-negative then the first element of the resulting Typed Array is `this[begin]`, the second `this[begin+1]` (etc.). If `begin` in negative, it is converted appropriately.

##### 20.4.7.2 Array methods

The following methods are basically the same as the methods of normal Arrays:

*   `TypedArray<T>.prototype.copyWithin(target : number, start : number, end = this.length) : This`
    Copies the elements whose indices are between `start` (including) and `end` (excluding) to indices starting at `target`. If the ranges overlap and the former range comes first then elements are copied in reverse order to avoid overwriting source elements before they are copied.
*   `TypedArray<T>.prototype.entries() : Iterable<[number,T]>`
    Returns an iterable over [index,element] pairs for this Typed Array.
*   `TypedArray<T>.prototype.every(callbackfn, thisArg?)`
    Returns `true` if `callbackfn` returns `true` for every element of this Typed Array. Otherwise, it returns `false`. `every()` stops processing the first time `callbackfn` returns `false`.
*   `TypedArray<T>.prototype.fill(value, start=0, end=this.length) : void`
    Set the elements whose indices range from `start` to `end` to `value`.
*   `TypedArray<T>.prototype.filter(callbackfn, thisArg?) : TypedArray<T>`
    Returns a Typed Array that contains every element of this Typed Array for which `callbackfn` returns `true`. In general, the result is shorter than this Typed Array.
*   `TypedArray<T>.prototype.find(predicate : T => boolean, thisArg?) : T`
    Returns the first element for which the function `predicate` returns `true`.
*   `TypedArray<T>.prototype.findIndex(predicate : T => boolean, thisArg?) : number`
    Returns the index of the first element for which `predicate` returns `true`.
*   `TypedArray<T>.prototype.forEach(callbackfn, thisArg?) : void`
    Iterates over this Typed Array and invokes `callbackfn` for each element.
*   `TypedArray<T>.prototype.indexOf(searchElement, fromIndex=0) : number`
    Returns the index of the first element that strictly equals `searchElement`. The search starts at `fromIndex`.
*   `TypedArray<T>.prototype.join(separator : string = ',') : string`
    Converts all elements to strings and concatenates them, separated by `separator`.
*   `TypedArray<T>.prototype.keys() : Iterable<number>`
    Returns an iterable over the indices of this Typed Array.
*   `TypedArray<T>.prototype.lastIndexOf(searchElement, fromIndex?) : number`
    Returns the index of the last element that strictly equals `searchElement`. The search starts at `fromIndex`, backwards.
*   `get TypedArray<T>.prototype.length : number`
    Returns the length of this Typed Array.
*   `TypedArray<T>.prototype.map(callbackfn, thisArg?) : TypedArray<T>`
    Returns a new Typed Array in which every element is the result of applying `callbackfn` to the corresponding element of this Typed Array.
*   `TypedArray<T>.prototype.reduce(callbackfn : (previousValue : any, currentElement : T, currentIndex : number, array : TypedArray<T>) => any, initialValue?) : any`
    `callbackfn` is fed one element at a time, together with the result that was computed so far and computes a new result. Elements are visited from left to right.
*   `TypedArray<T>.prototype.reduceRight(callbackfn : (previousValue : any, currentElement : T, currentIndex : number, array : TypedArray<T>) => any, initialValue?) : any`
    `callbackfn` is fed one element at a time, together with the result that was computed so far and computes a new result. Elements are visited from right to left.
*   `TypedArray<T>.prototype.reverse() : This`
    Reverses the order of the elements of this Typed Array and returns `this`.
*   `TypedArray<T>.prototype.slice(start=0, end=this.length) : TypedArray<T>`
    Create a new Typed Array that only has the elements of this Typed Array whose indices are between `start` (including) and `end` (excluding).
*   `TypedArray<T>.prototype.some(callbackfn, thisArg?)`
    Returns `true` if `callbackfn` returns `true` for at least one element of this Typed Array. Otherwise, it returns `false`. `some()` stops processing the first time `callbackfn` returns `true`.
*   `TypedArray<T>.prototype.sort(comparefn? : (number, number) => number)`
    Sorts this Typed Array, as specified via `comparefn`. If `comparefn` is missing, sorting is done ascendingly, by comparing via the less-than operator (`<`).
*   `TypedArray<T>.prototype.toLocaleString(reserved1?, reserved2?)`
*   `TypedArray<T>.prototype.toString()`
*   `TypedArray<T>.prototype.values() : Iterable<T>`
    Returns an iterable over the values of this Typed Array.

Due to all of these methods being available for Arrays, you can consult the following two sources to find out more about how they work:

*   The following methods are new in ES6 and explained in chapter “[New Array features](ch_arrays.html#ch_arrays)”: `copyWithin`, `entries`, `fill`, `find`, `findIndex`, `keys`, `values`.
*   All other methods are explained in chapter “[Arrays](http://speakingjs.com/es5/ch18.html)” of “Speaking JavaScript”.

Note that while normal Array methods are generic (any Array-like `this` is OK), the methods listed in this section are not (`this` must be a Typed Array).

### 20.4.8 `«ElementType»Array` constructor

Each Typed Array constructor has a name that follows the pattern `«ElementType»Array`, where `«ElementType»` is one of the element types in the table at the beginning. That means that there are 9 constructors for Typed Arrays: `Int8Array`, `Uint8Array`, `Uint8ClampedArray` (element type `Uint8C`), `Int16Array`, `Uint16Array`, `Int32Array`, `Uint32Array`, `Float32Array`, `Float64Array`.

Each constructor has five _overloaded_ versions – it behaves differently depending on how many arguments it receives and what their types are:

*   `«ElementType»Array(buffer, byteOffset=0, length?)`
    Creates a new Typed Array whose buffer is `buffer`. It starts accessing the buffer at the given `byteOffset` and will have the given `length`. Note that `length` counts elements of the Typed Array (with 1–4 bytes each), not bytes.
*   `«ElementType»Array(length)`
    Creates a Typed Array with the given `length` and the appropriate buffer (whose size in bytes is `length * «ElementType»Array.BYTES_PER_ELEMENT`).
*   `«ElementType»Array()`
    Creates a Typed Array whose `length` is 0\. It also creates an associated empty ArrayBuffer.
*   `«ElementType»Array(typedArray)`
    Creates a new Typed Array that has the same length and elements as `typedArray`. Values that are too large or small are converted appropriately.
*   `«ElementType»Array(arrayLikeObject)`
    Treats `arrayLikeObject` like an Array and creates a new TypedArray that has the same length and elements. Values that are too large or small are converted appropriately.

The following code shows three different ways of creating the same Typed Array:

```javascript
const tarr1 = new Uint8Array([1,2,3]);

const tarr2 = Uint8Array.of(1,2,3);

const tarr3 = new Uint8Array(3);
tarr3[0] = 0;
tarr3[1] = 1;
tarr3[2] = 2;
```
### 20.4.9 Static `«ElementType»Array` properties

*   `«ElementType»Array.BYTES_PER_ELEMENT`
    Counts how many bytes are needed to store a single element:

    ```javascript
   > Uint8Array.BYTES_PER_ELEMENT
   1
   > Int16Array.BYTES_PER_ELEMENT
   2
   > Float64Array.BYTES_PER_ELEMENT
   8
    ```

### 20.4.10 `«ElementType»Array.prototype` properties

*   `«ElementType»Array.prototype.BYTES_PER_ELEMENT`
    The same as `«ElementType»Array.BYTES_PER_ELEMENT`.

#### <span class="section-number">20.4.11</span> Concatenating Typed Arrays

Typed Arrays don’t have a method `concat()`, like normal Arrays do. The work-around is to use the method

```javascript
typedArray.set(arrayOrTypedArray, offset=0)
```

That method copies an existing Typed Array (or normal Array) into `typedArray` at index `offset`. Then you only have to make sure that `typedArray` is big enough to hold all (Typed) Arrays you want to concatenate:

```javascript
function concatenate(resultConstructor, ...arrays) {
    let totalLength = 0;
    for (const arr of arrays) {
        totalLength += arr.length;
    }
    const result = new resultConstructor(totalLength);
    let offset = 0;
    for (const arr of arrays) {
        result.set(arr, offset);
        offset += arr.length;
    }
    return result;
}
console.log(concatenate(Uint8Array, Uint8Array.of(1, 2), Uint8Array.of(3, 4)));
// Uint8Array [1, 2, 3, 4]
```
## 20.5 DataViews

### 20.5.1 `DataView` constructor

*   `DataView(buffer, byteOffset=0, byteLength=buffer.byteLength-byteOffset)`
    Creates a new DataView whose data is stored in the ArrayBuffer `buffer`. By default, the new DataView can access all of `buffer`, the last two parameters allow you to change that.

### 20.5.2 `DataView.prototype` properties

*   `get DataView.prototype.buffer`
    Returns the ArrayBuffer of this DataView.
*   `get DataView.prototype.byteLength`
    Returns how many bytes can be accessed by this DataView.
*   `get DataView.prototype.byteOffset`
    Returns at which offset this DataView starts accessing the bytes in its buffer.
*   `DataView.prototype.get«ElementType»(byteOffset, littleEndian=false)`
    Reads a value from the buffer of this DataView.
    *   `«ElementType»` can be: `Float32`, `Float64`, `Int8`, `Int16`, `Int32`, `Uint8`, `Uint16`, `Uint32`
*   `DataView.prototype.set«ElementType»(byteOffset, value, littleEndian=false)`
    Writes `value` to the buffer of this DataView.
    *   `«ElementType»` can be: `Float32`, `Float64`, `Int8`, `Int16`, `Int32`, `Uint8`, `Uint16`, `Uint32`

### 20.6 Browser APIs that support Typed Arrays

Typed Arrays have been around for a while, so there are quite a few browser APIs that support them.

#### 20.6.1 File API

[The file API](http://www.w3.org/TR/FileAPI/) lets you access local files. The following code demonstrates how to get the bytes of a submitted local file in an ArrayBuffer.

```javascript
const fileInput = document.getElementById('fileInput');
const file = fileInput.files[0];
const reader = new FileReader();
reader.readAsArrayBuffer(file);
reader.onload = function () {
    const arrayBuffer = reader.result;
    ···
};
```

#### 20.6.2 `XMLHttpRequest`

In newer versions of [the `XMLHttpRequest` API](http://www.w3.org/TR/XMLHttpRequest/), you can have the results delivered in an ArrayBuffer:

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', someUrl);
xhr.responseType = 'arraybuffer';

xhr.onload = function () {
    const arrayBuffer = xhr.response;
    ···
};

xhr.send();
```

#### 20.6.3 Fetch API

Similarly to `XMLHttpRequest`, [the Fetch API](https://fetch.spec.whatwg.org/) lets you request resources. But it is based on Promises, which makes it more convenient to use. The following code demonstrates how to download the content pointed to by `url` as an ArrayBuffer:

```javascript
fetch(url)
.then(request => request.arrayBuffer())
.then(arrayBuffer => ···);
```

### 20.6.4 Canvas

[Quoting the HTML5 specification](http://www.w3.org/TR/html5/scripting-1.html#the-canvas-element):

> The `canvas` element provides scripts with a resolution-dependent bitmap canvas, which can be used for rendering graphs, game graphics, art, or other visual images on the fly.

[The 2D Context of `canvas`](http://www.w3.org/TR/2dcontext/) lets you retrieve the bitmap data as an instance of `Uint8ClampedArray`:

```javascript
const canvas = document.getElementById('my_canvas');
const context = canvas.getContext('2d');
const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
const uint8ClampedArray = imageData.data;
```

### 20.6.5 WebSockets

[WebSockets](http://www.w3.org/TR/websockets/) let you send and receive binary data via ArrayBuffers:

```javascript
const socket = new WebSocket('ws://127.0.0.1:8081');
socket.binaryType = 'arraybuffer';

// Wait until socket is open
socket.addEventListener('open', function (event) {
    // Send binary data
    const typedArray = new Uint8Array(4);
    socket.send(typedArray.buffer);
});

// Receive binary data
socket.addEventListener('message', function (event) {
    const arrayBuffer = event.data;
    ···
});
```

### <span class="section-number">20.6.6</span> Other APIs

*   [WebGL](https://www.khronos.org/registry/webgl/specs/latest/2.0/) uses the Typed Array API for: accessing buffer data, specifying pixels for texture mapping, reading pixel data, and more.
*   [The Web Audio API](http://www.w3.org/TR/webaudio/) lets you [decode audio data](http://www.w3.org/TR/webaudio/#dfn-decodeAudioData) submitted via an ArrayBuffer.
*   [Media Source Extensions](http://www.w3.org/TR/media-source/): The HTML media elements are currently `<audio>` and `<video>`. The Media Source Extensions API enables you to create streams to be played via those elements. You can add binary data to such streams via ArrayBuffers, Typed Arrays or DataViews.
*   Communication with [Web Workers](http://www.w3.org/TR/workers/): If you send data to a Worker via [`postMessage()`](http://www.w3.org/TR/workers/#dom-worker-postmessage), either the message (which will be cloned) or the transferable objects can contain ArrayBuffers.
*   [Cross-document communication](https://html.spec.whatwg.org/multipage/comms.html#crossDocumentMessages): works similarly to communication with Web Workers and also uses the method `postMessage()`.

## 20.7 Extended example: JPEG SOF0 decoder

[Github](http://exploringjs.com/es6/images/leanpub_github-alt.png) The code of the following example is [on GitHub](https://github.com/rauschma/typed-array-demos). And you can [run it online](http://rauschma.github.io/typed-array-demos/).

The example is a web pages that lets you upload a JPEG file and parses its structure to determine the height and the width of the image and more.

### 20.7.1 The JPEG file format

A JPEG file is a sequence of _segments_ (typed data). Each segment starts with the following four bytes:

*   Marker (two bytes): declares what kind of data is stored in the segment. The first of the two bytes is always 0xFF. Each of the standard markers has a human readable name. For example, the marker 0xFFC0 has the name “Start Of Frame (Baseline DCT)”, short: “SOF0”.
*   Length of segment (two bytes): how long is this segment (in bytes, including the length itself)?

JPEG files are big-endian on all platforms. Therefore, this example demonstrates how important it is that we can specify endianness when using DataViews.

### 20.7.2 The JavaScript code

The following function `processArrayBuffer()` is an abridged version of the actual code; I’ve removed a few error checks to reduce clutter. `processArrayBuffer()` receives an ArrayBuffer with the contents of the submitted JPEG file and iterates over its segments.

```javascript
// JPEG is big endian
var IS_LITTLE_ENDIAN = false;

function processArrayBuffer(arrayBuffer) {
    try {
        var dv = new DataView(arrayBuffer);
        ···
        var ptr = 2;
        while (true) {
            ···
            var lastPtr = ptr;
            enforceValue(0xFF, dv.getUint8(ptr),
                'Not a marker');
            ptr++;
            var marker = dv.getUint8(ptr);
            ptr++;
            var len = dv.getUint16(ptr, IS_LITTLE_ENDIAN);
            ptr += len;
            logInfo('Marker: '+hex(marker)+' ('+len+' byte(s))');
            ···

            // Did we find what we were looking for?
            if (marker === 0xC0) { // SOF0
                logInfo(decodeSOF0(dv, lastPtr));
                break;
            }
        }
    } catch (e) {
        logError(e.message);
    }
}
```

This code uses the following helper functions (that are not shown here):

*   `enforceValue()` throws an error if the expected value (first parameter) doesn’t match the actual value (second parameter).
*   `logInfo()` and `logError()` display messages on the page.
*   `hex()` turns a number into a string with two hexadecimal digits.

`decodeSOF0()` parses the segment SOF0:


```javascript
function decodeSOF0(dv, start) {
    // Example (16x16):
    // FF C0 00 11 08 00 10 00 10 03 01 22 00 02 11 01 03 11 01
    var data = {};
    start += 4; // skip marker 0xFFC0 and segment length 0x0011
    var data = {
        bitsPerColorComponent: dv.getUint8(start), // usually 0x08
        imageHeight: dv.getUint16(start+1, IS_LITTLE_ENDIAN),
        imageWidth: dv.getUint16(start+3, IS_LITTLE_ENDIAN),
        numberOfColorComponents: dv.getUint8(start+5),
    };
    return JSON.stringify(data, null, 4);
}
```

More information on the structure of JPEG files:

*   “[JPEG: Syntax and structure](https://en.wikipedia.org/wiki/JPEG#Syntax_and_structure)” (on Wikipedia)
*   “[JPEG File Interchange Format: File format structure](https://en.wikipedia.org/wiki/JPEG_File_Interchange_Format#File_format_structure)” (on Wikipedia)

## 20.8 Availability

Much of the Typed Array API is implemented by all modern JavaScript engines, but several features are new to ECMAScript 6:

*   Static methods borrowed from Arrays: `TypedArray<T>.from()`, `TypedArray<T>.of()`
*   Prototype methods borrowed from Arrays: `TypedArray<T>.prototype.map()` etc.
*   Typed Arrays are iterable
*   Support for the species pattern
*   An inheritance hierarchy where `TypedArray<T>` is the superclass of all Typed Array classes

It may take a while until these are available everywhere. As usual, kangax’ “[ES6 compatibility table](https://kangax.github.io/compat-table/es6/#typed_arrays)” describes the status quo.

