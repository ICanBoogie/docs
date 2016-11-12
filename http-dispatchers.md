# HTTP dispatchers

A dispatcher is and instance or closure that given a request returns and response.





## Request dispatcher

A [RequestDispatcher][] instance dispatches requests using a collection of _domain dispatchers_, for
which the _request dispatcher_ provides a nice framework. The _request dispatcher_ sorts _domain
dispatchers_ according to their weight, fire events, and tries to rescue exceptions should they
occur.





### Domain dispatchers

A _domain dispatcher_ handles a very specific type of request. It may be an instance implementing
the [Dispatcher][] interface, or simple callable.

The following example demonstrates how a [RequestDispatcher][] instance may be created with
several _domain dispatchers_:

- `operation`: Defined by the `icanboogie/operation` package, handles operations.
- `routes`: Defined by the `icanboogie/routing` package, handles routes defined using
the `routes` configuration.
- `pages`: Defined by the `icybee/pages` package, handles managed pages.

```php
<?php

use ICanBoogie\HTTP\RequestDispatcher;

$dispatcher = new RequestDispatcher([

	'operation' => \ICanBoogie\Operation\OperationDispatcher::class,
	'route' => \ICanBoogie\Routing\RouteDispatcher::class,
	'page' => \Icybee\Modules\Pages\PageDisptacher::class

]);
```





### Weighted domain dispatchers

The order in which the dispatcher plugins are defined is important because each one of them is
invoked in turn until one returns a response or throws an exception. Some dispatcher plugins
might need to run before others, in that case they need to be defined using a
`WeightedDispatcher` instance.

The weight is defined as an integer; the special values `top` or `bottom`; or a position relative
to a target. Consider the following example:

```php
<?php

use ICanBoogie\HTTP\RequestDispatcher;

$dispatcher = new RequestDispatcher([

	'two' => 'dummy',
	'three' => 'dummy'

]);

$dispatcher['bottom']      = new WeightedDispatcher('dummy', 'bottom');
$dispatcher['megabottom']  = new WeightedDispatcher('dummy', 'bottom');
$dispatcher['hyperbottom'] = new WeightedDispatcher('dummy', 'bottom');
$dispatcher['one']         = new WeightedDispatcher('dummy', 'before:two');
$dispatcher['four']        = new WeightedDispatcher('dummy', 'after:three');
$dispatcher['top']         = new WeightedDispatcher('dummy', 'top');
$dispatcher['megatop']     = new WeightedDispatcher('dummy', 'top');
$dispatcher['hypertop']    = new WeightedDispatcher('dummy', 'top');

$order = '';

foreach ($dispatcher as $dispatcher_id => $dummy)
{
	$order .= ' ' . $dispatcher_id;
}

echo $order; //  hypertop megatop top one two three four bottom megabottom hyperbottom
```

Notice how the `before:` and `after:` prefixes are used to indicate how the dispatcher plugins
should be ordered relatively to the specified targets.





## Dispatcher provider

The `get_dispatcher()` helper is used to retrieve the dispatcher to use to dispatch requests
executed with `$request()`, `$request->send()`, `$request->post()`, … The helper uses
`DispatcherProvider::provide()` to obtain a dispatcher, and if no provider is defined it defines a
new instance of [ProvideDispatcher][] as provider.

The following example demonstrates how you can define your own dispatcher by defining its provider:

```php

use ICanBoogie\HTTP\DispatcherProvider;
use ICanBoogie\HTTP\RequestDispatcher;
use function ICanBoogie\HTTP\get_dispatcher();

// …

DispatcherProvider::define(function() use ($domain_dispatchers) {

	static $dispatcher;

	if (!$dispatcher)
	{
		$dispatcher = new RequestDispatcher($domain_dispatchers);

		new RequestDispatcher\AlterEvent($dispatcher);
	}

	return $dispatcher;

});

get_dispatcher() === DispatcherProvider::provide();   // true
```





### Altering the dispatcher

The `ICanBoogie\HTTP\RequestDispatcher::alter` event of class [RequestDispatcher\AlterEvent][] is
fired after the dispatcher has been created by an instance of [ProvideDispatcher][]. Event hooks may
attach to this event to register or alter domain dispatchers, or replace the request dispatcher
altogether.

The following code illustrate how a `hello` dispatcher, that returns
"Hello world!" when the request matches the path "/hello", can be registered.

```php
<?php

use ICanBoogie\HTTP\RequestDispatcher;
use ICanBoogie\HTTP\Request;
use ICanBoogie\HTTP\Response;

$app->events->attach(function(RequestDispatcher\AlterEvent $event, RequestDispatcher $target) {

	$target['hello'] = function(Request $request) {

		if ($request->path === '/hello')
		{
			return new Response('Hello world!');
		}

	}

});
```





## Dispatching requests

When the _request dispatcher_ is asked to handle a request, it invokes each of its
_domain dispatchers_ in turn until one returns a [Response][] instance or throws an exception.
If an exception is thrown during the dispatch, the _request dispatcher_ tries to _rescue_ it
using either the _domain dispatcher's_ `rescue()` method or the event system. Around that,
events are fired to allow event hooks to alter the request, or alter or replace the response.
If the request could not be resolved into a response, a [NotFound][] exception is thrown,
otherwise the response is returned.

```php
<?php

use ICanBoogie\HTTP\NotFound;
use ICanBoogie\HTTP\Request;

/* @var $dispatcher \ICanBoogie\HTTP\Dispatcher */

$request = Request::from('/path/to/resource.html');

try
{
	$response = $dispatcher($request);
	$response();
}
catch (NotFound $e)
{
	echo $e->getMessage();
}
```





### Before a request is dispatched

The `ICanBoogie\HTTP\RequestDispatcher::dispatch:before` event of class [BeforeDispatchEvent][]
is fired before a request is dispatched.

Event hooks may attach to this event to provide a response to the request before the domain
dispatchers are invoked. If a response is provided the domain dispatchers are skipped.

The event is usually used to redirect requests or provide cached responses. The following code
demonstrates how a request could be redirected if its path is not normalized. For instance a
request for "/index.html" would be redirected to "/".

```php
<?php

use ICanBoogie\HTTP\RequestDispatcher;
use ICanBoogie\HTTP\RedirectResponse;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(RequestDispatcher\BeforeDispatchEvent $event, RequestDispatcher $dispatcher) {

	$path = $event->request->path;
	$normalized_path = $event->request->normalized_path;

	if ($path === $normalized_path)
	{
		return;
	}

	$event->response = new RedirectResponse($normalized_path);
	$event->stop();

});
```

Notice how the `stop()` method of the event is invoked to stop the event propagation and
prevent other event hooks from altering the response.





### After a request was dispatched

The `ICanBoogie\HTTP\RequestDispatcher::dispatch` event of class [DispatchEvent][] is fired after a
request was dispatched, even if no response was provided by domain dispatchers.

Event hooks may attach to this event to alter or replace the response before it is returned by the
dispatcher. The following code demonstrates how a cache could be updated after a response with the
content type "text/html" was found for a request.

```php
<?php

use ICanBoogie\HTTP\RequestDispatcher;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(RequestDispatcher\DispatchEvent $event, RequestDispatcher $target) use($cache) {

	$response = $event->response;

	if ($response->content_type->type !== 'text/html')
	{
		return;
	}

	$cache[sha1($event->request->uri)] = $event->response;

});
```





### Rescuing exceptions

Most likely your application is going to throw exceptions, whether they are caused by software
bugs or logic, you might want to handle them. For example, you might want to present a login form
instead of the default exception message when a [AuthenticationRequired][] exception is thrown.

Exceptions can be rescued at two levels: the domain dispatcher level, using its `rescue()`
method; or the request dispatcher level, by listening to the `Exception::rescue` event.

Event hooks may attach to the `Exception::rescue` event of class [RescueEvent][] to provide a
response for an exception. The following example demonstrates how a login form can be returned as
response when a [AuthenticationRequired][] exception is thrown.

```php
<?php

use ICanBoogie\Event;
use ICanBoogie\HTTP\AuthenticationRequired;
use ICanBoogie\HTTP\Response;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(ICanBoogie\Exception\RescueEvent $event, AuthenticationRequired $target) {

	ICanBoogie\log_error($target->getMessage());

	$event->response = new Response(new DocumentDecorator(new LoginForm), $target->getCode());
	$event->stop();

});
```





#### The `X-ICanBoogie-Rescued-Exception` header field

The `X-ICanBoogie-Rescued-Exception` header field is added to the response obtained while rescuing
an exception, it indicates the origin of the exception, this might help you while tracking bugs.

Note that the origin path of the exception is relative to the `DOCUMENT_ROOT`.





#### Force redirect

If they are not rescued during the `Exception::rescue` event, [ForceRedirect][] exceptions are
resolved into [RedirectResponse][] instances.





### A second chance for `HEAD` requests

When a request with a `HEAD` method fails to get a response (a [NotFound][] exception was
thrown) the dispatcher tries the same request with a `GET` method instead. If a response is
provided a new response is returned with only its status and headers but with an empty body,
otherwise the dispatcher tries to rescue the exception.

Leveraging this feature, you won't have to implement a controller for the `HEAD` method if the
controller for the `GET` method is good enough.





### Stripping the body of responses to `HEAD` requests

The dispatcher cares about responses to `HEAD` requests and will strip responses of their body
before returning them.





## Dispatching a request matching a route

Routes are dispatched by a [RouteDispatcher][] instance, which may be used on its own or
as a _domain dispatcher_ by a [RequestDispatcher][] instance.

```php
<?php

use ICanBoogie\HTTP\Request;
use ICanBoogie\Routing\RouteDefinition;
use ICanBoogie\Routing\RouteDispatcher;
use ICanBoogie\Routing\RouteCollection;

$routes = new RouteCollection([

	'articles:delete' => [

		RouteDefinition::PATTERN => '/articles/<id:\d+>',
		RouteDefinition::CONTROLLER => ArticlesController::class,
		RouteDefinition::ACTION => 'delete',
		RouteDefinition::VIA => Request::METHOD_DELETE

	]

]);

$request = Request::from([

	Request::OPTION_URI => "/articles/123",
	Request::OPTION_IS_DELETE => true

]);

$dispatcher = new RouteDispatcher($routes);
$response = $dispatcher($request);
$response();
```





### Before a route is dispatched

Before a route is dispatched the `ICanBoogie\Routing\RouteDispatcher::dispatch:before` event
of class [RouteDispatcher\BeforeDispatchEvent][] is fired. Event hooks may use this event
to provide a response and thus cancel the dispatching.





### A route is dispatched

The `ICanBoogie\Routing\RouteDispatcher::dispatch` event of class [RouteDispatcher\DispatchEvent][]
is fired if the route has been dispatched successfully. Event hooks may use this event to
alter the response.





### Rescuing a route

If an exception is raised during dispatching, the `ICanBoogie\Routing\Route::rescue` event
of class [Route\RescueEvent][] is fired. Event hooks may use this event to rescue the route and
provide a response, or replace the exception that will be thrown if the rescue fails.





[AuthenticationRequired]:              http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.AuthenticationRequired.html
[BeforeDispatchEvent]:                 http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.RequestDispatcher.BeforeDispatchEvent.html
[DispatchEvent]:                       http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.RequestDispatcher.DispatchEvent.html
[Dispatcher]:                          http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Dispatcher.html
[DispatcherProvider]:                  http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.DispatcherProvider.html
[DispatcherProvider::provide()]:       http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.DispatcherProvider.html#_provide
[ForceRedirect]:                       http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.ForceRedirect.html
[NotFound]:                            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.NotFound.html
[ProvideDispatcher]:                   http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.ProvideDispatcher.html
[RedirectResponse]:                    http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.RedirectResponse.html
[RequestDispatcher]:                   http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.RequestDispatcher.html
[RequestDispatcher\AlterEvent]:        http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.RequestDispatcher.AlterEvent.html
[Response]:                            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Response.html
[RescueEvent]:                         http://api.icanboogie.org/http/2.6/class-ICanBoogie.Exception.RescueEvent.html
[Route\RescueEvent]:                   http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Route.RescueEvent.html
[RouteDispatcher]:                     http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteDispatcher.html
[RouteDispatcher\BeforeDispatchEvent]: http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteDispatcher.BeforeDispatchEvent.html
[RouteDispatcher\DispatchEvent]:       http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.RouteDispatcher.DispatchEvent.html
