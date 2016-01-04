# Inline templates

__The problem:__ if you need to use view templates in a small Backbone
application, defining your templates in JavaScript code will be unwieldy and
difficult to maintain.

__Solution:__ You may need some view templates to be inline in the HTML page.

This solution has been outlined by John Resig in his blog post about [JavaScript
micro templating](http://ejohn.org/blog/javascript-micro-templating/).

## Defining inline templates
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

## Using inline templates
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

## Integrating into Backbone

In practice, you will most likely be using this in the `render()` method of a
view like so.

``` javascript
var ContactView = Backbone.View.extend({
  template: _.template($("#template-contact").html()),

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
});
```

## Limitations

__Single-page apps only.__
This assumes that your Backbone application is all contained in one HTML page.
If your app spans across multiple HTML pages, and each page will be needing the
same templates, you may be redundantly streaming the template data to the
browser unnecessarily. Consider using JST templates instead.

Note that the given example assumes that the `#template-contact` element appears
before you include JavaScript files, as it requires the template element to be
accessible before the class is defined.
