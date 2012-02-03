# [Backbone patterns](http://ricostacruz.com/backbone-patterns)

This is a document with best practices in Backbone. This is a massive
work-in-progress.

This document assumes that you already have some knowledge of [Backbone.js][bb],
[jQuery][jq], and of course, JavaScript itself.

[bb]: http://documentcloud.github.com/backbone/
[jq]: http://jquery.com/

Model patterns
==============
Bootstrapping data
------------------

__The problem:__ Your application needs models to be available on page load.

__Solution:__ Bootstrap collections and models by creating collections in an
inline `<script>` block.

### Define collection data

Define your collection data in an inline script in the HTML file.
If you have collections for `Photos`, you may be doing it this way:

``` html
  <body>
    ...

    <script>
      App.photos = new Photos([
        { id: 2, name: "My dog", filename: "IMG_0392.jpg" },
        { id: 3, name: "Our house", filename: "IMG_0393.jpg" },
        { id: 4, name: "My favorite food", filename: "IMG_0394.jpg" },
        { id: 5, name: "His bag", filename: "IMG_0394.jpg" },
        ...
      ]);
    </script>
  </body>
```

### Accessing instances

To get a single `Photo`, instead of creating a `Photo` instance and using
`fetch()`, simply pluck it from the giant collection.

``` javascript
// Gets by ID
var photo = App.photos.get(2);

// Gets a bunch of photos based on criteria
var photo = App.photos.select(function(photo) {
  return photo.filename.match(/^IMG/);
});
```

### In Ruby (ERB)

In your server-side templates, you will probably be using `to_json` on a
collection of your server-side models.

``` html
<script>
  App.photos = new Photos(<%= @photos.to_json %>);
</script>
```

### In Ruby (HAML)

If you use HAML, you will need use a syntax similar to this.

``` haml
:javascript
  App.photos = new Photos(#{@photos.to_json});
```

### In PHP

In your server-side templates, you will probably be using `json_encode()` on a
collection of your server-side models.

``` php
<script>
  App.photos = new Photos(<?php echo json_encode($photos); ?>);
</script>
```


View patterns
=============

Inline templates
----------------

__The problem:__ if you need to use view templates in a small Backbone
application, defining your templates in JavaScript code will be unwieldy and
difficult to maintain.

__Solution:__ You may need some view templates to be inline in the HTML page.

This solution has been outlined by John Resig in his blog post about [JavaScript
micro templating](http://ejohn.org/blog/javascript-micro-templating/).

### Defining inline templates
You can put templates in an HTML `<script>` tag.

* *Change the `type` attribute* to something else so it will not be interpreted
  as JavaScript.

* *Set an `id`* so we can easily refer to it.

``` html
<script type="text/html" id="template-contact">
  <div class='contact'>
    <strong><%= name %></strong>
    <span><%= email %></span>
  </div>
</script>
```

### Using inline templates
In JavaScript, you can get the `innerHTML` (or jQuery's [.html()][html]) of that
HTML element to fetch the raw template data. You can pass this onto Underscore's
`_.template` to create a template function.

[html]: http://api.jquery.com/html

``` javascript
$("#template-contact").html();
//=> "<div class='contact'>\n<strong><%= name %></str..."

template = _.template($("#template-contact").html());
//=> function() { ... }
```

### Integrating into Backbone

In practice, you will most likely be using this in the `render()` method of a
view like so.

``` javascript
ContactView = Backbone.View.extend({
  template: function() {
    var template = _.template($("#template-contact").html());
    return template.apply(this, arguments);
  },

  render: function() {
    // This is a dictionary object of the attributes of the models.
    // => { name: "Jason", email: "j.smith@gmail.com" }
    var dict = this.model.toJSON();

    // Pass this object onto the template function.
    // This returns an HTML string.
    var html = this.template(dict);

    // Append the result to the view's element.
    $(this.el).append(html);

    // ...
  }
})
```

### Limitations

__Single-page apps only.__
This assumes that your Backbone application is all contained in one HTML page.
If your app spans across multiple HTML pages, and each page will be needing the
same templates, you may be redundantly streaming the template data to the
browser uneededly. Consider using JST templates instead.

JST templates
-------------

__The problem:__ if you need to use view templates in a small-to-large Backbone
application, defining your templates in JavaScript code will be unwieldy and
difficult to maintain.

__Solution:__ You may need put the templates in a JavaScript file.

### The structure

Your app will need to serve a _dynamically-created_ JavaScript file that
compiles your files.

A common JST file will create the `JST` object (in the window namespace), with
each of it's members defined as template functions. In this example, we'll use
Underscore's `_.template`, which returns functions.

``` javascript
// http://myapp.com/javascripts/jst.js
window.JST = {};

window.JST['person/contact'] = _.template(
    "<div class='contact'><%= name %> ..."
);

window.JST['person/edit'] = _.template(
    "<form method='post'><input type..."
);
```

You will then need to link to this JavaScript page in your HTML.

``` html
<script src="http://myapp.com/javascripts/jst.js"></script>
```

### Using JST templates

In your JavaScript code, simply access the JST object's members to access the
views.

``` javascript
var html = JST['person/edit']();

var dict = { name: "Jason", email: "j.smith@gmail.com" };
var html = JST['person/contact'](dict);
```

### Integration notes

* __Rails 3.1 and above__: The Rails Asset pipeline already comes with support
  for JST pages.
* __Rails 3.0 and below__: consider using [Sprockets][sprockets] or
  [Jammit][jammit].
* __In Sinatra__: The [sinatra-backbone][sinatra-backbone] gem can take care of
  dynamically serving JST templates.

[jammit]: http://documentcloud.github.com/jammit
[sprockets]: http://getsprockets.org
[sinatra-backbone]: http://ricostacruz.com/sinatra-backbone

Partials
--------

__The problem:__ there may be parts of HTML templates that can be reused in many
parts of the application. Defining them more than once is not DRY, which may
make your application less maintainable.

__Solution:__ separating these snippets into partials.

Partials are templates that are meant to be used *inside* other templates.

One typical use of partials is for lists where the template for list items may
be defined as a separate template from the list itself.

### Solution

You can pass the template function for the partial as a parameter to the first 
template.

In this example, the function `itemTemplate` is passed onto the parameters for 
`template()`.

``` javascript
TasksList = Backbone.View.extend({
  template: _.template([
    "<ul class='task_list'>",
      "<% items.each(function(item) { %>",
        "<%= itemTemplate(item) %>",
      "<% }); %>",
    "</ul>"
  ].join('')),

  itemTemplate: _.template(
    "<li><%= name %></li>"
  ),

  render: function() {
    var html = this.template({
      items: tasks /* a collection */,
      itemTemplate: this.itemTemplate
    });

    $(this.el).append(html);
  }
});
```

Animation buffer
----------------

__The problem:__ When you have events that trigger animations, they can mess up 
when the user clicks too fast.

__The solution:__ Make a buffering system to ensure that animations are fired
serially (one after the other) and never parallel (at the same time).

### The situation

Let's say you have this innocent code that performs an animation.

One fundamental flaw here is that it assumes that `.next()` will only be called
when it is not animating. When the user clicks "Next" while the animation is
working, unexpected results will occur.

``` javascript
PicturesView = Backbone.View.extend({
  events: {
    'click .next':     'next'
  },

  next: function() {
    var current = this.$(".current");
    var next    = this.$(".current + div");

    if (next.length == 0) { return; }

    // Make the current one move to the left via jQuery.
    // This uses jQuery.fn.animate() that changes CSS values, then fires
    // the function supplied when it's done.
    current.animate({ left: -300, opacity: 0 }, function() {
      current.removeClass('.current');
      next.addClass('.current');
    });
  }
});
```

### The solution

Here's a simple buffering solution. It provides two commands:

 * `add(fn)` which adds a given function to the buffer, and
 * `next()` which moves onto the next command.

To use this, put your animations inside an anonymous function to be passed onto
`add()`. Be sure to trigger `next()` when the animations are done.

``` javascript
Buffer = {
  commands: [],

  // Adds a command to the buffer, and executes it if it's the only command
  // to be ran.
  add: function(fn) {
    this.commands.push(fn);
    if (this.commands.length == 1) fn();
  },

  // Moves onto the next command in the buffer.
  next: function() {
    this.commands.shift();
    if (this.commands.length) this.commands[0]();
  }
};
```

### Example

This is our example from a while ago that has been modified to use the bufferer.

``` javascript
next: function() {
  var current = this.$(".current");
  var next    = this.$(".current + div");

  if (next.length == 0) { return; }

  // Ensure that the animation will not happen while another animation is
  // ongoing.
  Buffer.add(function() {
    current.animate({ left: -300, opacity: 0 }, function() {
      current.removeClass('.current');
      next.addClass('.current');

      // Trigger the next animation.
      Buffer.next();
    });
  });
}
```

Sub views
---------

__The problem:__ Your view code is starting to bloat as it tries to do too many
things in one class.

__The solution:__ Break it apart into smaller sub-views.

### The situation

This is a common occurence if you have one _giant_ view that takes care of the
entire page. View classes may become unwieldy once they get up to 200 lines.

### Solution 1: Sub views

It may be wise to delegate some areas of the view to be the reponsibility of
another view.

In this example, we have a view that handles the entire application "chrome."
Let's break apart some of it's parts on it's `render()` function. Notice that
we're using `this.$()` to select elements inside the `ChromeView`'s element
itself.

``` javascript
App.ChromeView = Backbone.View.extend({
  render: function() {
    // Instanciate some "sub" views to handle the responsibilities of
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

### Events

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

Splitting views
---------------

__The problem:__ Your view code is starting to bloat as it tries to do too many
things in one class, and making sub-views with it's child elements is not an
option.

__The solution:__ Make a sub-view with the same element.

###a Solution

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
  // Notice this view has it's own events. They will not
  // interfere with ChromeView's events.
  events: {
    'click nav.tabs a': 'switchTab'
  },
});
```

Conventions
===========

Naming convention
-----------------

Classes often start in uppercase letters, while instances start with lowercase
letters. This is a throwback of the general Python and Ruby practice of having
constant names start with uppercase letters.

``` javascript
// Classes:
Photo
Album
Author

// Instances:
photo
myAlbum
```

For names with multiple words, JavaScript often calls for CamelCase. Using
underscores are discouraged in JavaScript.

``` javascript
// Good (CamelCase):
PhotoAlbum
albumCover

// Avoid (under_scores):
photo_album
album_cover
```

Namespace convention
--------------------

The convention we use puts everything in one `App` namespace to keep things
organized properly.

``` javascript
window.App = {
    ...
};
```

Subsequent models, views, and other classes will be made in this namespace.

``` javascript
App.Photo = Backbone.Model.extend({
    ...
};
```

Some people prefer to use namespaces based on their app's name. Consider, say,
`BF.Photo` (instead of `App.Photo`) if your application name is "Bacefook."

    Models:                    App.Photo
    Collections:               App.Photos
    Views:                     App.PhotoView
    Main router:               App.Router
    Custom routers:            App.SpecialRouter

    Router instance:           App.router
    View instances:            App.photoView
    Singleton model instances: App.photo
    Collection instances:      App.photos

### Variation: two-level namespace

Some people prefer a verbose two-level version where the classes are divided
into their own namespaces as well.

This is often done to make it easy to iterate over all available models,
     collections, and views.

    Models:                    App.Models.Photo
    Collections:               App.Collections.Photos
    Views:                     App.Views.Photo

RequireJS and AMD
-----------------

You may adopt a [Asynchronous Module Definition][amd]-style method of
organization using a library like [RequireJS][require.js]. This will allow you
to organize your modules in the `require(...)` way familiar to those who use
NodeJS.

If you adopt an AMD library, there will be no need to use namespaces for your
JavaScript classes.

See the [RequireJS][require.js] website for more information on RequireJS and
AMD.

``` javascript
define(function() {
  var Photo = require('models/photo');
  var Photos = require('collections/photos');
  var MenuView = require('views/menu');
  var MainRouter = require('router/main');

  // ...
});
```

[require.js]: http://requirejs.org

File naming
-----------

For applications that do _not_ use [Asynchronous Module Definition][amd]-style
organization, there always seem to have 3 basic JavaScript files.

[amd]: http://requirejs.org/docs/whyamd.html

### The main namespace

This is often `app.js`, which defines the basic namespace.

``` javascript
// app.js
window.App = {
    ...
};
```

### The individual classes

If you use the namespacing method outlined earlier in this document, there are 2
common naming conventions for individual classes:

* Name the files as the exact class name they contain. For instance,
  `App.PhotoView` should be stored as `app/photoview.js`.

* Place each of the class types in their own folders. For instance,
  the `PhotoView` may be defined as `app/views/photoview.js`, or
  `views/photoview.js`.

In this approach, **avoid** putting code in the files other than the actual
class it defines. This makes your convention predictable for the benefit of
those new to your project.

``` javascript
// app/photoview.js
App.PhotoView = Backbone.View.extend({
    ...
});
```

### The setup/glue code file

This is the file where you do miscellaneous things that do not belong in any of
the Backbone classes:

* Instanciate the default view
* Initialize the Backbone Router
* Provide options for jQuery and it's plugins

This is often named `application.js` or `setup.js`.

In larger projects, this can span multiple files. Don't be afraid to refactor it
to multiple files.

This is often the only place you will want to put the onload hook
`$(function() { ... })`.

``` javascript
$(function() {
  // Set up some options for jQuery and plugins.
  $(document).ajaxError(function() {
    alert("There was an error.");
  });

  // Provide options for your plugins.
  $("a[rel~=lightbox]").click(function() {
    $(this).openAsLightbox();
  });

  Backbone.emulateJSON = true;

  // Initialize Backbone views.
  App.chromeView = new App.ChromeView({ el: $("body") });
  App.router = new App.Router;

  // Initialize the Backbone router.
  Backbone.history.start();
});
```

### Load order

Consider loading them in this order:

 * `app.js` (the namespace)
 * `app/*.js` (individual classes)
 * `setup.js` (the glue)

``` html
<script src="javascripts/app.js"></script>
<script src="javascripts/app/photo.js"></script>
<script src="javascripts/app/photoview.js"></script>
<script src="javascripts/app/photos.js"></script>
<script src="javascripts/setup.js"></script>
```

Anti-patterns
=============

Here are a few common practices that I believe should generally be avoided.

$() abuse
---------

jQuery allows you to defer execution of code until when the DOM is fully-loaded
with [$(document).ready(...)][jquery.ready], or it's short form, `$(...)`. This
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

Models should never know about views
------------------------------------

Models should always be independent of backbone views and never reference views.

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

Acknowledgements
================

© 2011-2012, Rico Sta. Cruz. Released under the [MIT
License](http://www.opensource.org/licenses/mit-license.php).

This document is authored and maintained by [Rico Sta. Cruz][rsc] with help from
it's [contributors][c]. It is sponsored by my startup, [Sinefunc, Inc][sf].

 * [My website](http://ricostacruz.com) (ricostacruz.com)
 * [Sinefunc, Inc.](http://sinefunc.com) (sinefunc.com)
 * [Github](http://github.com/rstacruz) (@rstacruz)
 * [Twitter](http://twitter.com/rstacruz) (@rstacruz)

[rsc]: http://ricostacruz.com
[c]:   http://github.com/rstacruz/backbone-patterns/contributors
[sf]:  http://sinefunc.com

### To do list

 - Model associations
 - View modes
 - Nested views
 - Router entry/exit
 - View helpers
