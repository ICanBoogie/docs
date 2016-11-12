# Routing

The [icanboogie/routing][] package handles URL rewriting in native PHP. A request is mapped
to a route, which in turn gets dispatched to a controller, and possibly an action. If the
process is successful a response is returned. Events are fired during the process to allow
event hooks to alter the request, the route, the controller, or the response.





## Route definitions

A route definition is an array, which may be created with the following keys:

- `RouteDefinition::PATTERN`: The pattern of the URL.
- `RouteDefinition::CONTROLLER`: The controller class and optional action, or a callable.
- `RouteDefinition::ID`: The identifier of the route.
- `RouteDefinition::VIA`: If the route needs to respond to one or more HTTP methods, e.g.
`Request::METHOD_GET` or `[ Request::METHOD_PUT, Request::METHOD_PATCH ]`.
Defaults: `Request::METHOD_GET`.
- `RouteDefinition::LOCATION`: To redirect the route to another location.
- `RouteDefinition::CONSTRUCTOR`: If the route should be instantiated from a class other than [Route][].

A route definition is considered valid when the `RouteDefinition::PATTERN` parameter is defined along one of
`RouteDefinition::CONTROLLER` or `RouteDefinition::LOCATION`. [PatternNotDefined][] is thrown if `RouteDefinition::PATTERN` is missing, and
[ControllerNotDefined][] is thrown if both `RouteDefinition::CONTROLLER` and `RouteDefinition::LOCATION` are missing.

> **Note:** You can add any parameter you want to the route definition, they are used to create
the route instance, which might be useful to provide additional information to a controller.
Better use a custom route class though.





### Route patterns

A pattern is used to match a URL with a route. Placeholders may be used to match multiple URL to a
single route and extract its parameters. Three types of placeholder are available:

- Relaxed placeholder: Only the name of the parameter is specified, it matches anything until
the following part. e.g. `/articles/:id/edit` where `:id` is the placeholder for
the `id` parameter.
 
- Constrained placeholder: A regular expression is used to match the parameter value.
e.g. `/articles/<id:\d+>/edit` where `<id:\d+>` is the placeholder for the `id` parameter
which value must match `/^\d+$/`.

- Anonymous constrained placeholder: Same as the constrained placeholder, except the parameter
has no name but an index e.g. `/articles/<\d+>/edit` where `<\d+>` in a placeholder
which index is 0.

Additionally, the joker character `*`—which can only be used at the end of a pattern—matches
anything. e.g. `/articles/123*` matches `/articles/123` and `/articles/123456` as well.

Finally, constraints RegEx are extended with the following:

- `{:sha1:}`: Matches [SHA-1](https://en.wikipedia.org/wiki/SHA-1) hashes. e.g. `/files/<hash:{:sha1:}>`.
- `{:uuid:}`: Matches [Universally unique identifiers](https://en.wikipedia.org/wiki/Universally_unique_identifier)
(UUID). e.g. `/articles/<uuid:{:uuid:}>/edit`.

You can use them in any combination:

- `/blog/:year-:month-:slug`
- `/blog/<year:\d{4}>-<month:\d{2}>-:slug`
- `/images/<uuid:{:uuid:}>/<size:\d+x|x\d+|\d+x\d+>*`





### Route controller

The `controller` key specifies the callable to invoke, or the class name of a callable.
The following value types are accepted:

- A controller class: `ArticlesShowController`
- A controller action: `ArticlesController#show`, where `ArticlesController` is
the controller class, and `show` is the action.
- A callable: `function() {}`, `new ArticlesShowController`, `ArticlesController::show`,
`articles_controller_show`, …





## Route collections

A [RouteCollection][] instance holds route definitions and is used to create [Route][] instances.
A route dispatcher uses an instance to map a request to a route. A route collection is usually
created with an array of route definitions, which may come from configuration fragments,
[RouteMaker][], or an expertly crafted array. After the route collection is created it may be
modified by using the collection as a array, or by adding routes using one of
the supported HTTP methods. Finally, a collection may be created from another using
the `filter()` method.





### Defining routes using configuration fragments

If the package is bound to [ICanBoogie][] using [icanboogie/bind-routing][], routes can be defined
using `routes` configuration fragments. Refer to [icanboogie/bind-routing][] documentation to
learn more about this feature.

```php
<?php

use ICanBoogie\Routing\RouteCollection;

// …

$routes = new RouteCollection($app->configs['routes']);
# or
$routes = $app->routes;
```





### Defining routes using offsets

Used as an array, routes can be defined by setting/unsetting the offsets of a [RouteCollection][].

```php
<?php

use ICanBoogie\HTTP\Request;
use ICanBoogie\Routing\RouteCollection;
use ICanBoogie\Routing\RouteDefinition;

$routes = new RouteCollection;

$routes['articles:index'] = [

	RouteDefinition::PATTERN => '/articles',
	RouteDefinition::CONTROLLER => ArticlesController::class,
	RouteDefinition::ACTION => 'index',
	RouteDefinition::VIA => Request::METHOD_GET

];

unset($routes['articles:index']);
```





### Defining routes using HTTP methods

Routes may be defined using HTTP methods, such as `get` or `delete`.

```php
<?php

use ICanBoogie\HTTP\Request;
use ICanBoogie\Routing\RouteCollection;
use ICanBoogie\Routing\RouteDefinition;

$routes = new RouteCollection;
$routes->any('/', function(Request $request) { }, [ RouteDefinition::ID => 'home' ]);
$routes->any('/articles', function(Request $request) { }, [ RouteDefinition::ID => 'articles:index' ]);
$routes->get('/articles/new', function(Request $request) { }, [ RouteDefinition::ID => 'articles:new' ]);
$routes->post('/articles', function(Request $request) { }, [ RouteDefinition::ID => 'articles:create' ]);
$routes->delete('/articles/<nid:\d+>', function(Request $request) { }, [ RouteDefinition::ID => 'articles:delete' ]);
```





### Filtering a route collection

Sometimes you want to work with a subset of a route collection, for instance the routes related to
the admin area of a website. The `filter()` method filters routes using a callable filter and
returns a new [RouteCollection][].

The following example demonstrates how to filter _index_ routes in an "admin" namespace.
You can provide a closure, but it's best to create filter classes that you can extend and reuse:

```php
<?php

class AdminIndexRouteFilter
{
	/**
	 * @param array $definition A route definition.
	 * @param string $id A route identifier.
	 * 
	 * @return bool
	 */
	public function __invoke(array $definition, $id)
	{
	    return strpos($id, 'admin:') === 0 && !preg_match('/:index$/', $id);
	}
}

$filtered_routes = $routes->filter(new AdminIndexRouteFilter);
```





## Mapping a path to a route

Routes are mapped using a [RouteCollection][] instance. A HTTP method and a namespace can optionally
be specified to determine the route more accurately. The parameters captured from the routes are
stored in the `$captured` variable, passed by reference. If the path contains a query string,
it is parsed and stored under `__query__` in `$captured`.

```php
<?php

use ICanBoogie\HTTP\Request;

$home_route = $routes->find('/?singer=madonna', $captured);
var_dump($captured);   // [ '__query__' => [ 'singer' => 'madonna' ] ]

$articles_delete_route = $routes->find('/articles/123', $captured, Request::METHOD_DELETE);
var_dump($captured);   // [ 'nid' => 123 ]
```





## Route

A route is represented by a [Route][] instance. It is usually created from a definition array
and contains all the properties of its definition.

```php
<?php

$route = $routes['articles:show'];
echo get_class($route); // ICanBoogie\Routing\Route;
```

A route can be formatted into a relative URL using its `format()` method and appropriate
formatting parameters. The method returns a [FormattedRoute][] instance, which can be used as
a string. The following properties are available:

- `url`: The URL contextualized with `contextualize()`.
- `absolute_url`: The contextualized URL _absolutized_ with the `absolute_url()` function.

```php
<?php

$route = $routes['articles:show'];
echo $route->pattern;      // /articles/:year-:month-:slug.html

$url = $route->format([ 'year' => '2014', 'month' => '06', 'slug' => 'madonna-queen-of-pop' ]);
echo $url;                 // /articles/2014-06-madonna-queen-of-pop.html
echo get_class($url);      // ICanBoogie\Routing\FormattedRoute
echo $url->absolute_url;   // http://icanboogie.org/articles/2014-06-madonna-queen-of-pop.html

$url->route === $route;    // true
```

You can format a route using a record, or any other object, as well:

```php
<?php

$record = $app->models['articles']->one;
$url = $routes['articles:show']->format($record);
```





### Assigning a formatting value to a route

The `assign()` method is used to assign a formatting value to a route. It returns an updated
clone of the route which can be formatted without requiring a formatting value. This is very
helpful when you need to pass around an instance of a route that is ready to be formatted.

The following example demonstrates how the `assign()` method can be used to assign a formatting
value to a route, that can later be used like a URL string:

```php
<?php

use ICanBoogie\Routing\RouteCollection;
use ICanBoogie\Routing\RouteDefinition;

$routes = new RouteCollection([

	'article:show' => [

		RouteDefinition::PATTERN => '/articles/<year:\d{4}>-<month:\d{2}>.html',
		RouteDefinition::CONTROLLER => ArticlesController::class,
		RouteDefinition::ACTION => 'show'

	]

]);

$route = $routes['article:show']->assign([ 'year' => 2015, 'month' => '02' ]);
$routes['article:show'] === $routes['article:show'];   // true
$route === $routes['article:show'];                    // false
$route->formatting_value;                              // [ 'year' => 2015, 'month' => 02 ]
$route->has_formatting_value;                          // true

echo $route;
// /articles/2015-02.html
echo $route->absolute_url;
// http://icanboogie.org/articles/2015-02.html
echo $route->format([ 'year' => 2016, 'month' => 10 ]);
// /articles/2016-10.html
```

> **Note:** Assigning a formatting value to an _assigned_ route creates another instance of the
route. Also, the formatting value is reset when an _assigned_ route is cloned.

Whether a route has an assigned formatting value or not, the `format()` method still requires
a formatting value, it does *not* use the assign formatting value. Thus, if you want to format
a route with its assigned formatting value, use the `formatting_value` property:

```php
<?php

echo $route->format($route->formatting_value);
```





## Exceptions

The exceptions defined by the package implement the `ICanBoogie\Routing\Exception` interface,
so that they are easy to recognize:

```php
<?php

try
{
	// …
}
catch (\ICanBoogie\Routing\Exception $e)
{
	// a routing exception
}
catch (\Exception $e)
{
	// another type of exception
}
```

The following exceptions are defined:

- [ActionNotDefined][]: Thrown when an action is not defined, for instance when a route handled
by a controller using [ActionTrait][] has an empty `action` property.
- [ControllerNotDefined][]: Thrown when trying to define a route without a controller nor location.
- [PatternNotDefined][]: Thrown when trying to define a route without pattern.
- [RouteNotDefined][]: Thrown when trying to obtain a route that is not defined in a
[RouteCollection][] instance.





## Helpers

The following helpers are available:

- [contextualize](http://api.icanboogie.org/routing/latest/function-ICanBoogie.Routing.contextualize.html): Contextualize a pathname.
- [decontextualize](http://api.icanboogie.org/routing/latest/function-ICanBoogie.Routing.decontextualize.html): Decontextualize a pathname.
- [absolutize_url](http://api.icanboogie.org/routing/latest/function-ICanBoogie.Routing.absolutize_url.html): Absolutize an URL.





### Patching helpers

Helpers can be patched using the `Helpers::patch()` method.

The following code demonstrates how routes can _start_ with the custom path "/my/application":

```php
<?php

use ICanBoogie\Routing;

$path = "/my/application";

Routing\Helpers::patch('contextualize', function($str) use($path) {

	return $path . $str;

});

Routing\Helpers::patch('decontextualize', function($str) use($path) {

	if (strpos($str, $path . '/') === 0)
	{
		$str = substr($str, strlen($path));
	}

	return $str;

});
```





[ControllerBindings]:                  http://api.icanboogie.org/bind-routing/0.2/class-ICanBoogie.Binding.Routing.ControllerBindings.html
[Response]:                            http://api.icanboogie.org/http/3.0/class-ICanBoogie.HTTP.Response.html
[Request]:                             http://api.icanboogie.org/http/3.0/class-ICanBoogie.HTTP.Request.html
[RequestDispatcher]:                   http://api.icanboogie.org/http/3.0/class-ICanBoogie.HTTP.RequestDispatcher.html
[documentation]:                       http://api.icanboogie.org/routing/4.0/
[ActionNotDefined]:                    http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.ActionNotDefined.html
[ActionTrait]:                         http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.ActionTrait.html
[Controller]:                          http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.html
[ControllerNotDefined]:                http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.ControllerNotDefined.html
[FormattedRoute]:                      http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.FormattedRoute.html
[Pattern]:                             http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Pattern.html
[PatternNotDefined]:                   http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.PatternNotDefined.html
[ResourceTrait]:                       http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.ResourceTrait.html
[Route]:                               http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Route.html
[Route\RescueEvent]:                   http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Route.RescueEvent.html
[RouteCollection]:                     http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteCollection.html
[RouteDispatcher]:                     http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteDispatcher.html
[RouteDispatcher\BeforeDispatchEvent]: http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteDispatcher.BeforeDispatchEvent.html
[RouteDispatcher\DispatchEvent]:       http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteDispatcher.DispatchEvent.html
[RouteMaker]:                          http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteMaker.html
[RouteNotDefined]:                     http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteNotDefined.html
[ICanBoogie]:                          https://github.com/ICanBoogie/ICanBoogie
[icanboogie/bind-routing]:             https://github.com/ICanBoogie/bind-routing
[icanboogie/routing]:                  https://github.com/ICanBoogie/routing
