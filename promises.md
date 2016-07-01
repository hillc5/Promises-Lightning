### Kyle Simpson

![alt-text](https://qph.is.quoracdn.net/main-qimg-b784cc8c42cc34950eac79ca3c44ff1e?convert_to_webp=true "You Don't know JS")

Video Workshop: https://frontendmasters.com/courses/rethinking-async-js/

---


## Callbacks
```javascript
// First half of program
setTimeout(() => {
  // Second half of program
  console.log('callback!');
}, 1000);
```

---


## The Problem With Callbacks
#### (Callback Hell)

* Inversion of Control
* Hard to Reason About


----


### Inversion of Control
#### It's a Trust Issue

* Callback gets called
* Callback not called too early, or too late
* Callback not called multiple times
* Errors are not swallowed

Callbacks themselves do not have a solution for the trust issues that arise from IOC


----


## Hard to Reason About

* Hard to handle Temporal Dependency
  * Managing state for 'parallel' calls
  * Pyramid of Doom
* Can't reason about them linearly/sequentially
* Code makes "Non-Local" jumps


----


## Parallel callback scenario


```javascript
function fakeAjax(url,cb) {
	let fake_responses = {
		"file1": "The first text",
		"file2": "The middle text",
		"file3": "The last text"
	},
	delay = (Math.round(Math.random() * 1E4) % 8000) + 1000;

	console.log("Requesting: " + url);

	setTimeout(() => {
		cb(fake_responses[url]);
	},delay);
}

```

----


```javascript
function getFile(file) {
	fakeAjax(file, text => {
		printText(file, text);
	});
}

// request all files at once in "parallel"
getFile("file1");
getFile("file2");
getFile("file3");
```


----


```javascript
const FILE_STATE = {};

function printText(file, text) {
	// Haven't seen this file, so add it to the state container
	if (!FILE_STATE[file]) {
		FILE_STATE[file] = { canPrint: true, text: text };
	}

	let files = ['file1', 'file2', 'file3'];

	for (let i = 0; i < files.length; i++) {
		let fileState = FILE_STATE[files[i]];
		if (fileState) {
			// Is this the first time we've seen this file?
			if (fileState.canPrint) {
				console.log(fileState.text);
				// Mark it false so we only print it once.
				fileState.canPrint = false;
			}
		} else {
			return;
		}
	}

	console.log('Complete');
}
```


---


## Promises
#### What are they?

* Future Value
  * A way to reason about a value without worrying about time
* Continuation Event
* Callback Manager??

Promises don't remove the reliance on callbacks, but rather they instill
into the callback process a level of trust.

----


## Promises
#### How do they solve our problems?

* Only resolved once
* Only either success or error
* exceptions become errors (not swallowed)
* Immutable once resolved


---


## API

```javascript
// How we construct a new promise
let promise = new Promise((resolve, reject) => {
  // call resolve() if successful
  // otherwise call reject()
})

promise.then(
  // Success Event/Error Event
  success,
  error
)

```


----


## API Example

```javascript
function evenAsync(num) {
  return new Promise((resolve, reject) => {
    value % 2 === 0 ? resolve(num) : reject(`${num} is not even`);
  });
}

const evenCall = evenAsync(2);

evenCall.then(
(data) => {
  console.log(`${data} is even!`);
},
(error) => {
  console.log(`There was an error: ${error}`);
});
```

---


## Chaining
Every `then`/continuation function returns a Promise
* Implicitly if there's no return
* If we return a value, it's wrapped in a resolved Promise

```javascript
evenAsync(2)
  .then(value => 4) // Gets Wrapped in a Promise
  .then()
  .then(value => {
    console.log(value); // undefined
    return evenAsync(4);
  })
  .then(value => {
    return evenAsync(value + 1);
  })
  .then(value => {
    console.log('Did I make it?'); // No, did not make it
  })
  .catch(error => {
    console.log(`Error ${error}`);
  })
```


----


## Previous Example With Promises

```javascript
  function getFile(file) {
    return new Promise((resolve, reject) => {
      fakeAjax(file, text => {
        resolve(text);
      })
    })
  }

  function output(text) {
    console.log(text);
  }
```

----


```javascript

  let p1 = getFile('file1'),
      p2 = getFile('file2'),
      p3 = getFile('file3');

  p1
  .then(output)
  .then(() => p2)
  .then(output)
  .then(() => p3)
  .then(output)
  .then(() => { output('Complete'); });
```


---


# Utilities

## Promise.resolve
* Resolves any value/thenable as a Promise
* Good for:
  * Ensuring that any promise ready api gives you the Promise implementation you are using
  * Wrapping sync calls to be asynchronous

```javascript
let p = Promise.resolve(42);
p.then(value => {
  console.log(`${value}, The meaning of life`)
});
```


----


## Promise.all (gating)
* Does not resolve until all of the component promises have resolved.
  * If any fail out, the entire list of promises fail out
* Get back the values from each promise in the order that they are called

```javascript

  let p1 = getFile('file1'),
      p2 = getFile('file2'),
      p3 = getFile('file3');

  Promise.all([p1, p2, p3])
    .then(values => {
      values.forEach(output);
      output('Complete');
    });
```


----


## Promise.race
* Like `Promise.all` but resolves as soon as the first component promise resolves
* Great for timeouts

```javascript
let p1 = someAsyncCall(),
    timeout;

    timeout = new Promise((resolve, reject) => {
      setTimeout(() => {
        reject('Timeout!');
      }, 5000)
    });

Promise.race([p1, timeout])
  .then(
    success,
    error)
```
