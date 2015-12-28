# Sub-views

__The problem:__ Your view code is starting to bloat as it tries to do too many
things in one class.

__The solution:__ Break it apart into smaller sub-views.

## The situation

This is a common occurrence if you have one _giant_ view that takes care of the
entire page. View classes may become unwieldy once they get up to 200 lines.

## Solution 1: Sub views

It may be wise to delegate some areas of the view to be the responsibility of
another view.

In this example, we have a view that handles the entire application "chrome."
Let's break apart some of its parts on its `render()` function. Notice that
we're using `this.$()` to select elements inside the `ChromeView`'s element
itself.

``` javascript
App.ChromeView = Backbone.View.extend({
  render: function() {
    // Instantiate some "sub" views to handle the responsibilities of
    // their respective elements.
    this.sidebar = new App.SidebarView({ el: this.$(".sidebar") });
    this.menu = new App.NavigationView({ el: this.$("nav") });
  }
});

$(function() {
  App.chrome = new App.ChromeView({ el: $("#chrome") });
});
```

We will then be able to access the sub-views like so:

``` javascript
App.chrome.sidebar.toggle();

App.chrome.menu.expand();
```

## Events

All Backbone objects can emit [events][events]. To maintain the separation of
responsibilities of the view classes, you may have the sub-views trigger
events that the parent view would need (and vice versa).

[events]: http://documentcloud.github.com/backbone/#Events

For instance, we may implement `SidebarView` to trigger events when the sidebar
is collapsed or expanded:

``` javascript
App.SidebarView = Backbone.View.extend({
  toggle: function() {
    if ($(this.el).is(':visible')) {
      $(this.el).hide();
      this.trigger('collapse');    // <==
    } else {
      $(this.el).show();
      this.trigger('expand');      // <==
    }
  },
});
```

And the parent view (`ChromeView`) may listen to them like so:

``` javascript
App.ChromeView = Backbone.View.extend({
  render: function() {
    this.sidebar = new App.SidebarView({ el: this.$(".sidebar") });

    this.sidebar
      .bind('collapse', this.onSidebarCollapse)
      .bind('expand',   this.onSidebarExpand);

    // ...
  }
});
```
