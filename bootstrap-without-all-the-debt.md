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

Bootstrap's *vocabulary* is where we can get into trouble. Using `<div class="container">` and so forth directly in your markup is a sure way to end up trapped by some serious Bootstrap lock-in.

To enjoy the benefits of Bootstrap's useful styles, without incurring all the debt of its vocabularity, use **semantic markup** on your pages and shoehorn Bootstrap's styles in by way of a higher-level stylesheet language such as [SASS](http://sass-lang.com/).

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
        <ul class="list-group">
          <li class="list-group-item"><a href="/">Home</a></li>
          <li class="list-group-item"><a href="/about">About</a></li>
          <li class="list-group-item"><a href="/contact">Contact</a></li>
        </ul>
      </div>
      <div class="col-lg-9">
        <h1>Heading</h1>
        <p>Lorem ipsum dolor sit amet [...]</p>
      </div>
    </div>
  </div>
</body>
```

Even for something so simple, we've already started down a pretty tightly-coupled path. Let's try starting over, sticking with semantic markup.

```html
<body>
  <header>
    <h1>Company Title</h1>
    <p>Slogan</p>
  </header>
  <main>
    <nav>
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about">About</a></li>
        <li><a href="/contact">Contact</a></li>
      </ul>
    </nav>
    <article>
      <h1>Heading</h1>
      <p>Lorem ipsum dolor sit amet [...]</p>
    </article>
  </main>
</body>
```

There, much better. Now there's no trace of Bootstrap in the markup itself. (Notice I am using some HTML5-specific tags such as `<header>`, `<main>` and `<article>`; if you were targeting older browsers you could still easily keep your markup semantic by using `<div>` elements with meaningful, non-Bootstrap class names.)

Now using SASS we can still style this page using Bootstrap's style definitions by `@extend`-ing Bootstrap classes:

```scss
@import "bootstrap";

body { @extend .container; }

body > header { @extend .page-header;
  p { @extend .lead; }
}

body > main { @extend .row;
  nav { @extend .col-lg-3;
    ul { @extend .list-group;
      li { @extend .list-group-item; }
    }
  }
  article { @extend .col-lg-9; }
}
```

Here's the result:

![Our Bootstrap-styled layout](/images/BootstrapLayout.png)

Let's try another example: a login form.

```html
<form>
  <div class="form-group">
    <label for="email">E-mail</label>
    <input type="text" name="email" class="form-control" />
  </div>
  <div class="form-group">
    <label for="password">Password</label>
    <input type="password" name="password" class="form-control" />
  </div>
  <div class="checkbox">
    <label for="remember">
      <input type="checkbox" name="remember"> Remember me
    </label>
  </div>
  <input type="submit" class="btn btn-primary" value="Log In" />
  <a href="../" class="btn btn-warning">Back</a>
</form>
```

Again, lots of Bootstrap in there: `<div class="form-group">`, `<input class="btn btn-primary">`, etc. Let's try to be more semantic again:

```html
<form id="login-form">
  <div class="email field">
    <label for="email">E-mail</label>
    <input type="text" name="email" />
  </div>
  <div class="password field">
    <label for="password">Password</label>
    <input type="password" name="password" />
  </div>
  <div class="remember field">
    <label for="remember">
      <input type="checkbox" name="remember"> Remember me
    </label>
  </div>
  <div class="actions">
    <input type="submit" value="Log In" />
    <a href="../">Back</a>`
  </div>
</form>
```

And again, we'll borrow from Bootstrap in our SASS:

```scss
#login-form {
  .field { @extend .form-group;
    input[type="text"],
    input[type="password"] { @extend .form-control; }

    &.remember { @extend .checkbox; }
  }

  .actions { @extend .form-group;
    input[type="submit"] { @extend .btn, .btn-primary; }
    a { @extend .btn, .btn-warning; }
  }
}
```

How does it look?

![Our Bootstrap-styled login form](/images/BootstrapForm.png)

Graduating from Bootstrap
-------------------------

So you can see that you don't necessarily need to clutter your markup with Bootstrap terminology in order to access the framework's handy selection of styles.

That said, of course there is more to the idea of "Bootstrap Bankruptcy" than the tech debt of tightly-coupled markup. Inevitably most sites that develop a significant user- or reader-base will eventually want to graduate from Bootstrap and define their own set of styles. The problem is typically that this can require great effort when Bootstrap is so deeply integrated into every corner of your HTML.

The approach I've suggested, of adhering to semantic markup even while leveraging Bootstrap's style definitions, gives you the best of both worlds. You get to throw together something attractive quickly, building off the work of the Bootstrap team. Then when it's finally time to move on and start work on a custom design, you can throw out the `@import "bootstrap"` line and you'll have a clear, semantic HTML structure to work with.