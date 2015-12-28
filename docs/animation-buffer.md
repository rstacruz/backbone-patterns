# Animation buffer

__The problem:__ When you have events that trigger animations, they can mess up
when the user clicks too fast.

__The solution:__ Make a buffering system to ensure that animations are fired
serially (one after the other) and never parallel (at the same time).

## The situation

Let's say you have this innocent code that performs an animation.

One fundamental flaw here is that it assumes that `.showNext()` will only be
called when it is not animating. When the user clicks "Next" while the animation
is working, unexpected results will occur.

``` javascript
PicturesView = Backbone.View.extend({
  events: {
    'click .next': 'showNext'
  },

  showNext: function() {
    var current = this.$(".current");
    var nextDiv = this.$(".current + div");

    if (nextDiv.length == 0) { return; }

    // Make the current one move to the left via jQuery.
    // This uses jQuery.fn.animate() that changes CSS values, then fires
    // the function supplied when it's done.
    current.animate({ left: -300, opacity: 0 }, function() {
      current.removeClass('.current');
      nextDiv.addClass('.current');
    });
  }
});
```

## The solution

Here's a simple buffering solution. It provides two commands:

 * `add(fn)` which adds a given function to the buffer, and
 * `next()` which moves onto the next command. This is passed onto the functions
 when they are called.

To use this, put your animations inside an anonymous function to be passed onto
`add()`. Be sure to trigger `next()` when the animations are done.

``` javascript
Buffer = {
  commands: [],

  add: function(fn) {
    // Adds a command to the buffer, and executes it if it's
    // the only command to be ran.
    var commands = this.commands;
    commands.push(fn);
    if (this.commands.length == 1) fn(next);

    // Moves onto the next command in the buffer.
    function next() {
      commands.shift();
      if (commands.length) commands[0](next);
    }
  }
};
```

## Example

This is our example from a while ago that has been modified to use the bufferer.

``` javascript
showNext: function() {
  var current = this.$(".current");
  var nextDiv = this.$(".current + div");

  if (nextDiv.length == 0) { return; }

  // Ensure that the animation will not happen while another
  // animation is ongoing.
  Buffer.add(function(next) {
    current.animate({ left: -300, opacity: 0 }, function() {
      current.removeClass('.current');
      nextDiv.addClass('.current');

      // Trigger the next animation.
      next();
    });
  });
}
```

## Variations

You can make the `Buffer` object into a class that you can instantiate. This
lets you have multiple buffers as you need. This way, you can have a buffer for
each view instance.

jQuery also provides a very similar function, [jQuery.fn.queue()][queue]. This
may be adequate for most simple animations.

[queue]: http://api.jquery.com/queue/
