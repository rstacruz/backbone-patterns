# Backbone patterns

<!-- {h1:.massive-heading.-with-tagline} -->

> Building apps with Backbone.js

Here, I try to document the good practices that our team has learned along the
way building [Backbone][bb] applications.

### Assumptions

This document assumes that you already have some knowledge of [Backbone.js][bb],
[jQuery][jq], and of course, JavaScript itself.

[rsc]: http://ricostacruz.com/
[bb]: http://documentcloud.github.com/backbone/
[jq]: http://jquery.com/

Anti-patterns
=============

Here are a few common practices that I believe should generally be avoided.

$() abuse
---------

jQuery allows you to defer execution of code until when the DOM is fully-loaded
with [$(document).ready(...)][jquery.ready], or its short form, `$(...)`. This
is useful for getting everything set up once your HTML document is ready.

[jquery.ready]: http://api.jquery.com/ready/

``` javascript
$(document).ready(function() {
  // Initialize the router.
  App.router = new App.MainRouter;

  // Initialize the main view.
  App.dashboard = new App.Dashboard({ ... });

  // and so on...
});

// Or its shorter form:
$(function() {
  // ...
});
```

A common anti-pattern is to put class definitions (for views, models, and such)
inside these blocks. They are not necessary.

``` javascript
// AVOID this:
$(function() {
  App.PhotoView = Backbone.View.extend({
    ...
  });
});
```

Your classes should be ready before the HTML DOM is. This will save you from
running into problems later where certain classes may not be available at
certain parts of your application.

``` javascript
// Consider instead:
App.PhotoView = Backbone.View.extend({
  ...
});
```

Things outside views
--------------------

Put things in your view class code as much as possible.

Event handlers outside views
----------------------------

Every time you make an event handler outside a view class, consider making a new
view class.

``` javascript
App.PhotoView = Backbone.View.extend({
  ...
});

// AVOID this!
$("a.photo").click(function() { ... });
```

Other links
===========

Other links of interest:

* [Backbone][backbone] official documentation
* [Backbonetutorials.com][bbtutorials] by Thomas Davis covers the basics of
  Backbone.js in more detail than the official docs.
* [Backbone Fundamentals][bbfund] by Addy Osmani is a Creative Commons book for
  beginners and advanced users alike.

[backbone]: http://documentcloud.github.com/backbone/
[bbtutorials]: http://backbonetutorials.com
[bbfund]: https://github.com/addyosmani/backbone-fundamentals

Acknowledgements
================

© 2011-2012, Rico Sta. Cruz. Released under the [MIT
License](http://www.opensource.org/licenses/mit-license.php).

This document is authored and maintained by [Rico Sta. Cruz][rsc] with help from
its [contributors][c]. It is sponsored by my startup, [Sinefunc, Inc][sf].

 * [My website](http://ricostacruz.com) (ricostacruz.com)
 * [Sinefunc, Inc.](http://sinefunc.com) (sinefunc.com)
 * [Github](http://github.com/rstacruz) (@rstacruz)
 * [Twitter](http://twitter.com/rstacruz) (@rstacruz)

[rsc]: http://ricostacruz.com
[c]:   http://github.com/rstacruz/backbone-patterns/contributors
[sf]:  http://sinefunc.com

### To do list

 - Model associations
 - Adding events to subclasses
 - View modes
 - Nested views
 - Router entry/exit
 - View helpers
