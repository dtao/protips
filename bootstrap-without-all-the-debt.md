My old coworker [Matt Copeland](https://coderwall.com/matthewcopeland) recently wrote an excellent article called [Bootstrap Bankruptcy](http://matthewcopeland.me/blog/2013/11/04/bootstrap-bankruptcy/), in which he compares using [Bootstrap](http://getbootstrap.com/) on a website to taking out a mortgage:

> When it comes to custom interfaces, Bootstrap isn't a box of bricks, panels,
> walls and doors with which you can build anything you want. Bootstrap is more
> like a mortgage. Mortgages can be good things. You didn't have the time to
> save up and buy your house in cash. You needed a mortgage to get a roof over
> your head. That's similar to using bootstrap to get some UI in place when you
> don't have the resources to build UI from scratch.

The way Bootstrap is generally used
-----------------------------------

Matt here is describing what I would guess the majority of users experience at some point with Bootstrap: what at first seemed like a great set of building blocks eventually starts to morph into a mountain of technical debt. Your HTML is littered with deeply nested structures with Bootstrap-specific attributes like `"row"` and `"col-lg-10"`. It isn't semantic, and it's difficult to imagine pulling Bootstrap *out*. In this common case I agree with Matt, and I think an argument can be made that a serious flaw in the framework is how gleefully it seems to facilitate such tight coupling and non-semantic markup.

A big part of the problem, of course, is that Bootstrap uses exactly this kind of markup throughout the examples in its documentation. But I don't fault the Bootstrap developers for that; in my opinion they've optimized for immediate usability---the kind where you can just copy and paste some code and it works right away---and that is a large part of the framework's success. Also, though the framework uses LESS internally for organization and reuse of styling rules, it is distributed mainly as CSS, which requires no additional setup and can simply be dropped into a website for low-friction consumption.

These shortcuts are a significant part of why Bootstrap draws so much criticism from some circles (you may not be aware of it, but it's out there if you want to go look for it); paradoxically, they're also a large part of of what has made it so popular. But that's another discussion. On to the promised milk and honey.

A better way to use Bootstrap
-----------------------------

Fundamentally, I think it's valuable to conceptualize Bootstrap as two distinct things:

1. A collection of carefully-crafted styling rules (think: the CSS properties)
2. A vocabulary for referring to those rules (think: the actual *class names*)

The first really is a set of building blocks. Just look at Bootstrap's website and you can see it all there: typography, spacing, color scheme, basic layout. Forget about class names and special attributes for a moment and just consider what you *see*. These are all reusable pieces. The trick is how you consume them.

Bootstrap's *vocabulary* is where we see so many devs getting into trouble. Using `<div class="container">` and so forth directly in your markup is how you can end up trapped by some serious Bootstrap lock-in.

The trick to nabbing the benefits of the first item, without incurring all the debt of the second, is to use semantic markup on your website and shoehorn Bootstrap's styles in by way of a higher-level stylesheet language such as [SASS](http://sass-lang.com/).

Here's what I mean. Say you have a Bootstrap-y HTML layout like this:

```html
<body>
  <div class="container">
    <div class="page-header">
      <h1>Company Title</h1>
      <p class="lead">Slogan</p>
    </div>

    <div class="row">
      <div class="col-lg-3">
      </div>

      <div class="col-lg-9">
      </div>
    </div>
  </div>
</body>
```
