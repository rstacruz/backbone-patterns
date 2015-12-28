# Namespace convention

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
