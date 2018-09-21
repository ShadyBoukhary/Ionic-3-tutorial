# JavaScript Need-To-Know Part 2

## Callbacks vs Promises vs Async/Await

### Callbacks

 A callback is simply a function that is called after another function has finished executing.

 In JavaScript, a function is an object. Therefore, functions can be passed as parameters and called within another function.

 Since JavaScript is event-driven, it will continue executing code and will not wait for a response.

 This means that when writing code like this, this is going to happen:

 ```javascript
 
// http request
this.http.get('http://someApi.com/getSomeData', {headers: headers})
        .subscribe((dat: any) => { console.log('Data Retrieved')});


console.log('After HTTP call');

/* Output
After HTTP call
Data Retrieved
*/
 ```
 Why? Because JavaScript will not wait until the http call is done and the result is retrieved to continue executing code. It will continue executing code; when the completion event is triggered, the function that was passed to the call will be executed.

 Another example of a callback:
 ```javascript

function printHello() {
    console.log('Hello');
}

function doSomething(data, callback) {
    // some processing for data
    // When done:
    callback();
}

// Somewhere in script
let data = [1, 2, 15];
// call doSomething and pass it the printHello function as a parameter
doSomething(data, printHello);

// OR
doSomething(data, function() {
    console.log('Hello');
});

// OR
doSomething(data, () => {
    console.log('Hello');
});

 ```
 You can pass parameters to callback
 ```javascript
function printResults(data) {
    console.log(`The result is: ${data}`);
}

function doSomething(data, callback) {
    // some processing for data
    data.push(23);
    data[1] = data[3] * data[4];
    // When done:
    callback(data);
}

// Somewhere in script
let data = [1, 2, 15];
// call doSomething and pass it the printHello function as a parameter
doSomething(data, printHello);

// OR
doSomething(data, function(data) {
    console.log(`The result is: ${data}`);
});

// OR
doSomething(data, (data) => {
    console.log(`The result is: ${data}`);
});

 ```

 One more example:

 ```javascript
database.get('path/something', params, function(err, data, response) {
  if(!err){
    // If there was no error
    // process data
    // err is null
    // data is not null
  } else {
    // data is null
    // err is not null
    console.log(err);
  }
});
 ```
 This is great and all. We only need to check for an error once and then process our data, right? Well... in short, the answer is ***NO***!

 While callbacks are a fundamental part of JS, **HUGE** problems might occur when functions become complicated.

 ```javascript
// http://callbackhell.com/


fs.readdir(source, function (err, files) {
  if (err) {
    console.log('Error finding files: ' + err)
  } else {
    files.forEach(function (filename, fileIndex) {
      console.log(filename)
      gm(source + filename).size(function (err, values) {
        if (err) {
          console.log('Error identifying file size: ' + err)
        } else {
          console.log(filename + ' : ' + values)
          aspect = (values.width / values.height)
          widths.forEach(function (width, widthIndex) {
            height = Math.round(width / aspect)
            console.log('resizing ' + filename + 'to ' + height + 'x' + height)
            this.resize(width, height).write(dest + 'w' + width + '_' + filename, function(err) {
              if (err) console.log('Error writing file: ' + err)
            })
          }.bind(this))
        }
      })
    })
  }
})



const verifyUser = function(username, password, callback){
   dataBase.verifyUser(username, password, (error, userInfo) => {
       if (error) {
           callback(error)
       }else{
           dataBase.getRoles(username, (error, roles) => {
               if (error){
                   callback(error)
               }else {
                   dataBase.logAccess(username, (error) => {
                       if (error){
                           callback(error);
                       }else{
                           callback(null, userInfo, roles);
                       }
                   })
               }
           })
       }
   })
};
 ```

 When functions become complicated we get:
 * **Unreadable code**
 * **Unmaintainable code**
 * **Possibility of a missing error check**
 * **The DRY Principle is shat (past tense of shit) on and thrown in trash**

So, what's the solution?

## Promises

A Promise is an object that represents the eventual value of an asynchronous operation: Completion or Failure of said operation.

Promises promise (pun not intended) to solve our Callback problems. No more nested callbacks and obscure error-handling.

Example of a promise:

```javascript
function doSomething() {
    // create a new promise
    return new Promise((resolve, reject) => {
        someOperationThatUsesCallbacks(function(err, data) {
            if (!err) {
                // resolve the promise and return the data
                resolve(data);
            } else {
                //reject the promise and return an error
                reject(err);
            }
        })
    });
}

// somewhere in your code
doSomething()
    .then((data) => console.log(data))
    .catch(e = console.log(e));

```
Here we did not need to pass a callback when calling doSomething(), we used a much cleaner promise-based solution.


Let's take the `verifyUser` example and refactor it using Promises:

```javascript
// Assuming .verifyUser and .getRoles return a promise
const verifyUser = function(username, password) {
    dataBase.verifyUser(username, password)
        .then((userInfo) => {
            return dataBase.getRoles(username)
                .then((roles) => {
                    return datBase.logAccess(username)
                        .then(() => {
                            someFunction(userInfo, roles);
                        })
                });
        })
        .catch(e => console.log(e));
}
```

The code is now much easier to read and the DRY principle is safe. We only had to have 1 error-handling statement instead since every function will return a promise. If an error occurs in the inner-most promise, the promise will be rejected and returned upwards until it reaches the original caller that handles the error. 

However, we still have the problem of nested promises.

Here's an example of something I wrote once. I was too lazy to refactor it. I changed the names of the variables in the code for this tutorial.

```javascript
    this.storage.get('something')
      .then((var1) => {
        this.list1 = var1;
        this.storage.get('somethingElse')
          .then((locs) => {
            this.storage.get('anotherThing')
              .then((keys) => {
                this.storage.get('oneMoreThing')
                  .then((varKeys) => {
                    if (var1) {
                      for (let i = 0; i < var1.length; i++) {
                        this.list1[i].$key = varKeys[i];
                      }
                      for (let i = 0; i < keys.length; i++) {
                        this.locations[keys[i]] = locs[i];
                      }
                      this.storage.get('anotherOneMoreThing')
                        .then((templateKeys) => {
                          this.storage.get('PleaseMakeItStop')
                            .then((templates) => {
                              for (let i = 0; i < templateKeys.length; i++) {
                                this.varTemplates[templateKeys[i]] = templates[i];
                              }
                            })
                        })
                    }

                  })
              })
            this.storage.get('again?')
              .then((prof) => {
                this.authenticatedUserProfile = prof;
                this.events.publish('ugghhh', {
                  userProfile: this.authenticatedUserProfile
                });
              })
            this.storage.get('one last time')
              .then((key) => {
                this.authenticatedUserProfile.$key = key;
              })


            if (!dismissed) {
              dismissed = true;
              loading.dismiss();

            }
          }).catch((e => alert(e.message)));
      }).catch((e) => alert(e.message));

```

So... as you can see, things can get really messy even with promises. Originally, it was very simple. Then, I kept coming back to it and adding more promises and i was too lazy to refactor it. So now it is very hard to read and very hard to maintain - it does work flawlessly though :).

So, what's the solution?

## Async/Await

Async/Await... the solution to all our problems... our gift from ES6...
Let's all take a moment and thank Brendan Eich and Ecma International for granting us this gift.

Basically, `Async/Await` is a special syntax used to work with promises. The `async` keyword allows for the use of `await`; it wraps your function's return value inside a promise. `Await` unwraps the result of the promise like using `.then()`.
Here's the previous example using async/await:
```javascript

function doSomething() {
    // create a new promise
    return new Promise((resolve, reject) => {
        someOperationThatUsesCallbacks(function(err, data) {
            if (!err) {
                // resolve the promise and return the data
                resolve(data);
            } else {
                //reject the promise and return an error
                reject(err);
            }
        })
    });
}

// somewhere in your code
async function someFunctionInYourCode() {
    try {
        let data = await doSomething();
    } catch(e) {
        console.log(e);
    }
}

```
Wait a second...
> But Shady, this doesn't look that much nicer, and it actually made our code longer...

dOeSn'T lOoK tHaT mUcH nIcEr, mAdE oUr cOdE lOnGer... shhh this isn't even my final form
<img src='./Mocking-Spongebob.jpg'>

Here's where the magic happens, let's refactor my old code (about time...) using async/await instead of the regular promise syntax.


```javascript

/*  NEW REFACTORED CODE   */
/*  The irrelevant code like loops has been removed from both old and new code to highlight the differences in promises only*/
async function refactoredFunction() {
    try {
        let list1 = await this.storage.get('something');
        let locs = await this.storage.get('somethingElse');
        let keys = await this.storage.get('anotherThing');
        let varKeys = await this.storage.get('oneMoreThing');
        let templateKeys = await this.storage.get('anotherOneMoreThing');
        let templates = await this.storage.get('PleaseMakeItStop');
        let prof = await this.storage.get('again?');
        let key = await this.storage.get('one last time');
        // process data 
        
    } catch(e) {
        console.log(e);
    }
}
/* *******************************************************************
THE ABOVE AND BELOW CODE BLOCKS ARE EXACLTY EQUIVALENT
********************************************************************** */
/* OLD CODE */
this.storage.get('something')
      .then((var1) => {
        this.list1 = var1;
        this.storage.get('somethingElse')
          .then((locs) => {
            this.storage.get('anotherThing')
              .then((keys) => {
                this.storage.get('oneMoreThing')
                  .then((varKeys) => {
                    if (var1) {
                      this.storage.get('anotherOneMoreThing')
                        .then((templateKeys) => {
                          this.storage.get('PleaseMakeItStop')
                            .then((templates) => {
                                // some code i removed to highlight relevant code only
                            })
                        })
                    }

                  })
              })
            this.storage.get('again?')
              .then((prof) => {
                  // some code i removed to highlight relevant code only
              })
            this.storage.get('one last time')
              .then((key) => {
                // some code i removed to highlight relevant code only
              })

          }).catch((e => alert(e.message)));
      }).catch((e) => alert(e.message));

```

Better rethink that statement about it not being a lot nicer, huh? Now imagine doing that using callbacks only... Good luck with that!

So, Async/Await brought us:
* **Incredibly clean and readable code**
* **Much shorter code**
* **DRY Principle Fullfilled**
* **Zero nested statements**
* **One error-handling statement for the entire code**
* **Synchronous-looking statements for asynchronous code**

***Touche***

## Conclusion
Use `Async/Await` whenever possible. **WHENEVER POSSIBLE**. If you don't, well... it's your decision. A lot of tutorials don't use them because the authors are not aware of their existance (this is very common).

However, there will be instances where you will not be able to use them (i'm looking at you, `Rxjs`) but there are workarounds to make your code cleaner.

Long live`Async/Await`, and long live `JavaScript`.

###### Shady Boukhary