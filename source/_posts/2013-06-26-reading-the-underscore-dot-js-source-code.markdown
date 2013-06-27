---
layout: post
title: "Reading the Underscore.js source code"
date: 2013-06-22 22:25
comments: true
categories: 
---

Now that I feel like I've got a better understanding of the Javascript language, it's always been a side goal of mine to read through as many of the major JS frameworks, libraries, etc. as I can. I feel like being able to read through and understand what others have done to make really awesome code, will help push me along the way to be an even better coder myself. 

The <a href="http://underscorejs.org/" target="_blank">*underscore.js*</a> library has a special place in my heart as it played a major role in the early stages of learning JS. So what better place to start -- here are a few notes from my runthrough of the *underscore.js* source:

**1. Setting the stage**

  The first bit of Underscore is all about setting the stage for the rest of the library by creating variables and setting references. This alone is not all that interesting, and something most programmers do. What caught my eye was the few lines meant to save bytes and boost performance.

```javascript
var ArrayProto = Array.prototype, ObjProto = Object.prototype, FuncProto = Function.prototype;

var push             = ArrayProto.push,
      slice            = ArrayProto.slice,
      concat           = ArrayProto.concat,
      toString         = ObjProto.toString,
      hasOwnProperty   = ObjProto.hasOwnProperty;
```

  The above lines create references to prototype functions and save them to local variables. This only requires the prototype lookup once and never again--a performance boost, albeit quite marginal, that you can see in this <a href="http://jsperf.com/underscoreprotoperf#run" target="_blank">JSPerf test</a>.

  As noted in the annotations, this also saves a few bytes in the minified version of the library. For example `push.call` is quite a few characters shorter than `Array.prototype.push.call`.

**2. `(obj.length === +obj.length)`**

  What the hell does that mean?

  Turns out, it's not that crazy. It basically just tests if the object's length property is a number. The unary `+` operator on the right side coerces whatever follows to a number. The `===` conditional then returns true only if the object's length property was already a number. This basically means that the only things to pass this conditional will be an array, string or a function<sup>1</sup>. Fancy, huh?

**3. `array.push` vs. `array[array.length]`**

  In Underscore's annoted source code, the map function uses `results[results.length]` to push items to the end of an array. Curious about why they chose this implementation, I researched a bit on the differnces between using `length` instead of the `push` method. There seems to be quite a bit of discussion around this topic (more than is necessary here), but this <a href="http://stackoverflow.com/questions/614126/why-is-array-push-sometimes-faster-than-arrayn-value" target="_blank">Stack Overflow discussion</a> delves into the native C implementation of an array behind the scenes and includes some more JSPerf tests.

  Worth noting is that in the most recent version of the underscore.js source on GitHub, the map function is now using `push`.

**4. Using `each` to loop backwards**

  I always thought of `each` as a simple looping function that started at the front and worked it's way to the end. It went front to back and nothing more. In Underscore's `reduceRight` function, they pull a sweet little trick that I had never really thought of before--looping backwards through a collection using `each`. 

  The trick is to keep a simple counter that begins at the collection's length, and rewrite the index with this counter every time through the `each` loop before doing anything else. Every time through the loop, simply reduce the counter. Magic!

**5. `void 0` and `undefined` is mutable**

  On line 243 of Underscore, right in the middle of the `where` function, we are presented with the lovely `void 0`:

```javascript
_.where = function(obj, attrs, first) {
  if (_.isEmpty(attrs)) return first ? void 0 : [];
  return _[first ? 'find' : 'filter'](obj, function(value) {
    for (var key in attrs) {
      if (attrs[key] !== value[key]) return false;
    }
    return true;
  });
};
```
  What the hell does that mean? Ya, I had no idea either. First time I've ever seen it. Come to find out that the keyword `void` is a special function that will always return `undefined` regardless of it's input (and it doesn't even need parens, much like `typeof` and `instanceof`.

```javascript
void 0 // returns undefined
void(0) // returns undefined
void 98457 // returns undefined
void 'I bet this will be undefined' // returns undefined
```

  So why not just use `undefined`? Well apparently `undefined` is simply a variable belonging to the all seeing 'global object' created at run time. This means, that *technically* it is mutable, i.e. some JS asshole can change it. Therefore, by using `void 0` (or `void [anything]` for that matter)<sup>2</sup>, we can be sure that `undefined` is always the return.


>1 Functions have a length property!? This was definitely a surprise to me as well, but they sure as hell do. It's equal to number of arguments the function takes.

>2 Although we can use `void [anything]` to get undefined, it has become common practice to use `void 0` because of it's simple, straightforward and some guy said so a long time ago and everyone else agreed with him.
