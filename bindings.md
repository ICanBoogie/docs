# Bindings

Almost all ICanBoogie packages can be used without involving the framework, but special packages
usually containing configuration files and additional classes make it very easy to use them with it.
These special packages are called _bindings_ and are usually prefixed with `bind-`. e.g.
[icanboogie/bind-routing][].

More over, the [icanboogie/prototype][] package allows methods of classes using the
[PrototypeTrait][] to be defined at runtime, which includes getters and setters. This unique feature
is used throughout the framework to reverse control over dependencies and maximize lazy loading. The
[Application][] instance has the most _bindings_, but is not the only one; the feature is also used
to bind views to controllers, or active record cache to models.

Packages defining these prototype methods always provide corresponding _binding trait_ that helps
type hinting. For instance, the [icanboogie/bind-routing][] package adds the `routes` getter and the
`url_for()` method to the [Application][] instance.

The binding trait looks something like this:

```php
<?php

namespace ICanBoogie\Binding\Routing;

use ICanBoogie\Routing\RouteCollection;

/**
 * @method string url_for($route_or_route_id, $values = null) Returns the contextualized URL of a route.
 *
 * @property RouteCollection $routes
 */
trait ApplicationBindings
{

}
```

And can be used to type hint the target class as follows:

```php
<?php

namespace ICanBoogie;

class Application extends Core
{
	use Binding\Routing\ApplicationBindings;
}
```

The framework includes the following bindings, but [more are available](https://github.com/ICanBoogie?utf8=%E2%9C%93&query=bind-):

- [icanboogie/bind-prototype][]
- [icanboogie/bind-event][]
- [icanboogie/bind-http][]
- [icanboogie/bind-routing][]





[Application]:                  /docs/4.0/the-application-class
[PrototypeTrait]:               http://api.icanboogie.org/prototype/3.0/class-ICanBoogie.PrototypeTrait.html
[icanboogie/bind-event]:        https://github.com/ICanBoogie/bind-event
[icanboogie/bind-prototype]:    https://github.com/ICanBoogie/bind-prototype
[icanboogie/bind-http]:         https://github.com/ICanBoogie/bind-http
[icanboogie/bind-routing]:      https://github.com/ICanBoogie/bind-routing
[icanboogie/prototype]:         https://github.com/ICanBoogie/Prototype