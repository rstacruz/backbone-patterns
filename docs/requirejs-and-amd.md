# RequireJS and AMD

You may adopt a [Asynchronous Module Definition][amd]-style method of
organization using a library like [RequireJS][require.js]. This will allow you
to organize your modules in the `require(...)` way familiar to those who use
NodeJS.

If you adopt an AMD library, there will be no need to use namespaces for your
JavaScript classes.

``` javascript
define(function(require) {
  var Photo = require('models/photo');
  var Photos = require('collections/photos');
  var MenuView = require('views/menu');
  var MainRouter = require('router/main');

  // ...
});
```

For more information on RequireJS, AMD, and using it on your Backbone project,
see:

 * [Organizing Backbone using Modules][bbt.modules] (via Backbonetutorials.com)
 * [RequireJS][require.js]'s official site

[require.js]: http://requirejs.org
[bbt.modules]: http://backbonetutorials.com/organizing-backbone-using-modules/
[amd]: http://requirejs.org/docs/whyamd.html
