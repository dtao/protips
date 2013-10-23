Ruby is a language that Yukihiro Matsumoto designed to be *fun*, to optimize developer happiness. I appreciate this more and more whenever I discover some quirky feature hidden in the nooks and crannies of the language. This is going to be a quick survey of some of those interesting, "fun" features.

Hopefully the tongue-in-cheek title makes it clear that I'm not recommending any of this for real, production code. That said, if you like any of these tricks and your teammates do too, I don't think they're necessarily harmful. Knock yourselves out, I say!

## Using `&` to create procs

Some Ruby devs know this, some don't. Say I have an array of strings and I want to capitalize them.

```ruby
strs = ['foo', 'bar', 'baz']
caps = strs.map { |str| str.upcase }
```

Pretty familiar. I can make this a bit more concise using the `&` operator:

```ruby
caps = strs.map(&:upcase)
```

The `&` takes any object and calls `to_proc` on it. And for the `Symbol` class, the `to_proc` method gives you a proc that, given some object, will send itself (the symbol) as a message to that object. So the above is essentially the same as this:

```ruby
caps = strs.map { |str| str.send(:upcase) }
```

Now, what if instead of calling an *instance* method, you want to pass an object as a *parameter* to a method?

```ruby
def piglatinify(word)
  word[1..-1] + word[0] + 'ay'
end

# Can we do this with & instead?
piglatinified = strs.map { |str| piglatinify(str) }
```

This can be done, as it turns out. First get a `Method` object using `method`, then stick a `&` in front of it to get a proc:

```ruby
piglatinified = strs.map(&method(:piglatinify))
```

## More fun with blocks and lambdas

Which brings me to another little trick. You have probably used blocks like this:

```ruby
# Define a method that *takes* a block
def apply_block(arg, &block)
  yield arg
end

# Call the method *with* a block
apply_block('foo') do |str|
  str.upcase
end
```

Meanwhile, I'm sure you've probably used lambdas before:

```ruby
upcase_lambda = lambda { |str| str.upcase }
upcase_lambda.call('foo')
```

Did you know you can easily use a lambda as a block?

```ruby
apply_block('foo', &upcase_lambda)
```

You can also use the `[]` method to invoke a lambda (rather than using `call`):

```ruby
upcase_lambda['foo']
```

Also, did you know that as of Ruby 1.9 there's an alternate syntax for defining lambdas? Frankly, it's pretty weird and I tend to avoid it. But it exists, and it looks like this:

```ruby
upcase_lambda = -> s { s.upcase }
```

What's this? You already knew all that? Fine. I hope you realize there are no prizes here.

## Playing with heredocs

You've probably used this feature of Ruby once or twice:

```ruby
heredoc_haiku = <<-EOT
  This is a heredoc
  It provides a nice syntax
  For multiline strings
EOT
```

Now, there is a dilemma here: the syntax is convenient, but it is takes white space literally. So if you put indentation in the *string* to remain consistent with your *code*, that indentation will be preserved. This is why you see ugly code like this sometimes:

```ruby
module Foo
  class Bar
    def print_haiku
      puts <<-EOT
This is a heredoc
It provides a nice syntax
For multiline strings
      EOT
    end
  end
end
```

As it turns out, you can manipulate a heredoc string from the point where it's declared:

```ruby
heredoc_haiku = <<-EOT.gsub(/^\s+/, '') # Eliminate leading space
  This is a heredoc
  It provides a nice syntax
  For multiline strings
EOT
```

Actually, "manipulate" is too strong a word. You can call *any* method on a heredoc string. For example, here we use heredoc syntax to get an array of sentences, each comprising an array of words:

```ruby
sentences = <<-EOT.lines.map(&:split)
  The quick brown fox jumped over the lazy dogs.
  The rain in Spain stays mainly in the plain.
  All work and no play makes Jack a dull boy.
EOT
```

## Playing with `*`, the splat operator

The splat operator (`*`) can be quite useful. You've probably seen it used to define a method that accepts a variable number of arguments:

```ruby
def count(*args)
  puts "Received #{args.length} arguments."
end

count(1, 2, 'foo', nil, false) # => 5
```

A helpful way to think of splatting is that it takes a value or an array of values, and evaluates them as if they'd been expressed literally in your code.

Here's an example:

```ruby
def sum(x, y, z)
  x + y + z
end

values = [1, 2, 3]

# This will be evaluated as if you had written the code:
# sum(1, 2, 3)
sum(*values)
```

And it works the other way, as we've already seen:

```ruby
def sum(*values)
  values.inject(0) { |x, y| x + y }
end

# This will be evaluated so that the 'values' argument
# gets passed in as [1, 2, 3].
sum(1, 2, 3)
```

There's a lot more you can do with splats. For example, say you have an array and you want to assign the first element to one variable, and the rest to another:

```ruby
arr = (1..10).to_a
first, *rest = arr
```

You can also use `*` on a single value, which will evaluate to the value if it isn't `nil` or otherwise will act as if it weren't there at all. Here's what I mean:

```ruby
def pack(x, y, z)
  [*x, *y, *z]
end
```

The above method will create an array of the value(s) from `x`, `y`, and `z` that are not `nil`. In other words it's functionally the same as:

```ruby
def pack(x, y, z)
  [x, y, z].compact
end
```

This opens up another scenario. Let's implement a `to_hash` method on `Enumerable`, where we'll accept a block that takes each element and can either return a key/value pair, or just a key (in which case the value will be the original element).

```ruby
module Enumerable
  def to_hash(&block)
    hash = {}
    self.each do |e|

      # If (yield e) returns TWO values, then val will take on the second one;
      # otherwise, it will be assigned the value of e.
      key, val = [*(yield e), e]

      hash[key] = val
    end
    hash
  end
end
```

Yeah, splats are fun. I've wasted a lot of time playing with them for no good reason.

## Chaining enumerators

What happens if you call `map` on an array and don't pass a block? You get an `Enumerator` object.

```ruby
e = [1, 2, 3].map
e.class # => Enumerator
```

One thing that's helpful to know here is that you can chain these together. For example, suppose you want to `map` with some logic that needs an element's index. You could always do it the sloppy way:

```ruby
i       = 0
results = array.map do |e|
  # Do something w/ e and i
  i += 1
end
```

But you could chain this atop `each_with_index` instead:

```ruby
results = array.each_with_index.map do |e, i|
  # Do something w/ e and i
end
```

Or you might like to group a sequence with `each_slice`:

```ruby
pairs = [1, 2, 3, 4, 5, 6].each_slice(2).map(&:reverse)
# => [[2, 1], [4, 3], [6, 5]]
```

## Abrupt conclusion

Well, I think that's enough for now. Maybe I'll write another protip with even more quirky Ruby features down the road.

If you learned even one new thing from this protip, I'm glad to have taught you something! Even if it isn't that useful. On the other hand, if you already knew all of this and I just wasted ~5 minutes of your time, my sincere apologies. But it was all free, so... you get what you pay for, eh?
