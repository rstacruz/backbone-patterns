# Event handlers outside views

Every time you make an event handler outside a view class, consider making a new
view class.

``` javascript
App.PhotoView = Backbone.View.extend({
  ...
});

// AVOID this!
$("a.photo").click(function() { ... });
```
