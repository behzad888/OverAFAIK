---
title: Writing Unit Test with Jest 
date: '2019-04-18'
spoiler: How to test everything.
---

There are many steps that every front-end developers and maintainers should know, they are:
 - [How to formatting codes](https://overafaik.com/formatting-codes)
 - [How to reporting on patterns found in ECMAScript/JavaScript code](https://overafaik.com/linting-codes)
 - [How to check object types](https://overafaik.com/flow-type-check)
 - [How to test everything](https://overafaik.com/jest)
 - [How to build the project](https://overafaik.com/build-project)
 - [How to use CI](https://overafaik.com/continous-integration)
 - How forced contributors to staying on contributing guidelines?

Why we should writing the unit test? 

> My answer is: Because I don't want my teeth are aching.

Imagine, If you don't write test your project will have a bug. Its ok cause you will fix it but maybe your solution impresses on another place. so, you will have another bug. It'll be fixed but it's not guaranteed. Then, your boss will fire you and you won't have any money then your GF will break up with you. Finally, your teeth will ache and you have no money to treat it. So, we have to writing unit tests ;)

## Testing takes too much time and effort

**There’s no time**. You’re busy already.

**There’s no obvious ROI**. You can’t get the buy-in or budget for testing efforts.

**There’s no way to test everything**. Most testing is click click clicking around every turn of your application. It takes forever and feels like a waste of time—time you want to spend shipping new features, not QAing.

Nobody has time for that. But one way or another, your application will be tested. If not by you, then by your users.

**Cross your fingers and push to prod.**

So... click click click hope for the best? That’s what we’re doing?

If we want to ship high-quality, well-tested JavaScript applications there has to be a better way.

#### Before we start we need to know:
 - What should I test?
 - When do I test it?
 - Do I need 100% coverage?
 - How many tests are enough? 

I want to show how to check out these questions in each section.

There are many test tools and framework for writing unit tests. Although, I'm advocating with a focus in React and [Jest](https://jestjs.io/) is one of the great javascript testing frameworks, I want to show you how to use it.

>Jest is a delightful JavaScript Testing Framework with a focus on simplicity.
It works with projects using: Babel, TypeScript, Node, React, Angular, Vue and more!

Let's get started by writing a test for a hypothetical function that adds two numbers. First, create a `sum.js` file:

```js
function sum(a, b) {
  return a + b;
}
module.exports = sum;
```

Then, create a file named `sum.test.js`. This will contain our actual test:

```js
const sum = require('./sum');

test('adds 1 + 2 to equal 3', () => {
  expect(sum(1, 2)).toBe(3);
});
```

Add the following section to your `package.json`:

```json
{
  "scripts": {
    "test": "jest"
  }
}
```
Finally, run `yarn test` or `npm run test` and Jest will print this message:

> PASS  ./sum.test.js  
✓ adds 1 + 2 to equal 3 (5ms)

There is something that you should know:

 - `describe`

By default, the before and after blocks apply to every test in a file. You can also group tests together using a `describe` block. When they are inside a `describe` block, the before and after blocks only apply to the tests within that describe block.

for eaxmple:

```js
describe('Event responder : Press', () =>{
  beforeEach(() => {    
    container = document.createElement('div');
    document.body.appendChild(container);
  });

  afterEach(() => {
    document.body.removeChild(container);
    container = null;
  });
});
```

You can also use nested `describe` blocks:

```js
describe('Event responder : Press', () =>{
  beforeEach(() => {    
  });

  afterEach(() => {
  });

  describe('onPressStart', () =>{
    beforeEach(() => {    
    });

    afterEach(() => {
    });  
  });
});

```

So, we can cover and block all tests by `describe`. 

In this blog we try to write unit tests for Pressing event. Press event could have many category to test such as `onPressStart`, `onPressEnd`, `onPressChange`, `Nested responders` and etc. but these four action is good enough. 

Well, Lets get started with describe all actions:

```js
describe('Event responder : Press', () =>{
  beforeEach(() => {    
  });

  afterEach(() => {
  });

  describe('onPressStart', () =>{ ... });
  describe('onPressEnd', () =>{ ... });
  describe('onPressChange', () =>{ ... });
  describe('Nested responders', () =>{ ... });
    
});
```

All describe blocks needs to determine `beforeEach` to create a component that we want to use its press event that component could be a simple `button`.

Next things we need to know is `test` or `it` method. Note that `it` is an alias of `test`.
All you need in a test file is the `test` method which runs a test. 

for example:
```js
it('is called after "mousedown" event', () => {
   ...
});
```

Last things that you need is `expect` function and matchers.`expect` is used every time you want to test value and `matchers` let you test values in different ways. You will rarely call expect by itself. Instead, you will use `expect` along with a `matcher` function to assert something about value.

```js
test('two plus two is four', () => {
  expect(2 + 2).toBe(4);
});
```

In this code, `expect(2 + 2)` returns an *expectation* object. You typically won't do much with these expectation objects except call matchers on them. In this code, `.toBe(4)` is the `matcher`. When Jest runs, it tracks all the failing matchers so that it can print out nice error messages for you.

There are many built-in and common matchers in Jest that you can use them and I think they are good enough. Although, You can add use your own matchers to Jest by `expect.extend(matchers)`. For more information see https://jestjs.io/docs/en/expect#expectvalue

Now, get back to our example. We want to test `onPressStart` event. see below:

```js
  let React;
  let ReactDOM;

  const createPointerEvent = type => {
    const event = document.createEvent('Event');
    event.initEvent(type, true, true);
    return event;
  };

  const createKeyboardEvent = (type, data) => {
    return new KeyboardEvent(type, {
      bubbles: true,
      cancelable: true,
      ...data,
    });
  };
  
  describe('Event responder: Press', () => {
    let container;

    beforeEach(() => {
      jest.resetModules();
      React = require('react');
      ReactDOM = require('react-dom');    

      container = document.createElement('div');
      document.body.appendChild(container);
    });

    afterEach(() => {
      document.body.removeChild(container);
      container = null;
    });

    describe('onPressStart', () => {
      let onPressStart, ref;

      beforeEach(() => {
        onPressStart = jest.fn();
        ref = React.createRef();
        const element = (
          <button ref={ref} onPressStart={onPressStart}>          
          </button>
        );
        ReactDOM.render(element, container);
      });

      it('is called after "mousedown" event', () => {
        ref.current.dispatchEvent(createPointerEvent('mousedown'));
        expect(onPressStart).toHaveBeenCalledTimes(1);
      });

      it('is called once after "keydown" events for Enter', () => {
        ref.current.dispatchEvent(createKeyboardEvent('keydown', {key: 'Enter'}));
        ref.current.dispatchEvent(createKeyboardEvent('keydown', {key: 'Enter'}));
        ref.current.dispatchEvent(createKeyboardEvent('keydown', {key: 'Enter'}));
        expect(onPressStart).toHaveBeenCalledTimes(1);
      });

      //...
    });
    //...
  });
```

In this example, First, I want to be sure for each `describe` block `root` element created and after remove from DOM. 

`jest.resetModules()` method resets the module registry - the cache of all required modules. This is useful to isolate modules where local state might conflict between tests.

`jest.fn()` returns a new, unused mock function. Optionally takes a mock implementation. We can have our function instead.

```js
 onPressStart = function(e){
   console.log(e);
   // Or do what do you want
 }
```

`createPointerEvent` and `createKeyboardEvent` returns new experimental event of mouse events and keybord events based on https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent and https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent.

So, to simulate events we can use React `dispatchEvent` and check valuse by matchers. In this example I used [`toHaveBeenCalledTimes`](https://jestjs.io/docs/en/expect#tohavebeencalledtimesnumber)  to ensure that a mock function got called exact number of times.

Now, You can test what you want to check but dont forget:
 - What should I test?
 - When do I test it?
 - Do I need 100% coverage?
 - How many tests are enough? 

