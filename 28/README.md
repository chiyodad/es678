# 28. Metaprogramming with proxies
이 단원에선 ECMAScript 6의 특징 프록시를 설명합니다. 프록시는 object에 대한 연산 수행을 가로채 사용자정의 할 수 있게 합니다.
`This chapter explains the ECMAScript 6 feature proxies. Proxies enable you to intercept and customize operations performed on objects (such as getting properties). They are a metaprogramming feature.`

## 28.1 개요
다음 예제에서, 프록시는 오브젝트의 연산을 가로채기 하며 핸들러는 가로채기를 처리하는 오브젝트입니다. 이때는 단지 get( 읽기속성 ) 하나의 연산을 가로챕니다.
`In the following example, proxy is the object whose operations we are intercepting and handler is the object that handles the interceptions. In this case, we are only intercepting a single operation, get (getting properties).`
```javascript
const target = {};
const handler = {
    get(target, propKey, receiver) {
        console.log('get ' + propKey);
        return 123;
    }
};
const proxy = new Proxy(target, handler);
```
우리가 프록시의 proxy.foo속성을 얻을 때 핸들러는 연산을 가로챕니다.
`When we get the property proxy.foo, the handler intercepts that operation:`
```javascript
> proxy.foo
get foo
123
```
[이 단원의 마지막 섹션](http://exploringjs.com/es6/ch_proxies.html#sec_reference-proxy-api)에서 API와 가로챌 수 있는 연산들 목록을 참조로 제공합니다.
`A section at the end of this chapter serves as a reference to the complete API and lists what operations can be intercepted.`

## 28.2 프로그래밍 vs 메타프로그래밍
프록시가 무엇인지 그리고 왜 유용한지를 알기 전에 우리는 첫째로 메타프로그래밍이 무엇인지 이해해야합니다.
`Before we can get into what proxies are and why they are useful, we first need to understand what metaprogramming is.`

프로그래밍에는 수준이 있습니다.
`In programming, there are levels:`
* 기초수준 (애플리케이션 수준), 사용자 입력 처리 코드.`At the base level (also called: application level), code processes user input.`
* 메타수준, 기초단계 처리 코드.`At the meta level, code processes base level code.`

기초와 메타수준은 다른 언어가 될 수 있습니다. 다음 메타프로그램에서, 메타프로그래밍 언어는 자바스크립트이고 기초프로그래밍 언어는 자바입니다.
`Base and meta level can be different languages. In the following meta program, the metaprogramming language is JavaScript and the base programming language is Java.`
```javascript
const str = 'Hello' + '!'.repeat(3);
console.log('System.out.println("'+str+'")');
```
메타프로그래밍은 다른 형태를 취할 수 있습니다. 이전 예제에서, 우리는 자바코드를 콘솔에 출력했습니다. 자, 메타프로그래밍 언어와 기초프로그래밍 언어 둘다 자바스크립트로 사용해봅시다. eval()함수는 자바스크립트 코드를 즉석에서 평가/컴파일 하는 전형적인 예입니다. eval() 실제 사용사례가 많지는 않습니다. 아래의 interaction에서, 우리는 5 + 2 표현식을 평가하는데 eval()을 사용했습니다.
`Metaprogramming can take different forms. In the previous example, we have printed Java code to the console. Let’s use JavaScript as both metaprogramming language and base programming language. The classic example for this is the eval() function, which lets you evaluate/compile JavaScript code on the fly. There are not that many actual use cases for eval(). In the interaction below, we use it to evaluate the expression 5 + 2.`
```javascript
> eval('5 + 2')
7
```
다른 자바스크립트 연산들은 아마 메타프로그래밍처럼 보이지 않을 수 있습니다. 하지만 실제로 자세히 보면:
`Other JavaScript operations may not look like metaprogramming, but actually are, if you look closer:`
```javascript
// 기초수준
const obj = {
    hello() {
        console.log('Hello!');
    }
};

// 메타수준
for (const key of Object.keys(obj)) {
    console.log(key);
}
```
프로그램은 실행 하는 동안 자신의 구조를 조사합니다. 이것은 메타프로그래밍처럼 보이지 않습니다. 왜냐하면 자바스크립트에서는 프로그래밍 구조와 데이터 구조 사이의 구분이 흐릿(불분명? 모호?)하기 때문입니다. 모든 Object.* 메소드들은 메타프로그래밍 기능으로 고려될 수 있습니다.
`The program is examining its own structure while running. This doesn’t look like metaprogramming, because the separation between programming constructs and data structures is fuzzy in JavaScript. All of the Object.* methods can be considered metaprogramming functionality.`

### 28.2.1 메타프로그래밍의 유형들
반영 메타프로그래밍은 프로그램 자체를 처리하는 것을 의미합니다. [Kiczales외[2].](http://exploringjs.com/es6/ch_proxies.html) 반영 메타프로그래밍의 유형을 세가지로 구별합니다.
`Reflective metaprogramming means that a program processes itself. Kiczales et al. [2] distinguish three kinds of reflective metaprogramming:`

* 자기관찰 : 프로그램의 구조에 읽기속성으로 접근합니다.`Introspection: you have read-only access to the structure of a program.`
* 자체수정 : 구조를 변경할 수 있습니다. `Self-modification: you can change that structure`
* 중재 : 어떤 언어 연산들의 의미를 재정의 할 수 있다`Intercession: you can redefine the semantics of some language operations.`

자 예제를 봅시다.
`Let’s look at examples.`

예: 자기관찰. Object.keys()는 자기관찰을 수행 합니다.(이전 예제를 보세요.)
`Example: introspection. Object.keys() performs introspection (see previous example).`

예: 자체수정. 다음 moveProperty함수는 source에서 target으로 하나의 속성을 옮깁니다. moveProperty함수는 속성에 접근하는 대괄호 연산자, 할당연산자 그리고 삭제연산자를 통해 자체수정을 수행합니다.(생산코드에서, 당신은 아마 이 작업에 대한 속성 설명자를 사용해야합니다.)
`Example: self-modification. The following function moveProperty moves a property from a source to a target. It performs self-modification via the bracket operator for property access, the assignment operator and the delete operator. (In production code, you’d probably use property descriptors for this task.)`
```javascript
function moveProperty(source, propertyName, target) {
    target[propertyName] = source[propertyName];
    delete source[propertyName];
}
```
moveProperty() 사용: `Using moveProperty():`
```javascript
> const obj1 = { prop: 'abc' };
> const obj2 = {};
> moveProperty(obj1, 'prop', obj2);

> obj1
{}
> obj2
{ prop: 'abc' }
```
ECMAScript 5는 중재를 지원하지 않습니다; 프록시는 그 공백을 채우기 위해 만들어졌습니다.
`ECMAScript 5 doesn’t support intercession; proxies were created to fill that gap.`

## 28.3 프록시 처음보기
`## 28.3 A first look at proxies`
ECMAScript 6 프록시는 자바스크립트에서 중재를 하게합니다. 프록시는 다음과 같이 동작합니다. 오브젝트 obj에서 수행할 수 있는 많은 연산들이 있습니다. 예를들어:
`ECMAScript 6 proxies bring intercession to JavaScript. They work as follows. There are many operations that you can perform on an object obj. For example:`

* 오브젝트 obj에서 속성 prop가져오기 (obj.prop) `Getting the property prop of an object obj (obj.prop)`
* 오브젝트 obj가 속성 prop을 가졌는지 확인하기 ('prop' in obj) `Checking whether an object obj has a property prop ('prop' in obj)`

프록시는 명령의 일부를 사용자정의 하도록 허락하는 특별한 오브젝트입니다. 프록시는 두 인자로 만들어집니다:
`Proxies are special objects that allow you customize some of these operations. A proxy is created with two parameters:`

* 핸들러: 
`handler: For each operation, there is a corresponding handler method that – if present – performs that operation. Such a method intercepts the operation (on its way to the target) and is called a trap (a term borrowed from the domain of operating systems).`
* 타겟: `target: If the handler doesn’t intercept an operation then it is performed on the target. That is, it acts as a fallback for the handler. In a way, the proxy wraps the target.`


In the following example, the handler intercepts the operations get and has.
```javascript
const target = {};
const handler = {
    /** Intercepts: getting properties */
    get(target, propKey, receiver) {
        console.log(`GET ${propKey}`);
        return 123;
    },

    /** Intercepts: checking whether properties exist */
    has(target, propKey) {
        console.log(`HAS ${propKey}`);
        return true;
    }
};
const proxy = new Proxy(target, handler);
```
When we get property foo, the handler intercepts that operation:
```javascript
> proxy.foo
GET foo
123
```
Similarly, the in operator triggers has:
```javascript
> 'hello' in proxy
HAS hello
true
```
The handler doesn’t implement the trap set (setting properties). Therefore, setting proxy.bar is forwarded to target and leads to target.bar being set.
```javascript
> proxy.bar = 'abc';
> target.bar
'abc'
```
### 28.3.1 Function-specific traps
If the target is a function, two additional operations can be intercepted:

apply: Making a function call, triggered via
proxy(···)
proxy.call(···)
proxy.apply(···)
construct: Making a constructor call, triggered via
new proxy(···)
The reason for only enabling these traps for function targets is simple: You wouldn’t be able to forward the operations apply and construct, otherwise.

### 28.3.2 Intercepting method calls
If you want to intercept method calls via a proxy, there is one challenge: you can intercept the operation get (getting property values) and you can intercept the operation apply (calling a function), but there is no single operation for method calls that you could intercept. That’s because method calls are viewed as two separate operations: First a get to retrieve a function, then an apply to call that function.

Therefore, you must intercept get and return a function that intercepts the function call. The following code demonstrates how that is done.
```javascript
function traceMethodCalls(obj) {
    const handler = {
        get(target, propKey, receiver) {
            const origMethod = target[propKey];
            return function (...args) {
                const result = origMethod.apply(this, args);
                console.log(propKey + JSON.stringify(args)
                    + ' -> ' + JSON.stringify(result));
                return result;
            };
        }
    };
    return new Proxy(obj, handler);
}
```
I’m not using a Proxy for the latter task, I’m simply wrapping the original method with a function.

Let’s use the following object to try out traceMethodCalls():
```javascript
const obj = {
    multiply(x, y) {
        return x * y;
    },
    squared(x) {
        return this.multiply(x, x);
    },
};
```
tracedObj is a traced version of obj. The first line after each method call is the output of console.log(), the second line is the result of the method call.
```javascript
> const tracedObj = traceMethodCalls(obj);
> tracedObj.multiply(2,7)
multiply[2,7] -> 14
14
> tracedObj.squared(9)
multiply[9,9] -> 81
squared[9] -> 81
81
```
The nice thing is that even the call this.multiply() that is made inside obj.squared() is traced. That’s because this keeps referring to the proxy.

This is not the most efficient solution. One could, for example, cache methods. Furthermore, Proxies themselves have an impact on performance.

### 28.3.3 Revocable proxies
ECMAScript 6 lets you create proxies that can be revoked (switched off):

const {proxy, revoke} = Proxy.revocable(target, handler);
On the left hand side of the assignment operator (=), we are using destructuring to access the properties proxy and revoke of the object returned by Proxy.revocable().

After you call the function revoke for the first time, any operation you apply to proxy causes a TypeError. Subsequent calls of revoke have no further effect.
```javascript
const target = {}; // Start with an empty object
const handler = {}; // Don’t intercept anything
const {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
console.log(proxy.foo); // 123

revoke();

console.log(proxy.foo); // TypeError: Revoked
```
### 28.3.4 Proxies as prototypes
A proxy proto can become the prototype of an object obj. Some operations that begin in obj may continue in proto. One such operation is get.
```javascript
const proto = new Proxy({}, {
    get(target, propertyKey, receiver) {
        console.log('GET '+propertyKey);
        return target[propertyKey];
    }
});

const obj = Object.create(proto);
obj.bla;
// Output:
// GET bla
```
The property bla can’t be found in obj, which is why the search continues in proto and the trap get is triggered there. There are more operations that affect prototypes; they are listed at the end of this chapter.

### 28.3.5 Forwarding intercepted operations
Operations whose traps the handler doesn’t implement are automatically forwarded to the target. Sometimes there is some task you want to perform in addition to forwarding the operation. For example, a handler that intercepts all operations and logs them, but doesn’t prevent them from reaching the target:
```javascript
const handler = {
    deleteProperty(target, propKey) {
        console.log('DELETE ' + propKey);
        return delete target[propKey];
    },
    has(target, propKey) {
        console.log('HAS ' + propKey);
        return propKey in target;
    },
    // Other traps: similar
}
```
For each trap, we first log the name of the operation and then forward it by performing it manually. ECMAScript 6 has the module-like object Reflect that helps with forwarding: for each trap

handler.trap(target, arg_1, ···, arg_n)
Reflect has a method

Reflect.trap(target, arg_1, ···, arg_n)
If we use Reflect, the previous example looks as follows.
```javascript
const handler = {
    deleteProperty(target, propKey) {
        console.log('DELETE ' + propKey);
        return Reflect.deleteProperty(target, propKey);
    },
    has(target, propKey) {
        console.log('HAS ' + propKey);
        return Reflect.has(target, propKey);
    },
    // Other traps: similar
}
```
Now what each of the traps does is so similar that we can implement the handler via a proxy:
```javascript
const handler = new Proxy({}, {
    get(target, trapName, receiver) {
        // Return the handler method named trapName
        return function (...args) {
            // Don’t log args[0]
            console.log(trapName.toUpperCase()+' '+args.slice(1));
            // Forward the operation
            return Reflect[trapName](...args);
        }
    }
});
```
For each trap, the proxy asks for a handler method via the get operation and we give it one. That is, all of the handler methods can be implemented via the single meta method get. It was one of the goals for the proxy API to make this kind of virtualization simple.

Let’s use this proxy-based handler:
```javascript
> const target = {};
> const proxy = new Proxy(target, handler);
> proxy.foo = 123;
SET foo,123,[object Object]
> proxy.foo
GET foo,[object Object]
123
```
The following interaction confirms that the set operation was correctly forwarded to the target:
```javascript
> target.foo
123
28.4 Use cases for proxies
```
This section demonstrates what proxies can be used for. That will give you the opportunity to see the API in action.

### 28.4.1 Tracing property accesses (get, set)
Let’s assume we have a function tracePropAccess(obj, propKeys) that logs whenever a property of obj, whose key is in the Array propKeys, is set or got. In the following code, we apply that function to an instance of the class Point:
```javascript
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `Point(${this.x}, ${this.y})`;
    }
}
// Trace accesses to properties `x` and `y`
const p = new Point(5, 7);
p = tracePropAccess(p, ['x', 'y']);
```
Getting and setting properties of the traced object p has the following effects:
```javascript
> p.x
GET x
5
> p.x = 21
SET x=21
21
```
Intriguingly, tracing also works whenever Point accesses the properties, because this now refers to the traced object, not to an instance of Point.
```javascript
> p.toString()
GET x
GET y
'Point(21, 7)'
```
In ECMAScript 5, you’d implement tracePropAccess() as follows. We replace each property with a getter and a setter that traces accesses. The setters and getters use an extra object, propData, to store the data of the properties. Note that we are destructively changing the original implementation, which means that we are metaprogramming.
```javascript
function tracePropAccess(obj, propKeys) {
    // Store the property data here
    const propData = Object.create(null);
    // Replace each property with a getter and a setter
    propKeys.forEach(function (propKey) {
        propData[propKey] = obj[propKey];
        Object.defineProperty(obj, propKey, {
            get: function () {
                console.log('GET '+propKey);
                return propData[propKey];
            },
            set: function (value) {
                console.log('SET '+propKey+'='+value);
                propData[propKey] = value;
            },
        });
    });
    return obj;
}
```
In ECMAScript 6, we can use a simpler, proxy-based solution. We intercept property getting and setting and don’t have to change the implementation.
```javascript
function tracePropAccess(obj, propKeys) {
    const propKeySet = new Set(propKeys);
    return new Proxy(obj, {
        get(target, propKey, receiver) {
            if (propKeySet.has(propKey)) {
                console.log('GET '+propKey);
            }
            return Reflect.get(target, propKey, receiver);
        },
        set(target, propKey, value, receiver) {
            if (propKeySet.has(propKey)) {
                console.log('SET '+propKey+'='+value);
            }
            return Reflect.set(target, propKey, value, receiver);
        },
    });
}
```
### 28.4.2 Warning about unknown properties (get, set)
When it comes to accessing properties, JavaScript is very forgiving. For example, if you try to read a property and misspell its name, you don’t get an exception, you get the result undefined. You can use proxies to get an exception in such a case. This works as follows. We make the proxy a prototype of an object.

If a property isn’t found in the object, the get trap of the proxy is triggered. If the property doesn’t even exist in the prototype chain after the proxy, it really is missing and we throw an exception. Otherwise, we return the value of the inherited property. We do so by forwarding the get operation to the target (the prototype of the target is also the prototype of the proxy).

```javascript
const PropertyChecker = new Proxy({}, {
    get(target, propKey, receiver) {
        if (!(propKey in target)) {
            throw new ReferenceError('Unknown property: '+propKey);
        }
        return Reflect.get(target, propKey, receiver);
    }
});
```
Let’s use PropertyChecker for an object that we create:

```javascript
> const obj = { __proto__: PropertyChecker, foo: 123 };
> obj.foo  // own
123
> obj.fo
ReferenceError: Unknown property: fo
> obj.toString()  // inherited
'[object Object]'
```
If we turn PropertyChecker into a constructor, we can use it for ECMAScript 6 classes via extends:

```javascript
function PropertyChecker() { }
PropertyChecker.prototype = new Proxy(···);

class Point extends PropertyChecker {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
}

const p = new Point(5, 7);
console.log(p.x); // 5
console.log(p.z); // ReferenceError
```
If you are worried about accidentally creating properties, you have two options: You can either wrap a proxy around objects that traps set. Or you can make an object obj non-extensible via Object.preventExtensions(obj), which means that JavaScript doesn’t let you add new (own) properties to obj.

### 28.4.3 Negative Array indices (get)
Some Array methods let you refer to the last element via -1, to the second-to-last element via -2, etc. For example:

```javascript
> ['a', 'b', 'c'].slice(-1)
[ 'c' ]
```
Alas, that doesn’t work when accessing elements via the bracket operator ([]). We can, however, use proxies to add that capability. The following function createArray() creates Arrays that support negative indices. It does so by wrapping proxies around Array instances. The proxies intercept the get operation that is triggered by the bracket operator.

```javascript
function createArray(...elements) {
    const handler = {
        get(target, propKey, receiver) {
            // Sloppy way of checking for negative indices
            const index = Number(propKey);
            if (index < 0) {
                propKey = String(target.length + index);
            }
            return Reflect.get(target, propKey, receiver);
        }
    };
    // Wrap a proxy around an Array
    const target = [];
    target.push(...elements);
    return new Proxy(target, handler);
}
const arr = createArray('a', 'b', 'c');
console.log(arr[-1]); // c
```
Acknowledgement: The idea for this example comes from a blog post by hemanth.hm.

### 28.4.4 Data binding (set)
Data binding is about syncing data between objects. One popular use case are widgets based on the MVC (Model View Controler) pattern: With data binding, the view (the widget) stays up-to-date if you change the model (the data visualized by the widget).

To implement data binding, you have to observe and react to changes made to an object. In the following code snippet, I sketch how observing changes could work for an Array.

```javascript
function createObservedArray(callback) {
    const array = [];
    return new Proxy(array, {
        set(target, propertyKey, value, receiver) {
            callback(propertyKey, value);
            return Reflect.set(target, propertyKey, value, receiver);
        }
    });    
}
const observedArray = createObservedArray(
    (key, value) => console.log(`${key}=${value}`));
observedArray.push('a');
```
Output:
```javascript
0=a
length=1
```
### 28.4.5 Accessing a restful web service (method calls)
A proxy can be used to create an object on which arbitrary methods can be invoked. In the following example, the function createWebService creates one such object, service. Invoking a method on service retrieves the contents of the web service resource with the same name. Retrieval is handled via an ECMAScript 6 Promise.

```javascript
const service = createWebService('http://example.com/data');
// Read JSON data in http://example.com/data/employees
service.employees().then(json => {
    const employees = JSON.parse(json);
    ···
});
```
The following code is a quick and dirty implementation of createWebService in ECMAScript 5. Because we don’t have proxies, we need to know beforehand what methods will be invoked on service. The parameter propKeys provides us with that information, it holds an Array with method names.

```javascript
function createWebService(baseUrl, propKeys) {
    const service = {};
    propKeys.forEach(function (propKey) {
        service[propKey] = function () {
            return httpGet(baseUrl+'/'+propKey);
        };
    });
    return service;
}
```
The ECMAScript 6 implementation of createWebService can use proxies and is simpler:

```javascript
function createWebService(baseUrl) {
    return new Proxy({}, {
        get(target, propKey, receiver) {
            // Return the method to be called
            return () => httpGet(baseUrl+'/'+propKey);
        }
    });
}
```
Both implementations use the following function to make HTTP GET requests (how it works is explained in the chapter on Promises.

```javascript
function httpGet(url) {
    return new Promise(
        (resolve, reject) => {
            const request = new XMLHttpRequest();
            Object.assign(request, {
                onload() {
                    if (this.status === 200) {
                        // Success
                        resolve(this.response);
                    } else {
                        // Something went wrong (404 etc.)
                        reject(new Error(this.statusText));
                    }
                },
                onerror() {
                    reject(new Error(
                        'XMLHttpRequest Error: '+this.statusText));
                }
            });
            request.open('GET', url);
            request.send();
        });
}
```
### 28.4.6 Revocable references
Revocable references work as follows: A client is not allowed to access an important resource (an object) directly, only via a reference (an intermediate object, a wrapper around the resource). Normally, every operation applied to the reference is forwarded to the resource. After the client is done, the resource is protected by revoking the reference, by switching it off. Henceforth, applying operations to the reference throws exceptions and nothing is forwarded, anymore.

In the following example, we create a revocable reference for a resource. We then read one of the resource’s properties via the reference. That works, because the reference grants us access. Next, we revoke the reference. Now the reference doesn’t let us read the property, anymore.

```javascript
const resource = { x: 11, y: 8 };
const {reference, revoke} = createRevocableReference(resource);

// Access granted
console.log(reference.x); // 11

revoke();

// Access denied
console.log(reference.x); // TypeError: Revoked
```
Proxies are ideally suited for implementing revocable references, because they can intercept and forward operations. This is a simple proxy-based implementation of createRevocableReference:

```javascript
function createRevocableReference(target) {
    let enabled = true;
    return {
        reference: new Proxy(target, {
            get(target, propKey, receiver) {
                if (!enabled) {
                    throw new TypeError('Revoked');
                }
                return Reflect.get(target, propKey, receiver);
            },
            has(target, propKey) {
                if (!enabled) {
                    throw new TypeError('Revoked');
                }
                return Reflect.has(target, propKey);
            },
            ···
        }),
        revoke() {
            enabled = false;
        },
    };
}
```
The code can be simplified via the proxy-as-handler technique from the previous section. This time, the handler basically is the Reflect object. Thus, the get trap normally returns the appropriate Reflect method. If the reference has been revoked, a TypeError is thrown, instead.

```javascript
function createRevocableReference(target) {
    let enabled = true;
    const handler = new Proxy({}, {
        get(dummyTarget, trapName, receiver) {
            if (!enabled) {
                throw new TypeError('Revoked');
            }
            return Reflect[trapName];
        }
    });
    return {
        reference: new Proxy(target, handler),
        revoke() {
            enabled = false;
        },
    };
}
```
However, you don’t have to implement revocable references yourself, because ECMAScript 6 lets you create proxies that can be revoked. This time, the revoking happens in the proxy, not in the handler. All the handler has to do is forward every operation to the target. As we have seen that happens automatically if the handler doesn’t implement any traps.

```javascript
function createRevocableReference(target) {
    const handler = {}; // forward everything
    const { proxy, revoke } = Proxy.revocable(target, handler);
    return { reference: proxy, revoke };
}
```
#### 28.4.6.1 Membranes
Membranes build on the idea of revocable references: Environments that are designed to run untrusted code, wrap a membrane around that code to isolate it and keep the rest of the system safe. Objects pass the membrane in two directions:

The code may receive objects (“dry objects”) from the outside.
Or it may hand objects (“wet objects”) to the outside.
In both cases, revocable references are wrapped around the objects. Objects returned by wrapped functions or methods are also wrapped. Additionally, if a wrapped wet object is passed back into a membrane, it is unwrapped.

Once the untrusted code is done, all of the revocable references are revoked. As a result, none of its code on the outside can be executed anymore and outside objects that it has cease to work, as well. The Caja Compiler is “a tool for making third party HTML, CSS and JavaScript safe to embed in your website”. It uses membranes to achieve this task.

### 28.4.7 Implementing the DOM in JavaScript
The browser Document Object Model (DOM) is usually implemented as a mix of JavaScript and C++. Implementing it in pure JavaScript is useful for:

Emulating a browser environment, e.g. to manipulate HTML in Node.js. jsdom is one library that does that.
Speeding the DOM up (switching between JavaScript and C++ costs time).
Alas, the standard DOM can do things that are not easy to replicate in JavaScript. For example, most DOM collections are live views on the current state of the DOM that change dynamically whenever the DOM changes. As a result, pure JavaScript implementations of the DOM are not very efficient. One of the reasons for adding proxies to JavaScript was to help write more efficient DOM implementations.

### 28.4.8 Other use cases
There are more use cases for proxies. For example:

Remoting: Local placeholder objects forward method invocations to remote objects. This use case is similar to the web service example.
Data access objects for databases: Reading and writing to the object reads and writes to the database. This use case is similar to the web service example.
Profiling: Intercept method invocations to track how much time is spent in each method. This use case is similar to the tracing example.
Type checking: Nicholas Zakas has used proxies to type-check objects.
28.5 The design of the proxy API
In this section, we go deeper into how proxies work and why they work that way.

### 28.5.1 Stratification: keeping base level and meta level separate
Firefox has allowed you to do some interceptive metaprogramming for a while: If you define a method whose name is __noSuchMethod__, it is notified whenever a method is called that doesn’t exist. The following is an example of using __noSuchMethod__.

```javascript
const obj = {
    __noSuchMethod__: function (name, args) {
        console.log(name+': '+args);
    }
};
// Neither of the following two methods exist,
// but we can make it look like they do
obj.foo(1);    // Output: foo: 1
obj.bar(1, 2); // Output: bar: 1,2
```
Thus, __noSuchMethod__ works similarly to a proxy trap. In contrast to proxies, the trap is an own or inherited method of the object whose operations we want to intercept. The problem with that approach is that base level (normal methods) and meta level (__noSuchMethod__) are mixed. Base-level code may accidentally invoke or see a meta level method and there is the possibility of accidentally defining a meta level method.

Even in standard ECMAScript 5, base level and meta level are sometimes mixed. For example, the following metaprogramming mechanisms can fail, because they exist at the base level:

obj.hasOwnProperty(propKey): This call can fail if a property in the prototype chain overrides the built-in implementation. For example, it fails if obj is:
  { hasOwnProperty: null }
A safe way to call this method is:
```javascript
  Object.prototype.hasOwnProperty.call(obj, propKey)

  // Abbreviated version:
  {}.hasOwnProperty.call(obj, propKey)
  ```
func.call(···), func.apply(···): For each of these two methods, problem and solution are the same as with hasOwnProperty.
obj.__proto__: In most JavaScript engines, __proto__ is a special property that lets you get and set the prototype of obj. Hence, when you use objects as dictionaries, you must be careful to avoid __proto__ as a property key.
By now, it should be obvious that making (base level) property keys special is problematic. Therefore, proxies are stratified – base level (the proxy object) and meta level (the handler object) are separate.

### 28.5.2 Virtual objects versus wrappers
Proxies are used in two roles:

As wrappers, they wrap their targets, they control access to them. Examples of wrappers are: revocable resources and tracing proxies.
As virtual objects, they are simply objects with special behavior and their targets don’t matter. An example is a proxy that forwards method calls to a remote object.
An earlier design of the proxy API conceived proxies as purely virtual objects. However, it turned out that even in that role, a target was useful, to enforce invariants (which is explained later) and as a fallback for traps that the handler doesn’t implement.

### 28.5.3 Transparent virtualization and handler encapsulation
Proxies are shielded in two ways:

It is impossible to determine whether an object is a proxy or not (transparent virtualization).
You can’t access a handler via its proxy (handler encapsulation).
Both principles give proxies considerable power for impersonating other objects. One reason for enforcing invariants (as explained later) is to keep that power in check.

If you do need a way to tell proxies apart from non-proxies, you have to implement it yourself. The following code is a module lib.js that exports two functions: one of them creates proxies, the other one determines whether an object is one of those proxies.

```javascript
// lib.js
const proxies = new WeakSet();

export function createProxy(obj) {
    const handler = {};
    const proxy = new Proxy(obj, handler);
    proxies.add(proxy);
    return proxy;
}

export function isProxy(obj) {
    return proxies.has(obj);
}
```
This module uses the ECMAScript 6 data structure WeakSet for keeping track of proxies. WeakSet is ideally suited for this purpose, because it doesn’t prevent its elements from being garbage-collected.

The next example shows how lib.js can be used.

```javascript
// main.js
import { createProxy, isProxy } from './lib.js';

const p = createProxy({});
console.log(isProxy(p)); // true
console.log(isProxy({})); // false
```
### 28.5.4 The meta object protocol and proxy traps
This section examines how JavaScript is structured internally and how the set of proxy traps was chosen.

In the context of programming languages and API design, a protocol is a set of interfaces plus rules for using them. The ECMAScript specification describes how to execute JavaScript code. It includes a protocol for handling objects. This protocol operates at a meta level and is sometimes called the meta object protocol (MOP). The JavaScript MOP consists of own internal methods that all objects have. “Internal” means that they exist only in the specification (JavaScript engines may or may not have them) and are not accessible from JavaScript. The names of internal methods are written in double square brackets.

The internal method for getting properties is called [[Get]]. If we pretend that property names with square brackets are legal, this method would roughly be implemented as follows in JavaScript.

```javascript
// Method definition
[[Get]](propKey, receiver) {
    const desc = this.[[GetOwnProperty]](propKey);
    if (desc === undefined) {
        const parent = this.[[GetPrototypeOf]]();
        if (parent === null) return undefined;
        return parent.[[Get]](propKey, receiver); // (A)
    }
    if ('value' in desc) {
        return desc.value;
    }
    const getter = desc.get;
    if (getter === undefined) return undefined;
    return getter.[[Call]](receiver, []);
}
```
The MOP methods called in this code are:

```javascript
[[GetOwnProperty]] (trap getOwnPropertyDescriptor)
[[GetPrototypeOf]] (trap getPrototypeOf)
[[Get]] (trap get)
[[Call]] (trap apply)
```
In line (A) you can see why proxies in a prototype chain find out about get if a property isn’t found in an “earlier” object: If there is no own property whose key is propKey, the search continues in the prototype parent of this.

Fundamental versus derived operations. You can see that [[Get]] calls other MOP operations. Operations that do that are called derived. Operations that don’t depend on other operations are called fundamental.

#### 28.5.4.1 The MOP of proxies
The meta object protocol of proxies is different from that of normal objects. For normal objects, derived operations call other operations. For proxies, each operation (regardless of whether it is fundamental or derived) is either intercepted by a handler method or forwarded to the target.

What operations should be interceptable via proxies? One possibility is to only provide traps for fundamental operations. The alternative is to include some derived operations. The advantage of doing so is that it increases performance and is more convenient. For example, if there weren’t a trap for get, you’d have to implement its functionality via getOwnPropertyDescriptor. One problem with derived traps is that they can lead to proxies behaving inconsistently. For example, get may return a value that is different from the value in the descriptor returned by getOwnPropertyDescriptor.

#### 28.5.4.2 Selective intercession: what operations should be interceptable?
Intercession by proxies is selective: you can’t intercept every language operation. Why were some operations excluded? Let’s look at two reasons.

First, stable operations are not well suited for intercession. An operation is stable if it always produces the same results for the same arguments. If a proxy can trap a stable operation, it can become unstable and thus unreliable. Strict equality (===) is one such stable operation. It can’t be trapped and its result is computed by treating the proxy itself as just another object. Another way of maintaining stability is by applying an operation to the target instead of the proxy. As explained later, when we look at how invariants are enfored for proxies, this happens when Object.getPrototypeOf() is applied to a proxy whose target is non-extensible.

A second reason for not making more operations interceptable is that intercession means executing custom code in situations where that normally isn’t possible. The more this interleaving of code happens, the harder it is to understand and debug a program. It also affects performance negatively.

#### 28.5.4.3 Traps: get versus invoke
If you want to create virtual methods via ECMAScript 6 proxies, you have to return functions from a get trap. That raises the question: why not introduce an extra trap for method invocations (e.g. invoke)? That would enable us to distinguish between:

Getting properties via obj.prop (trap get)
Invoking methods via obj.prop() (trap invoke)
There are two reasons for not doing so.

First, not all implementations distinguish between get and invoke. For example, Apple’s JavaScriptCore doesn’t.

Second, extracting a method and invoking it later via call() or apply() should have the same effect as invoking the method via dispatch. In other words, the following two variants should work equivalently. If there was an extra trap invoke then that equivalence would be harder to maintain.

```javascript
// Variant 1: call via dynamic dispatch
const result = obj.m();

// Variant 2: extract and call directly
const m = obj.m;
const result = m.call(obj);
```
##### 28.5.4.3.1 Use cases for invoke
Some things can only be done if you are able to distinguish between get and invoke. Those things are therefore impossible with the current proxy API. Two examples are: auto-binding and intercepting missing methods. Let’s examine how one would implement them if proxies supported invoke.

Auto-binding. By making a proxy the prototype of an object obj, you can automatically bind methods:

Retrieving the value of a method m via obj.m returns a function whose this is bound to obj.
obj.m() performs a method call.
Auto-binding helps with using methods as callbacks. For example, variant 2 from the previous example becomes simpler:

```javascript
const boundMethod = obj.m;
const result = boundMethod();
```
Intercepting missing methods. invoke lets a proxy emulate the previously mentioned __noSuchMethod__ mechanism that Firefox supports. The proxy would again become the prototype of an object obj. It would react differently depending on how an unknown property foo is accessed:

If you read that property via obj.foo, no intercession happens and undefined is returned.
If you make the method call obj.foo() then the proxy intercepts and, e.g., notifies a callback.
28.5.5 Enforcing invariants for proxies
Before we look at what invariants are and how they are enforced for proxies, let’s review how objects can be protected via non-extensibility and non-configurability.

#### 28.5.5.1 Protecting objects
There are two ways of protecting objects:

Non-extensibility protects objects
Non-configurability protects properties (or rather, their attributes)
Non-extensibility. If an object is non-extensible, you can’t add properties and you can’t change its prototype:

```javascript
'use strict'; // switch on strict mode to get TypeErrors

const obj = Object.preventExtensions({});
console.log(Object.isExtensible(obj)); // false
obj.foo = 123; // TypeError: object is not extensible
Object.setPrototypeOf(obj, null); // TypeError: object is not extensible
````
Non-configurability. All the data of a property is stored in attributes. A property is like a record and attributes are like the fields of that record. Examples of attributes:

The attribute value holds the value of a property.
The boolean attribute writable controls whether a property’s value can be changed.
The boolean attribute configurable controls whether a property’s attributes can be changed.
Thus, if a property is both non-writable and non-configurable, it is read-only and remains that way:

```javascript
'use strict'; // switch on strict mode to get TypeErrors

const obj = {};
Object.defineProperty(obj, 'foo', {
    value: 123,
    writable: false,
    configurable: false
});
console.log(obj.foo); // 123
obj.foo = 'a'; // TypeError: Cannot assign to read only property

Object.defineProperty(obj, 'foo', {
    configurable: true
}); // TypeError: Cannot redefine property
```
For more details on these topics (including how Object.defineProperty() works) consult the following sections in “Speaking JavaScript”:

Property Attributes and Property Descriptors
Protecting Objects
#### 28.5.5.2 Enforcing invariants
Traditionally, non-extensibility and non-configurability are:

Universal: they work for all objects.
Monotonic: once switched on, they can’t be switched off again.
These and other characteristics that remain unchanged in the face of language operations are called invariants. With proxies, it is easy to violate invariants, as they are not intrinsically bound by non-extensibility etc.

The proxy API prevents proxies from violating invariants by checking the parameters and results of handler methods. The following are four examples of invariants (for an arbitrary object obj) and how they are enforced for proxies (an exhaustive list is given at the end of this chapter).

The first two invariants involve non-extensibility and non-configurability. These are enforced by using the target object for bookkeeping: results returned by handler methods have to be mostly in sync with the target object.

Invariant: If Object.preventExtensions(obj) returns true then all future calls must return false and obj must now be non-extensible.
Enforced for proxies by throwing a TypeError if the handler returns true, but the target object is not extensible.
Invariant: Once an object has been made non-extensible, Object.isExtensible(obj) must always return false.
Enforced for proxies by throwing a TypeError if the result returned by the handler is not the same (after coercion) as Object.isExtensible(target).
The remaining two invariants are enforced by checking return values:

Invariant: Object.isExtensible(obj) must return a boolean.
Enforced for proxies by coercing the value returned by the handler to a boolean.
Invariant: Object.getOwnPropertyDescriptor(obj, ···) must return an object or undefined.
Enforced for proxies by throwing a TypeError if the handler doesn’t return an appropriate value.
Enforcing invariants has the following benefits:

Proxies work like all other objects with regard to extensibility and configurability. Therefore, universality is maintained. This is achieved without preventing proxies from virtualizing (impersonating) protected objects.
A protected object can’t be misrepresented by wrapping a proxy around it. Misrepresentation can be caused by bugs or by malicious code.
The next two sections give examples of invariants being enforced.

#### 28.5.5.3 Example: the prototype of a non-extensible target must be represented faithfully
In response to the getPrototypeOf trap, the proxy must return the target’s prototype if the target is non-extensible.

To demonstrate this invariant, let’s create a handler that returns a prototype that is different from the target’s prototype:

```javascript
const fakeProto = {};
const handler = {
    getPrototypeOf(t) {
        return fakeProto;
    }
};
```
Faking the prototype works if the target is extensible:

```javascript
const extensibleTarget = {};
const ext = new Proxy(extensibleTarget, handler);
console.log(Object.getPrototypeOf(ext) === fakeProto); // true
```
We do, however, get an error if we fake the prototype for a non-extensible object.

```javascript
const nonExtensibleTarget = {};
Object.preventExtensions(nonExtensibleTarget);
const nonExt = new Proxy(nonExtensibleTarget, handler);
Object.getPrototypeOf(nonExt); // TypeError
```
#### 28.5.5.4 Example: non-writable non-configurable target properties must be represented faithfully
If the target has a non-writable non-configurable property then the handler must return that property’s value in response to a get trap. To demonstrate this invariant, let’s create a handler that always returns the same value for properties.

```javascript
const handler = {
    get(target, propKey) {
        return 'abc';
    }
};
const target = Object.defineProperties(
    {}, {
        foo: {
            value: 123,
            writable: true,
            configurable: true
        },
        bar: {
            value: 456,
            writable: false,
            configurable: false
        },
    });
const proxy = new Proxy(target, handler);
```
Property target.foo is not both non-writable and non-configurable, which means that the handler is allowed to pretend that it has a different value:

```javascript
> proxy.foo
'abc'
```
However, property target.bar is both non-writable and non-configurable. Therefore, we can’t fake its value:

```javascript
> proxy.bar
TypeError: Invariant check failed
```

## 28.6 FAQ: proxies
### 28.6.1 Where is the enumerate trap?
ES6 originally had a trap enumerate that was triggered by for-in loops. But it was recently removed, to simplify proxies. Reflect.enumerate() was removed, as well. (Source: TC39 notes)

## 28.7 Reference: the proxy API
This section serves as a quick reference for the proxy API: the global objects Proxy and Reflect.

### 28.7.1 Creating proxies
There are two ways to create proxies:

const proxy = new Proxy(target, handler)
Creates a new proxy object with the given target and the given handler.
const {proxy, revoke} = Proxy.revocable(target, handler)
Creates a proxy that can be revoked via the function revoke. revoke can be called multiple times, but only the first call has an effect and switches proxy off. Afterwards, any operation performed on proxy leads to a TypeError being thrown.
28.7.2 Handler methods
This subsection explains what traps can be implemented by handlers and what operations trigger them. Several traps return boolean values. For the traps has and isExtensible, the boolean is the result of the operation. For all other traps, the boolean indicates whether the operation succeeded or not.

Traps for all objects:

defineProperty(target, propKey, propDesc) : boolean
Object.defineProperty(proxy, propKey, propDesc)
deleteProperty(target, propKey) : boolean
delete proxy[propKey]
delete proxy.foo // propKey = 'foo'
get(target, propKey, receiver) : any
receiver[propKey]
receiver.foo // propKey = 'foo'
getOwnPropertyDescriptor(target, propKey) : PropDesc|Undefined
Object.getOwnPropertyDescriptor(proxy, propKey)
getPrototypeOf(target) : Object|Null
Object.getPrototypeOf(proxy)
has(target, propKey) : boolean
propKey in proxy
isExtensible(target) : boolean
Object.isExtensible(proxy)
ownKeys(target) : Array<PropertyKey>
Object.getOwnPropertyPropertyNames(proxy) (only uses string keys)
Object.getOwnPropertyPropertySymbols(proxy) (only uses symbol keys)
Object.keys(proxy) (only uses enumerable string keys; enumerability is checked via Object.getOwnPropertyDescriptor)
preventExtensions(target) : boolean
Object.preventExtensions(proxy)
set(target, propKey, value, receiver) : boolean
receiver[propKey] = value
receiver.foo = value // propKey = 'foo'
setPrototypeOf(target, proto) : boolean
Object.setPrototypeOf(proxy, proto)
Traps for functions (available if target is a function):

apply(target, thisArgument, argumentsList) : any
proxy.apply(thisArgument, argumentsList)
proxy.call(thisArgument, ...argumentsList)
proxy(...argumentsList)
construct(target, argumentsList, newTarget) : Object
new proxy(..argumentsList)
28.7.2.1 Fundamental operations versus derived operations
The following operations are fundamental, they don’t use other operations to do their work: apply, defineProperty, deleteProperty, getOwnPropertyDescriptor, getPrototypeOf, isExtensible, ownKeys, preventExtensions, setPrototypeOf

All other operations are derived, they can be implemented via fundamental operations. For example, for data properties, get can be implemented by iterating over the prototype chain via getPrototypeOf and calling getOwnPropertyDescriptor for each chain member until either an own property is found or the chain ends.

### 28.7.3 Invariants of handler methods
Invariants are safety constraints for handlers. This subsection documents what invariants are enforced by the proxy API and how. Whenever you read “the handler must do X” below, it means that a TypeError is thrown if it doesn’t. Some invariants restrict return values, others restrict parameters. The correctness of a trap’s return value is ensured in two ways: Normally, an illegal value means that a TypeError is thrown. But whenever a boolean is expected, coercion is used to convert non-booleans to legal values.

This is the complete list of invariants that are enforced:

```javascript
apply(target, thisArgument, argumentsList)
```
No invariants are enforced.
construct(target, argumentsList, newTarget)
The result returned by the handler must be an object (not null or a primitive value).
defineProperty(target, propKey, propDesc)
If the target is not extensible then you can’t add properties and propKey must be one of the own keys of the target.
If propDesc sets the attribute configurable to false then the target must have a non-configurable own property whose key is propKey.
If propDesc were to be used to (re)define an own property for the target then that must not cause an exception. An exception is thrown if a change is forbidden by the attributes writable and configurable (non-extensibility is handled by the first rule).
deleteProperty(target, propKey)
Non-configurable own properties of the target can’t be deleted.
get(target, propKey, receiver)
If the target has an own, non-writable, non-configurable data property whose key is propKey then the handler must return that property’s value.
If the target has an own, non-configurable, getter-less accessor property then the handler must return undefined.
getOwnPropertyDescriptor(target, propKey)
The handler must return either an object or undefined.
Non-configurable own properties of the target can’t be reported as non-existent by the handler.
If the target is non-extensible then exactly the target’s own properties must be reported by the handler as existing.
If the handler reports a property as non-configurable then that property must be a non-configurable own property of the target.
If the result returned by the handler were used to (re)define an own property for the target then that must not cause an exception. An exception is thrown if the change is not allowed by the attributes writable and configurable (non-extensibility is handled by the third rule). Therefore, the handler can’t report a non-configurable property as configurable and it can’t report a different value for a non-configurable non-writable property.
getPrototypeOf(target)
The result must be either an object or null.
If the target object is not extensible then the handler must return the prototype of the target object.
has(target, propKey)
A handler must not hide (report as non-existent) a non-configurable own property of the target.
If the target is non-extensible then no own property of the target may be hidden.
isExtensible(target)
The result returned by the handler is coerced to boolean.
After coercion to boolean, the value returned by the handler must be the same as target.isExtensible().
ownKeys(target)
The handler must return an object, which treated as Array-like and converted into an Array.
Each element of the result must be either a string or a symbol.
The result must contain the keys of all non-configurable own properties of the target.
If the target is not extensible then the result must contain exactly the keys of the own properties of the target (and no other values).
preventExtensions(target)
The result returned by the handler is coerced to boolean.
If the handler returns a truthy value (indicating a successful change) then target.isExtensible() must be false afterwards.
set(target, propKey, value, receiver)
If the target has an own, non-writable, non-configurable data property whose key is propKey then value must be the same as the value of that property (i.e., the property can’t be changed).
If the target has an own, non-configurable, setter-less accessor property then a TypeError is thrown (i.e., such a property can’t be set).
setPrototypeOf(target, proto)
The result returned by the handler is coerced to boolean.
If the target is not extensible, the prototype can’t be changed. This is enforced as follows: If the target is not extensible and the handler returns a truthy value (indicating a successful change) then proto must be the same as the prototype of the target. Otherwise, a TypeError is thrown.
In the spec, the invariants are listed in the section “Proxy Object Internal Methods and Internal Slots”.

### 28.7.4 Operations that affect the prototype chain
The following operations of normal objects perform operations on objects in the prototype chain. Therefore, if one of the objects in that chain is a proxy, its traps are triggered. The specification implements the operations as internal own methods (that are not visible to JavaScript code). But in this section, we pretend that they are normal methods that have the same names as the traps. The parameter target becomes the receiver of the method call.

```javascript
target.get(propertyKey, receiver)
```
If target has no own property with the given key, get is invoked on the prototype of target.
target.has(propertyKey)
Similarly to get, has is invoked on the prototype of target if target has no own property with the given key.
target.set(propertyKey, value, receiver)
Similarly to get, set is invoked on the prototype of target if target has no own property with the given key.
All other operations only affect own properties, they have no effect on the prototype chain.

In the spec, these (and other) operations are described in the section “Ordinary Object Internal Methods and Internal Slots”.

### 28.7.5 Reflect
The global object Reflect implements all interceptable operations of the JavaScript meta object protocol as methods. The names of those methods are the same as those of the handler methods, which, as we have seen, helps with forwarding operations from the handler to the target.

Reflect.apply(target, thisArgument, argumentsList) : any
Same as Function.prototype.apply().
Reflect.construct(target, argumentsList, newTarget=target) : Object
The new operator as a function. target is the constructor to invoke, the optional parameter newTarget points to the constructor that started the current chain of constructor calls. More information on how constructor calls are chained in ES6 is given in the chapter on classes.
Reflect.defineProperty(target, propertyKey, propDesc) : boolean
Similar to Object.defineProperty().
Reflect.deleteProperty(target, propertyKey) : boolean
The delete operator as a function. It works slightly differently, though: It returns true if it successfully deleted the property or if the property never existed. It returns false if the property could not be deleted and still exists. The only way to protect properties from deletion is by making them non-configurable. In sloppy mode, the delete operator returns the same results. But in strict mode, it throws a TypeError instead of returning false.
Reflect.get(target, propertyKey, receiver=target) : any
A function that gets properties. The optional parameter receiver is needed when get reaches a getter later in the prototype chain. Then it provides the value for this.
Reflect.getOwnPropertyDescriptor(target, propertyKey) : PropDesc|Undefined
Same as Object.getOwnPropertyDescriptor().
Reflect.getPrototypeOf(target) : Object|Null
Same as Object.getPrototypeOf().
Reflect.has(target, propertyKey) : boolean
The in operator as a function.
Reflect.isExtensible(target) : boolean
Same as Object.isExtensible().
Reflect.ownKeys(target) : Array<PropertyKey>
Returns all own property keys (strings and symbols!) in an Array.
Reflect.preventExtensions(target) : boolean
Similar to Object.preventExtensions().
Reflect.set(target, propertyKey, value, receiver=target) : boolean
A function that sets properties.
Reflect.setPrototypeOf(target, proto) : boolean
The new standard way of setting the prototype of an object. The current non-standard way, that works in most engines, is to set the special property __proto__.
Several methods have boolean results. For has and isExtensible, they are the results of the operation. For the remaining methods, they indicate whether the operation succeeded.

#### 28.7.5.1 Use cases for Reflect besides forwarding
Apart from forwarding operations, why is Reflect useful [4]?

Different return values: Reflect duplicates the following methods of Object, but its methods return booleans indicating whether the operation succeeded (where the Object methods return the object that was modified).
Object.defineProperty(obj, propKey, propDesc) : Object
Object.preventExtensions(obj) : Object
Object.setPrototypeOf(obj, proto) : Object
Operators as functions: The following Reflect methods implement functionality that is otherwise only available via operators:
Reflect.construct(target, argumentsList, newTarget=target) : Object
Reflect.deleteProperty(target, propertyKey) : boolean
Reflect.get(target, propertyKey, receiver=target) : any
Reflect.has(target, propertyKey) : boolean
Reflect.set(target, propertyKey, value, receiver=target) : boolean
Shorter version of apply(): If you want to be completely safe about invoking the method apply() on a function, you can’t do so via dynamic dispatch, because the function may have an own property with the key 'apply':
  ```javascript
  func.apply(thisArg, argArray) // not safe
  Function.prototype.apply.call(func, thisArg, argArray) // safe
  ```
Using Reflect.apply() is shorter:

  Reflect.apply(func, thisArg, argArray)
No exceptions when deleting properties: the delete operator throws in strict mode if you try to delete a non-configurable own property. Reflect.deleteProperty() returns false in that case.
#### 28.7.5.2 Object.* versus Reflect.*
Going forward, Object will host operations that are of interest to normal applications, while Reflect will host operations that are more low-level.

## 28.8 Conclusion
This concludes our in-depth look at the proxy API. For each application, you have to take performance into consideration and – if necessary – measure. Proxies may not always be fast enough. On the other hand, performance is often not crucial and it is nice to have the metaprogramming power that proxies give us. As we have seen, there are numerous use cases they can help with.

## 28.9 Further reading
[1] “On the design of the ECMAScript Reflection API” by Tom Van Cutsem and Mark Miller. Technical report, 2012. [Important source of this chapter.]

[2] “The Art of the Metaobject Protocol” by Gregor Kiczales, Jim des Rivieres and Daniel G. Bobrow. Book, 1991.

[3] “Putting Metaclasses to Work: A New Dimension in Object-Oriented Programming” by Ira R. Forman and Scott H. Danforth. Book, 1999.

[4] “Harmony-reflect: Why should I use this library?” by Tom Van Cutsem. [Explains why Reflect is useful.]
