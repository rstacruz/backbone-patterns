# JST templates

__The problem:__ if you need to use view templates in a small-to-large Backbone
application, defining your templates in JavaScript code will be unwieldy and
difficult to maintain.

__Solution:__ You may need put the templates in a JavaScript file.

## The structure

Your app will need to serve a _dynamically-created_ JavaScript file that
compiles your files.

A common JST file will create the `JST` object (in the window namespace), with
each of its members defined as template functions. In this example, we'll use
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

## Using JST templates

In your JavaScript code, simply access the JST object's members to access the
views.

``` javascript
var html = JST['person/edit']();

var dict = { name: "Jason", email: "j.smith@gmail.com" };
var html = JST['person/contact'](dict);
```

## Integration notes

* __Rails 3.1 and above__: The Rails Asset pipeline already comes with support
  for JST pages.
* __Rails 3.0 and below__: consider using [Sprockets][sprockets] or
  [Jammit][jammit].
* __In Sinatra__: The [sinatra-backbone][sinatra-backbone] gem can take care of
  dynamically serving JST templates.

[jammit]: http://documentcloud.github.com/jammit
[sprockets]: http://getsprockets.org
[sinatra-backbone]: http://ricostacruz.com/sinatra-backbone
