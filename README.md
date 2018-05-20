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
                                                                                                                                                                                                                         
// Tail Recursion                                                                                                                                                                                                        
function factorial(n) {                                                                                                                                                                                                  
    var f = new TailRecursive(function(accu, n) {                                                                                                                                                                        
        if(n == 0) return accu;                                                                                                                                                                                          
        else return this.tailcall(accu * n, n - 1);                                                                                                                                                                      
    });                                                                                                                                                                                                                  
    return f.run(1, n);                                                                                                                                                                                                  
}                                                                                                                                                                                                                        
                                                                                                                                                                                                                         
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

## Source Code

```javascript
class Thunk {                                                                                                                                                                                                            
    constructor(expr) { this.expr = expr; }                                                                                                                                                                              
    value() { return (this.expr = (x => () => x)(this.expr()))(); }                                                                                                                                                      
}                                                                                                                                                                                                                        
                                                                                                                                                                                                                         
class Variant {                                                                                                                                                                                                          
    constructor(constructors) {                                                                                                                                                                                          
        this.prototype = {};                                                                                                                                                                                             
        for(let cons in constructors) this[cons] = function() {                                                                                                                                                          
            return Object.assign(Object.create(                                                                                                                                                                          
                Object.assign({match: (cases) => cases[(cons in cases) ? cons : "_"].apply(null, arguments)},                                                                                                            
                              this.prototype)), constructors[cons].apply(null, arguments));                                                                                                                              
        };                                                                                                                                                                                                               
    }                                                                                                                                                                                                                    
}                                                                                                                                                                                                                        
                                                                                                                                                                                                                         
class TailRecursive {                                                                                                                                                                                                    
    constructor(f) { this.f = f; }                                                                                                                                                                                       
    tailcall() { return Object.assign(this, {args: arguments}); }                                                                                                                                                        
    run() {                                                                                                                                                                                                              
        for(this.args = arguments, this.res = this; this.res == this; this.res = this.f.apply(this, this.args));                                                                                                         
        return this.res;                                                                                                                                                                                                 
    }                                                                                                                                                                                                                    
}                                                                                                                                                                                                                        
                                                                                                                                                                                                                         
class Tuple {                                                                                                                                                                                                            
    constructor() { for(var i = 0; i < (this.length = arguments.length); i++) this[i] = arguments[i]; }                                                                                                                  
    unpack(f) { return f.apply(null, this); }                                                                                                                                                                            
}                                                                                                                                                                                                                        
                                                                                                                                                                                                                         
class IO {                                                                                                                                                                                                               
    constructor(execute) { this.execute = execute; }                                                                                                                                                                     
    static pure(value) { return new IO(() => value); }                                                                                                                                                                   
    bind(f) { return new IO(() => f(this.execute()).execute()); }                                                                                                                                                        
    map(f) { return this.bind(x => IO.pure(f(x))); }                                                                                                                                                                     
}
```
