# Partials

__The problem:__ there may be parts of HTML templates that can be reused in many
parts of the application. Defining them more than once is not DRY, which may
make your application less maintainable.

__Solution:__ separating these snippets into partials.

Partials are templates that are meant to be used *inside* other templates.

One typical use of partials is for lists where the template for list items may
be defined as a separate template from the list itself.

## Solution

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
