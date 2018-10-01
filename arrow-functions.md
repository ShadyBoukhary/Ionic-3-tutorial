# Arrow Functions vs Function Expressions

## What are arrow functions

Arrow functions were introduced in ES6. They provide a more concise syntax than function expressions.

Arrow functions are anonymous. This means `this` does not behave the same way as it would in a regular function.

### Few Examples of arrow functions vs expressions

```javascript

// Print a string

// Arrow function
const print = str => console.log(str)
print('Hello!');

// Function expression
function print(str) {
    console.log(str);
}
print('Hello!');

```
When an arrow function takes only one argument, no parenthesis are required around it.

Also, when it has 1 statement, no brackets are required and the return statement is implicit.

```javascript
// Multiply 2 numbers

// Arrow Function
const multiply = (a, b) => a * b
multiply(2, 3);

// Function expression
function(a, b) {
    return a * b;
}

// More than one statement
const doSomething = (a, b) => {
    // some logic here
}

// No arguments: needs empty parenthesis
const sayHi = () => console.log('Hi');

```

### Behavior Differences 

Arrow functions are more than just syntactical sugar. Arrow functions behave differently than function expressions when it comes to the binding of `this`.

#### Function Expressions

Function expressions create their own instance of `this`. That means that `this` inside a function expression refers to the function itself - not the environment that called it. 

##### Examples

```javascript

var languages = {
  names: ['C#', 'C++', 'JavaScript'],
  person: 'John Doe',
  description: function(){
    return this.names.map(function(language){
      return `${this.person} knows a language called ${language}.`
    });
  }
};
languages.description()

// Output example

// undefined knows a language called C#. undefined knows a language called C++. undefined knows a language called JavaScript.
```

Oops. What happened? `this` refers to the function, rather than the object that called the function.

In ES5, there were numerous ways around this, such as 

- Binding `this`
  
    ```javascript
    var languages = {
        names: ['C#', 'C++', 'JavaScript'],
        person: 'John Doe',
        description: function(){
            return this.names.map(function(language){
                return `${this.person} knows a language called $  {language}.`
            }.bind(this));
        }
    };
    languages.description()

    ```
- Passing `this` as an argument
    ```javascript
    var languages = {
        names: ['C#', 'C++', 'JavaScript'],
        person: 'John Doe',
        description: function(){
            return this.names.map(function(language){
                return `${this.person} knows a language called $  {language}.`
            }, languages);
        }
    };
    languages.description()

    ```
- Set the value of `this` outside the function

    ```javascript
    var languages = {
        names: ['C#', 'C++', 'JavaScript'],
        person: 'John Doe',
        description: function(){
            var that = this;
            return this.names.map(function(language){
                return `${that.person} knows a language called $  {language}.`
            });
        }
    };
    languages.description()

    ```

Yeah yeah, it's all nice and dandy, but why go through all that effort (which will definitely cause a load of problems) when you can use arrow functions to access the `lexical scope`?

#### Arrow Functions


Arrow functions do not create their own scope, they use `lexical scoping`, they do not have the `arguments` variable, and are `non-constructable`.

They are concise and eliminate issues when dealing with `this`.

##### Examples

```javascript

var languages = {
  names: ['C#', 'C++', 'JavaScript'],
  person: 'John Doe',
  description: function(){
    return this.names.map((language) => {
      return `${this.person} knows a language called ${language}.`
    });
  }
};
languages.description()

// Output example

// John Doe knows a language called C#. John Doe knows a language called C++. John Doe knows a language called JavaScript.
```

```javascript

// global scope
function someFunction() {
    return new Promise((resolve, reject) => {
        // if some condition
        reject('Sorry but no');
        // else
        resolve('Hurray');
    })
};

// the above function returns a promise, so let's show the extent of how useful arrow functions can be


// some class that has access to the above function
class SomeClass {

    importantVariable;
    constructor() {}

    // method
    function1() {
        someFunction()
            .then((a) => {
                console.log(a);
                // call again (let's assume it's a new function that returns a promise too)
                someFunction((b) => {
                    console.lob(b);
                    someFunction((c) => {
                        // we can use this without any problems
                        this.importantVariable = c;
                    })
                })
            })
    }
}

```

***MAGIC***

If done using function expressions, `this` would have needed to be passed down to the inner-most function using `.bind(this)`

It's not about the differences in simple examples. It's about the benefits that can be reaped when codebases become large and need maintenance. The cost of using function expressions is unreadable code, introduction of scoping bugs, and less concise syntax.

### Conclusion

Arrow functions provide concise syntax and resolve issues related to `this` as they use the `lexical scope`. 
There are times where arrow functions cannot be used, such as in contructors or functions for a prototype.

In all other cases, especially for callbacks, arrow functions are the most suitable solution.

