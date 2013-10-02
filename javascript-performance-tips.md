Here's a disorganized jumble of JS performance tips I've accumulated over time. I've considered posting these elsewhere, like [on my blog](http://philosopherdeveloper.com/); but that didn't feel like the right place for them. Some or most of these you may already know, especially if you're a seasoned JavaScript developer.

I'll start with JavaScript-specific tips, then provide a couple of more general pointers.

Array pre-allocation isn't a thing
----------------------------------

I've seen this misinformation spread in a few places here and there. Devs coming from slightly lower-level language backgrounds tend to come up with this idea: to populate an array, use the `Array` constructor and specify a "capacity" to pre-allocate the array. The assumption here is that since `push` presumably does some sort of dynamic resizing under the hood--e.g., doubling and copying elements whenever a limit is reached, like Java's `ArrayList<E>` or .NET's `List<T>`--you can save on that resizing cost if you specify a number of elements in advance.

It just isn't true, my friends.

function preallocate(N) {
  var preallocated = new Array(N);
  for (var i = 0; i < N; ++i) {
    preallocated[i] = someValue(i);
  }
  return preallocated;
}

function justPush(N) {
  var array = [];
  for (var i = 0; i < N; ++i) {
    array.push(someValue(i));
  }
  return array;
}

function someValue(x) {
  return x;
}
