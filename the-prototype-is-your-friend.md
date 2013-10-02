The `this` keyword in JavaScript *can* be really confusing. That I won't dispute. And so a lot of developers I really respect actually advocate for avoiding it altogether, which is [definitely possible](http://blog.thepete.net/blog/2012/02/06/class-less-javascript/). You can sort of ignore the existence of JavaScript's prototype system and follow your own object-building approach:

```javascript
function createValueObject(value) {
  return {
    get: function() { return value; }
  };
}
```

Or you can embrace the prototype system:

```javascript
function ValueObject(value) {
  this.value = value;
}

ValueObject.prototype.get = function() {
  return this.value;
}
```

Obviously this a contrived example; it's just meant to concisely illustrate the two approaches I'm talking about.

The *readability* of these two examples is a debatable issue. There are certainly many valid reasons for preferring the former, including its avoidance of `this`. However, *if* performance is a serious concern, you [should consider going with the second approach](http://jsperf.com/prototype-vs-factory-performance/3). Using a prototype to define the methods an object should have is pretty much faster across the board, though *how much faster* depends on the browser (only a bit in Firefox, a lot in Chrome).

![Prototype vs. factory method performance](https://coderwall-assets-0.s3.amazonaws.com/uploads/picture/file/2147/PrototypeVsFactoryMethod.png)

The *why* is a question for another post. I can't speak with authority on that, but I have a strong suspicion it relates to JS engines' use of hidden classes internally and the efficiency of vtables. (To get an idea of what I'm talking about, I recommend reading [Eric Lippert's series on implementing the virtual method pattern in C#](http://blogs.msdn.com/b/ericlippert/archive/2011/03/17/implementing-the-virtual-method-pattern-in-c-part-one.aspx), which goes over the efficiency considerations in designing a method lookup system. Clearly C# is not JavaScript, but I think similar principles may be at play here.)