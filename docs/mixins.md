# Mixins

__The problem:__ Sometimes you have the same functionality for multiple objects and it doesn't
make sense to wrap your objects in a parent object. For example, if you have
two views that share methods but don't -- and shouldn't -- have a shared
parent view.

__The solution:__ For this scenario, it's appropriate to use a mixin.

## Defining mixins

You can define an object that has attributes and methods that can be shared
across different classes. This is called a [mixin][mixin].

You can define a mixin as a regular object literal with functions in it.

``` javascript
App.Mixins.Navigation = {

  toggle: function() { /* ... */ },

  open: function() { /*... */ },

  close: function() { /* ... */ }

};
```

## Using mixins

You may then extend your classes with these mixins. You can use Underscore's
[_.extend][extend] function to attach these to your class prototypes.

[mixin]: http://en.wikipedia.org/wiki/Mixin
[extend]: http://documentcloud.github.com/underscore/#extend

``` javascript
App.Views.Menu = Backbone.View.extend({
  // I need to know how to toggle, open, and close!
});

_.extend(App.Views.Menu.prototype, App.Mixins.Navigation);

App.Views.Tabs = Backbone.View.extend({
  // I too need to know how to toggle, open, and close!
});

_.extend(App.Views.Tabs.prototype, App.Mixins.Navigation);

```

## Alternative syntax

The above presents two caveats, which can be problematic in some situations:

  * Your attributes and methods in your mixin will *override* the methods you
  define in the class itself (via `Backbone.View.extend`). Ideally, it should be
  the other way around.

  * The `_.extend(...)` line is after all the methods you've defined in the
  class, and can easily be neglected by developers new to your project.

To remedy this, you can use this alterative syntax. This will let you write
methods and attributes in your class that will override the mixin's default
behavior.

``` javascript
App.Views.Menu = Backbone.View.extend(
  _.extend({}, App.Mixins.Navigation, {

  // (Methods and attributes here)

}));

App.Views.Tabs = Backbone.View.extend(
  _.extend({}, App.Mixins.Navigation, {

  // (Methods and attributes here)

}));
```

## Result

The prototypes for your views now both have the methods defined in your mixin.
New `App.Views.Tabs` and `App.Views.Menu` instances will now be able to respond
to `.toggle()`, `.open()` and `.close()`.

``` javascript
var tabs = new App.Views.Tabs;

// These will call the methods you've defined
// in App.Mixins.Navigation.
tabs.toggle();
tabs.open();
tabs.close();
```

## Models and routers

You can also use mixins in Models and Routers as well.

``` javascript
// Router
App.PageRouter = Backbone.Router.extend(
  _.extend({}, App.Mixins.HasSettings, {

  // (Methods and attributes here)

}));

// Model
App.Widget = Backbone.Model.extend(
  _.extend({}, App.Mixins.IsDeletable, {

  // (Methods and attributes here)

}));
```

