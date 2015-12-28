# document.ready abuse

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
