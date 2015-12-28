# Bootstrapping data

__The problem:__ Your application needs models to be available on page load.

__Solution:__ Bootstrap collections and models by creating collections in an
inline `<script>` block.

This is mentioned in the official Backbone documentation under the [Loading
bootstrapped models][bb.bootstrap] section.

## Define collection data

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

## Accessing instances

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

## In Ruby (ERB)

In your server-side templates, you will probably be using `to_json` on a
collection of your server-side models.

``` html
<script>
  App.photos = new Photos(<%= @photos.to_json %>);
</script>
```

On Rails, you may need to use the `.html_safe` method.

``` html
<script>
  App.photos = new Photos(<%= @photos.to_json.html_safe %>);
</script>
```

## In Ruby (HAML)

If you use HAML, you will need use a syntax similar to this.

``` haml
:javascript
  App.photos = new Photos(#{@photos.to_json});
```

## In PHP

In your server-side templates, you will probably be using `json_encode()` on a
collection of your server-side models.

``` php
<script>
  App.photos = new Photos(<?php echo json_encode($photos); ?>);
</script>
```

## In C# (Razor)

In your Razor view, you may parse your server side models into JSON using `Json.Encode()`.

```
<script>
  App.photos = new Photos(@Html.Raw(Json.Encode(photos)))
</script>
```

[bb.bootstrap]: http://documentcloud.github.com/backbone/#FAQ-bootstrap
