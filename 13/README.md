13. Arrow functions

13.1 Overview
There are two benefits to arrow functions.

First, they are less verbose than traditional function expressions:

const arr = [1, 2, 3];
const squares = arr.map(x => x * x);

// Traditional function expression:
const squares = arr.map(function (x) { return x * x });
Second, their this is picked up from surroundings (lexical). Therefore, you don’t need bind() or that = this, anymore.

function UiComponent() {
    const button = document.getElementById('myButton');
    button.addEventListener('click', () => {
        console.log('CLICK');
        this.handleClick(); // lexical `this`
    });
}
The following variables are all lexical inside arrow functions:

arguments
super
this
new.target
13.2 Traditional functions are bad non-method functions, due to this
In JavaScript, traditional functions can be used as:

Non-method functions
Methods
Constructors
These roles clash: Due to roles 2 and 3, functions always have their own this. But that prevents you from accessing the this of, e.g., a surrounding method from inside a callback (role 1).

You can see that in the following ES5 code:

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) { // (A)
    'use strict';
    return arr.map(function (x) { // (B)
        // Doesn’t work:
        return this.prefix + x; // (C)
    });
};
In line C, we’d like to access this.prefix, but can’t do that because the this of the function from line B shadows the this of the method from line A. In strict mode, this is undefined in non-method functions, which is why we get an error if we use Prefixer:

> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
TypeError: Cannot read property 'prefix' of undefined
There are three ways to work around this problem in ECMAScript 5.

13.2.1 Solution 1: that = this
You can assign this to a variable that isn’t shadowed. That’s what’s done in line A, below:

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    var that = this; // (A)
    return arr.map(function (x) {
        return that.prefix + x;
    });
};
Now Prefixer works as expected:

> var pre = new Prefixer('Hi ');
> pre.prefixArray(['Joe', 'Alex'])
[ 'Hi Joe', 'Hi Alex' ]
13.2.2 Solution 2: specifying a value for this
A few Array methods have an extra parameter for specifying the value that this should have when invoking the callback. That’s the last parameter in line A, below.

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }, this); // (A)
};
13.2.3 Solution 3: bind(this)
You can use the method bind() to convert a function whose this is determined by how it is called (via call(), a function call, a method call, etc.) to a function whose this is always the same fixed value. That’s what we are doing in line A, below.

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map(function (x) {
        return this.prefix + x;
    }.bind(this)); // (A)
};
13.2.4 ECMAScript 6 solution: arrow functions
Arrow functions are basically solution 3, with a more convenient syntax. With an arrow function, the code looks as follows.

function Prefixer(prefix) {
    this.prefix = prefix;
}
Prefixer.prototype.prefixArray = function (arr) {
    return arr.map((x) => {
        return this.prefix + x;
    });
};
To fully ES6-ify the code, you’d use a class and a more compact variant of arrow functions:

class Prefixer {
    constructor(prefix) {
        this.prefix = prefix;
    }
    prefixArray(arr) {
        return arr.map(x => this.prefix + x); // (A)
    }
}
In line A we save a few characters by tweaking two parts of the arrow function:

If there is only one parameter and that parameter is an identifier then the parentheses can be omitted.
An expression following the arrow leads to that expression being returned.
In the code, you can also see that the methods constructor and prefixArray are defined using new, more compact ES6 syntax that also works in object literals.

13.3 Arrow function syntax
The “fat” arrow => (as opposed to the thin arrow ->) was chosen to be compatible with CoffeeScript, whose fat arrow functions are very similar.

Specifying parameters:

    () => { ... } // no parameter
     x => { ... } // one parameter, an identifier
(x, y) => { ... } // several parameters
Specifying a body:

x => { return x * x }  // block
x => x * x  // expression, equivalent to previous line
The statement block behaves like a normal function body. For example, you need return to give back a value. With an expression body, the expression is always implicitly returned.

Note how much an arrow function with an expression body can reduce verbosity. Compare:

const squares = [1, 2, 3].map(function (x) { return x * x });
const squares = [1, 2, 3].map(x => x * x);
13.3.1 Omitting parentheses around single parameters
Omitting the parentheses around the parameters is only possible if they consist of a single identifier:

> [1,2,3].map(x => 2 * x)
[ 2, 4, 6 ]
As soon as there is anything else, you have to type the parentheses, even if there is only a single parameter. For example, you need parens if you destructure a single parameter:

> [[1,2], [3,4]].map(([a,b]) => a + b)
[ 3, 7 ]
And you need parens if a single parameter has a default value (undefined triggers the default value!):

> [1, undefined, 3].map((x='yes') => x)
[ 1, 'yes', 3 ]
13.4 Lexical variables
13.4.1 Sources of variable values: static versus dynamic
The following are two ways in which a variable can receive its value.

First, statically (lexically): Its value is determined by the structure of the program; it receives its value from a surrounding scope. For example:

const x = 123;

function foo(y) {
    return x; // value received statically
}
Second, dynamically: It receives its value via a function call. For example:

function bar(arg) {
    return arg; // value received dynamically
}
13.4.2 Variables that are lexical in arrow functions
The source of this is an important distinguishing aspect of arrow functions:

Traditional functions have a dynamic this; its value is determined by how they are called.
Arrow functions have a lexical this; its value is determined by the surrounding scope.
The complete list of variables whose values are determined lexically is:

arguments
super
this
new.target
13.5 Syntax pitfalls
There are a few syntax-related details that can sometimes trip you up.

13.5.1 Arrow functions bind very loosely
Syntactically, arrow functions bind very loosely. The reason is that you want every expression that can appear in an expression body to “stick together”; it should bind more tightly than the arrow function:

const f = x => (x % 2) === 0 ? x : 0;
As a consequence, you often have to wrap arrow functions in parentheses if they appear somewhere else. For example:

console.log(typeof () => {}); // SyntaxError
console.log(typeof (() => {})); // OK
On the flip side, you can use typeof as an expression body without putting it in parens:

const f = x => typeof x;
13.5.2 No line break after arrow function parameters
ES6 forbids a line breaks between the parameter definitions and the body of an arrow function:

const func1 = a // SyntaxError
=> a*2;
const func2 = a => a*2; // OK

const func3 = (x, y) // SyntaxError
=> {
    return x + y;
};
const func4 = (x, y) => { // OK
    return x + y;
};
const func5 = (x, // OK
y) => {
    return x + y;
};
The rationale for this restriction is that it keeps the options open w.r.t. to “headless” arrow functions in the future: if there are zero parameters, you’d be able to omit the parentheses.

13.5.3 You can’t use statements as expression bodies
13.5.3.1 Expressions versus statements
Quick review (consult “Speaking JavaScript” for more information):

Expressions produce (are evaluated to) values. Examples:

3 + 4
foo(7)
'abc'.length
Statements do things. Examples:

while (true) { ··· }
return 123;
Most expressions1 can be used as statements, simply by mentioning them in statement positions:

function bar() {
    3 + 4;
    foo(7);
    'abc'.length;
}
13.5.3.2 The bodies of arrow functions
If an expression is the body of an arrow function, you don’t need braces:

asyncFunc.then(x => console.log(x));
However, statements have to be put in braces:

asyncFunc.catch(x => { throw x });
13.5.4 Returning object literals
Having a block body in addition to an expression body means that if you want the expression body to be an object literal, you have to put it in parentheses.

The body of this arrow function is a block with the label bar and the expression statement 123.

const f = x => { bar: 123 }
The body of this arrow function is an expression, an object literal:

const f = x => ({ bar: 123 })
13.6 Immediately-invoked arrow functions
Remember Immediately Invoked Function Expressions (IIFEs)? They look as follows and are used to simulate block-scoping and value-returning blocks in ECMAScript 5:

(function () { // open IIFE
    // inside IIFE
}()); // close IIFE
You can save a few characters if you use an Immediately Invoked Arrow Function (IIAF):

(() => {
    return 123
})();
Similarly to IIFEs, you should terminate IIAFs with semicolons (or use an equivalent measure), to avoid two consecutive IIAFs being interpreted as a function call (the first one as the function, the second one as the parameter).

Even if the IIAF has a block body, you must wrap it in parentheses, because it can’t be (directly) function-called, due to how loosely it binds. Note that the parentheses must be around the arrow function. With IIFEs you have a choice: you can either put the parentheses around the whole statement or just around the function expression.

As mentioned in the previous section, arrow functions binding loosely is useful for expression bodies, where you want this expression:

const value = () => foo()
to be interpreted as:

const value = () => (foo())
and not as:

const value = (() => foo)()
A section in the chapter on callable entities has more information on using IIFEs and IIAFs in ES6.

13.7 Arrow functions versus normal functions
An arrow function is different from a normal function in only two ways:

The following constructs are lexical: arguments, super, this, new.target
It can’t be used as a constructor: Normal functions support new via the internal method [[Construct]] and the property prototype. Arrow functions have neither, which is why new (() => {}) throws an error.
Apart from that, there are no observable differences between an arrow function and a normal function. For example, typeof and instanceof produce the same results:

> typeof (() => {})
'function'
> () => {} instanceof Function
true

> typeof function () {}
'function'
> function () {} instanceof Function
true
Consult the chapter on callable entities for more information on when to use arrow functions and when to use traditional functions.

The exceptions are function expressions and object literals, which you have to put in parentheses, because they look like function declarations and code blocks.↩
