# Backbone patterns

This is a document with best practices in Backbone. This is a massive 
work-in-progress.

Model patterns
==============

Preloading collections
----------------------

__The problem:__ Your application needs models to be available on page load.

__Solution:__ Preload model instances by creating collections on page load.

### Define collections

Define your collections as an inline script in the HTML file.
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

``` ruby
:javascript
  != "App.photos = new Photos(#{@photos.to_json});"
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

Inline views
------------

__The problem:__ if you need to use view templates in a small Backbone 
application, defining your templates in JavaScript code will be unwieldy and 
difficult to maintain.

__Solution:__ You may need some view templates to be inline in the HTML page.

### Defining inline views
You can put views in an HTML `<script>` tag.

* *Change the `type` attribute* to something else so it will not be interpreted 
  as JavaScript.

* *Set an `id`* so we can easily refer to it.

``` html
<script type="text/html" id="foo">
  <div class='contact'>
    <strong><%= name %></strong>
    <span><%= email %></span>
  </div>
</script>
```

### Using inline views
In JavaScript, you can get the `innerText` of that HTML element to fetch the raw 
template data. You can pass this onto Underscore's `_.template` to create a 
template function.

``` javascript
$("#foo").text();
//=> "<div class='contact'>\n<strong><%= name %></str..."

template = _.template($("#foo").text());
//=> function() { ... }
```

### Integrating into Backbone

In practice, you will most likely be using this in the `render()` method of a 
view like so.

``` javascript
ContactView = Backbone.View.extend({
  template: function() {
    var template = _.template($("#foo").text());
    return template.apply(this, arguments);
  },

  render: function() {
    // This is a dictionary object of the attributes of the models.
    // => { name: "Jason", email: "j.smith@gmail.com" }
    var dict = this.model.toJSON();

    // Pass this object onto the template function.
    // This returns an HTML string.
    var html = this.template(hash);

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

You will then need to include this JavaScript in your HTML.

``` html
<script src="http://myapp.com/javascripts/jst.js"></script>
```

### Using JST templates

In your JavaScript code, simply access the JST object's members to access the 
views.

``` javascript
var html = JST['person/edit']();

var dict = { name: "Jason", email: "j.smith@gmail.com };
var html = JST['person/contact'](dict);
```

### Integration notes

* __Rails 3.1__: Sprockets already comes with support for JST pages.
* __Rails 3.0 and below__: consider using Sprockets an Jammit.
* __In Sinatra__: The `sinatra-backbone` gem can take care of dynamically 
  serving JST templates.

Nested views
------------

__The problem:__ You have a lot of views.

Router patterns
===============

Entry and exit
--------------

TODO.

Conventions
===========

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
window.App.Photo = Backbone.Model.extend({
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
Some people prefer a verbose two-level version where the classes are divided ino 
their own namespaces as well.

This is often done to make it easy to iterate over all available models, 
     collections, and views.

    Models:                    App.Models.Photo
    Collections:               App.Collections.Photos
    Views:                     App.Views.Photo

### Variation: Instances in window
Some prefer to have instances in the global namespace. I personally do not 
recommend this.

    Router instance:           App.router
    View instances:            window.photoView
    Singleton model instances: window.photo
    Collection instances:      window.photos

File naming
-----------

Most applications always seem to have 3 basic JavaScript files.

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
popular naming conventions for individual classes:

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

Things NOT to do!

$() abuse
---------

Some people like putting things in jQuery's `$(function() { .. })` to defer 
execution until the DOM is ready.  Don't do it unless you have to.

``` javascript
// AVOID this!
$(function() {
  App.PhotoView = Backbone.View.extend({
    ...
  });
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

Acknowledgements
================


© 2011, Rico Sta. Cruz. Released under the [MIT 
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
