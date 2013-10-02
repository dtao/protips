Here's a disorganized jumble of JS performance tips I've accumulated over time. I've considered posting these elsewhere, like [on my blog](http://philosopherdeveloper.com/); but that didn't feel like the right place for them. Some or most of these you may already know, especially if you're a seasoned JavaScript developer.

I'll start with JavaScript-specific tips, then provide a couple of more general pointers.

`for`..`in` is faster than `Object.keys`
---------------------------------------

This is one I just recently discovered. And it sort of surprised me, though I'm not exactly sure why. Maybe it's because I have come to associate [the `in` keyword with slowness](http://jsperf.com/is-in-slow); or maybe it's because I assumed `Object.keys` would be heavily optimized by the browser (i.e., maybe every object internally already has an array of its keys, in which case `Object.keys` could just return a read-only view of that array). In any case, the truth is that if you want to iterate over an object's properties, a plain vanilla `for`..`in` loop is your best bet from a performance perspective.

*But what about `hasOwnProperty` and non-enumerable properties etc.?* Yes, yes. There are gotchas associated with `for`..`in`, and it's important to be aware of them. But again, I'm mainly focused on performance here. And in your run-of-the-mill situation, where you just have a normal object that was initialized with `{ ... }`, a `for`..`in` loop works just fine.

Use `Date.now()` if you can
---------------------------

It isn't supported in every browser (in particular, IE < 9), but [`Date.now()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/now) is how you should prefer to get the current timestamp as an integer. The alternative, `new Date().getTime()`, is [a lot slower](http://jsperf.com/date-now-versus-new-date-gettime) AND [creates a lot more garbage](https://github.com/jashkenas/underscore/pull/1235#issuecomment-25007606).
