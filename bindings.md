# Bindings

Almost all [ICanBoogie packages][] can be used on their own, without involving the framework, but
special packages, usually containing configuration files and additional classes, make them very easy
to use with it. These special packages are called _bindings_ and are prefixed with `bind-`. For
instance, [icanboogie/bind-routing][] binds [icanboogie/routing][] to ICanBoogie.

It's not unusual for these packages to add methods and getters/setters to classes through
[the prototype system][]. The [Application][] instance usually gets the most _bindings_, but is not
the only one; as views are bound to controllers, and active record cache to models. For instance,
the [icanboogie/bind-routing][] package adds the `url_for()` method and the `routes` getter to the
[Application][] instance:

```php
<?php

/* @var $app \ICanBoogie\Application */

$app->url_for('articles:index');
$app->routes->get('/articles', ArticleController::class);
```

These packages always provide corresponding _binding traits_ to help type hinting. The binding
trait from the routing package looks something like this:

```php
<?php

namespace ICanBoogie\Binding\Routing;

use ICanBoogie\Routing\RouteCollection;

/**
 * @method string url_for($route_or_route_id, $values = null)
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

The framework includes the following bindings, but [more are available](https://github.com/ICanBoogie?utf8=%E2%9C%93&q=bind-&type=&language=#org-repositories):

- [icanboogie/bind-event][]
- [icanboogie/bind-http][]
- [icanboogie/bind-prototype][]
- [icanboogie/bind-routing][]





## Prototyped bindings

When the application is instantiated it adds the `app` getter to [Prototyped][] instances, which
allows these instances to obtain the instance of the application.

The following example demonstrates how the application instance can be obtained from a
[Prototyped][] instance. The [PrototypedBindings][] trait may be used to type hint these instances:

```php
<?php

namespace ICanBoogie;

use ICanBoogie\Binding\PrototypedBindings;

/* @var $app Application */
/* @var $prototyped Prototyped|PrototypedBindings */

$prototyped = new Prototyped;

try
{
	$prototyped->app;
}
catch (PropertyNotDefined $e)
{
	// `app` is not defined yet
}

$app = boot();
$app === $prototyped->app;
// true
```





[Application]:                  the-application-class.md
[the prototype system]:         prototypes.md
[icanboogie/bind-event]:        https://github.com/ICanBoogie/bind-event
[icanboogie/bind-prototype]:    https://github.com/ICanBoogie/bind-prototype
[icanboogie/bind-http]:         https://github.com/ICanBoogie/bind-http
[icanboogie/bind-routing]:      https://github.com/ICanBoogie/bind-routing
[icanboogie/routing]:           https://github.com/ICanBoogie/routing
[icanboogie/prototype]:         https://github.com/ICanBoogie/Prototype
[PrototypedBindings]:           https://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Binding.PrototypedBindings.html
[Prototyped]:                   https://api.icanboogie.org/prototype/3.0/class-ICanBoogie.Prototyped.html
[PrototypeTrait]:               https://api.icanboogie.org/prototype/3.0/class-ICanBoogie.PrototypeTrait.html
[ICanBoogie packages]:          https://packagist.org/packages/icanboogie/
