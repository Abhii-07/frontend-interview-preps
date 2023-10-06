# Polyfills

Polyfills in JavaScript are code snippets or scripts that provide modern functionalities to older browsers or environments that lack support for those features.
Some new features doesn't work on older browser, we write polyfill of such functionality to provide support in older browser.

**Why do you need to know to write polyfill?**

It gives you better understanding of the prgramming.

[Table of Contents](#polyfills)

- [Promise](#promise)
- [Promise All](#promise-all)
- Call, Bind, Apply
- ForEach, Map, Reduce

[Go Back ↩](../README.md)

## Promise

Let's see how you write an actual promise in JavaScript

```javascript
let myP = new Promise((res, rej) => {
	setTimeout(() => { res('success') }
})

myP.then(data => console.log(data)).catch(err => console.error(err)).finally(data => console.log('everything is done')
```

Let's write polyfill using JS Classes

```javascript
class MyPromise {
  constructor(executor) {
    this.state = "Pending";
    this.value = null;
    this.reason = null;
    this.thenCB;
    this.catchCB;
    this.finallyCB;
    const res = dataValue => {
      if (this.thenCB) {
        this.state = "Fulfilled";
        this.thenCB(dataValue);
      } else {
        this.value = dataValue;
      }
      if (this.finallyCB) {
        this.finallyCB();
      }
    };
    const rej = errorValue => {
      if (this.catchCB) {
        this.state = "Rejected";
        this.catchCB(errorValue);
      } else {
        this.reason = errorValue;
      }
      if (this.finallyCB) {
        this.finallyCB();
      }
    };
    try {
      executor(res, rej);
    } catch (err) {
      rej(err);
    }
  }
  then(cb) {
    if (this.state === "Pending" && this.value) {
      this.state = "Resolved";
      const data = cb(this.value);
    } else {
      this.thenCB = cb;
    }
    return this;
  }
  catch(cb) {
    if (this.state === "Pending" && this.reason) {
      this.state = "Rejected";
      const data = cb(this.value);
    } else {
      this.catchCB = cb;
    }
    return this;
  }
  finally(cb) {
    if (this.state !== "Pending" && this.value) {
      cb(this.data);
    } else {
      this.finallyCB = cb;
    }
    return this;
  }
}

let executorFunction = (resolve, reject) => {
  resolve("Success");
};

let MPromise = new MyPromise(executorFunction);

MPromise.then(data => {
  console.log(`Data recieved in then is -> ${data}`);
  return "Ish";
})
  .catch(err => console.log(`Error recieved in catch is -> ${err}`))
  .finally(data => console.log(`Data recieved in finally -> ${data}`));
```

This is a simple promise polyfill that takes care of most of the cases except for **chaining then and catch**
Please folllow this video [Writing Polyfill for Promise in JavaScript](https://www.youtube.com/watch?v=lKdFKuttdfM) to get a better understanding.

## Promise All
Let's see how to write Promise.all in Javascript

```javascript
 const promiseOne= Promise.resolve(4);
 const promiseTwo= Promise.resolve(true);
 const myPromiseAll= Promise.all([promiseOne,promiseTwo]);
 myPromiseAll.then((response)=>{
	console.log(response)// [4,true]
	}).catch((error)=>{
	console.log(error)
	})
```
Now let's write polyfill for Promise.all
```javascript
function isPromise(promise) {
  if (typeof promise === 'object' && typeof promise.then === 'function') {
    return true;
  }
  return false;
}
function promiseAll(promises){
  const resolvedPromiseResult=[];
  let resolvedPromiseCount=0;
  return new Promise((resolve,reject)=>{
    promises.forEach((promise,index) => {
    if(isPromise(promise)){
        promise.then((response)=>{
          resolvedPromiseResult[index]=response;
          resolvedPromiseCount+=1;
          if(resolvedPromiseCount===promises.length){
            resolve(resolvedPromiseResult)
          }
        }).catch((err)=>{
          reject(err)
        });
    }else{
      resolvedPromiseResult[index]=promise;
      resolvedPromiseCount+=1;
      if(resolvedPromiseCount===promises.length){
        resolve(resolvedPromiseResult)
      }
    }
    });
  });
}
// All promises resolved case

const promiseOne= Promise.resolve(4);
const promiseTwo= Promise.resolve(5);
const myPromiseAll= promiseAll([promiseOne,promiseTwo]);

myPromiseAll.then((response)=>{
  console.log(response);
}).catch(err=>{
  console.log(err);
});

// One of the promise got rejected case

const promiseThree= Promise.reject("Some error!");
const myPromiseAllRejected=promiseAll([promiseOne,promiseThree,promiseTwo]);

myPromiseAllRejected.then((response)=>{
  console.log(response)
}).catch(err=>{
  console.log(err) //Some error! 
})

// Non promise case

const promise1 = Promise.resolve(3);
const promise3 = 42
const promise2 = new Promise((resolve, reject) => {
  setTimeout(reject, 100, 'foo');
});
promiseAll([promise1, promise2, promise3]).then((values) => {
  console.log(values);
});
```
