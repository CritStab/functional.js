# functional.js

Functional Programming for JavaScript

```javascript
// Lazy Evaluation
var th = new Thunk(() => {
    console.log("hello world");
    return 2 + 2;
});
th.value(); // => hello world, 4
th.value(); // => 4

// Sum Types
var Maybe = new Variant({
    Just: (value) => ({value: value}),
    Nothing: () => ({})
});
Maybe.pure = Maybe.Just;
Maybe.prototype.map = function(f) {
    return this.match({
        Just: (x) => Maybe.Just(f(x)),
        _: () => Maybe.Nothing()
    });
};
Maybe.pure(5).map(x => x + 1).value; // => 6

// Tail Recursion
function factorial(n) {
    var f = new TailRecursive(function(accu, n) {
        if(n == 0) return accu;
        else return this.tailcall(accu * n, n - 1);
    });
    return f.run(1, n);
}
factorial(5); // => 120

// Simple Product Types
var tup = new Tuple(1,2,3);
tup.unpack((x,y,z) => x+z); // => 4
tup.length; // => 3

// Pure Input/Output
var ioPrompt = m => new IO(() => prompt(m));
var ioAlert = m => new IO(() => alert(m));
var main =
    ioPrompt("enter your name").map(name => "hello " + name).bind(ioAlert);
IO.pure("hello world").bind(ioAlert).execute(); // => hello world
```
