# File naming

For applications that do _not_ use [Asynchronous Module Definition][amd]-style
organization, there always seem to be 3 basic JavaScript files.

[amd]: http://requirejs.org/docs/whyamd.html

## The main namespace

This is often `app.js`, which defines the basic namespace.

``` javascript
// app.js
window.App = {
    ...
};
```

## The individual classes

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

## The setup/glue code file

This is the file where you do miscellaneous things that do not belong in any of
the Backbone classes:

* Instantiate the default view
* Initialize the Backbone Router
* Provide options for jQuery and its plugins

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

## Load order

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
