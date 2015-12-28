# Delegate views

__The problem:__ Your view code is starting to bloat as it tries to do too many
things in one class, and making sub-views with its child elements is not an
option.

__The solution:__ Make a sub-view with the same element. This will allow you to
[delegate][delegate] certain responsibilities to another view class.

[delegate]: http://en.wikipedia.org/wiki/Delegation_pattern

## Solution

You can make 2 or more views that target the same element. This is useful when
there are many controls in a view, but creating sub-views (with their scopes
limited to a set of elements in the bigger view) may be too messy, or just not
possible.

In this example, `ChromeView` will make a sub-view that shares the same element
as it does.

``` javascript
App.ChromeView = Backbone.View.extend({
  events: {
    'click button': 'onButtonClick'
  },
  render: function() {
    // Pass our own element to the other view.
    this.tabs = new App.TabView({ el: this.el });
  }
});

App.TabView = Backbone.View.extend({
  // Notice this view has its own events. They will not
  // interfere with ChromeView's events.
  events: {
    'click nav.tabs a': 'switchTab'
  },

  switchTab: function(tab) {
    // ...
  },

  hide: function() {
    // ...
  }
});
```

## Using delegate views

You can delegate some functionality to the sub-view. In this example, we can
write the (potentially long) code for hiding tabs in the `TabView`, making
`ChromeView` easier to maintain and manage.

``` javascript
App.ChromeView = Backbone.View.extend({
  // ...

  goFullscreen: function() {
    this.tabs.hide();
  }
});
```

You may also provide publicly-accessible methods to `TabView` that will be meant
to be accessed outside of `ChromeView`.

``` javascript
var chrome = new App.ChromeView;
chrome.tabs.switchTab('home');
```

## Variation: private delegate views

You can also make delegate views *private* by design: that is, it shouldn't be
used outside the parent view (`ChromeView` in our example).

As JavaScript lacks true private attributes, you can set prefix it with an
underscore to signify that it's private and is not part of it's public
interface. (This is a practice taken from Python's [official style
guide][pep8].)

``` javascript
App.ChromeView = Backbone.View.extend({
  render: function() {
    this._tabs = new App.TabView({ el: this.el });
  }
});
```

[pep8]: http://www.python.org/dev/peps/pep-0008/

