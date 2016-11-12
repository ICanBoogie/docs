# Introduction to HTTP layer

The **icanboogie/http** package provides an API to handle HTTP requests, with representations for
requests, request files, responses, and headers. A request dispatcher is also provided, that can be
used with your favorite routing solution with very little effort.

The following example demonstrates how you can use a simple closure to create a _Hello world!_
application:

```php
<?php

use ICanBoogie\HTTP\Request;
use ICanBoogie\HTTP\RequestDispatcher;
use ICanBoogie\HTTP\Response;

require 'vendor/autoload.php';

$dispatcher = new RequestDispatcher([

	'hello world' => function(Request $request) {

		$who = $request['name'] ?: 'world';

		return new Response("Hello $who!", Response::STATUS_OK, [

			'Content-Type' => 'text/plain'

		]);

	}

]);

$request = Request::from($_SERVER);
$response = $dispatcher($request);
$response();
```

> **Note:** You might want to check the [icanboogie/routing][] package if you require a nice router.





## Exceptions

The following exceptions are defined by the HTTP package:

* [ClientError][]: thrown when a client error occurs.
	* [NotFound][]: thrown when a resource is not found. For instance, this exception is
	thrown by the dispatcher when it fails to resolve a request into a response.
	* [AuthenticationRequired][]: thrown when the authentication of the client is required. Implements [SecurityError][].
	* [PermissionRequired][]: thrown when the client lacks a required permission. Implements [SecurityError][].
	* [MethodNotSupported][]: thrown when a HTTP method is not supported.
* [ServerError][]: throw when a server error occurs.
	* [ServiceUnavailable][]: thrown when a server is currently unavailable
	(because it is overloaded or down for maintenance).
* [ForceRedirect][]: thrown when a redirect is absolutely required.
* [StatusCodeNotValid][]: thrown when a HTTP status code is not valid.

Exceptions defined by the package implement the `ICanBoogie\HTTP\Exception` interface.
Using this interface one can easily catch HTTP related exceptions:

```php
<?php

try
{
	// …
}
catch (\ICanBoogie\HTTP\Exception $e)
{
	// HTTP exception types
}
catch (\Exception $e)
{
	// Other exception types
}
```





## Helpers

The following helpers are available:

* [`dispatch()`][]: Dispatches a request using the dispatcher returned by [`get_dispatcher()`][].
* [`get_dispatcher()`][]: Returns the request dispatcher. If no dispatcher provider is defined,
the method defines a new instance of [DispatcherProvider][] as provider and use it to retrieve the
dispatcher.
* [`get_initial_request()`][]: Returns the initial request.

```php
<?php

namespace ICanBoogie\HTTP;

$request = get_initial_request();
$response = dispatch($request);
```





[AuthenticationRequired]:        http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.AuthenticationRequired.html
[ClientError]:                   http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.ClientError.html
[DispatcherProvider]:            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.DispatcherProvider.html
[ForceRedirect]:                 http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.ForceRedirect.html
[MethodNotSupported]:            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.MethodNotSupported.html
[NotFound]:                      http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.NotFound.html
[PermissionRequired]:            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.PemissionRequired.html
[SecurityError]:                 http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.SecurityError.html
[ServerError]:                   http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.ServerError.html
[ServiceUnavailable]:            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.ServiceUnavailable.html
[StatusCodeNotValid]:            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.StatusCodeNotValid.html
[`dispatch()`]:                  http://api.icanboogie.org/http/2.6/function-ICanBoogie.HTTP.dispatch.html
[`get_dispatcher()`]:            http://api.icanboogie.org/http/2.6/function-ICanBoogie.HTTP.get_dispatcher.html
[`get_initial_request()`]:       http://api.icanboogie.org/http/2.6/function-ICanBoogie.HTTP.get_initial_request.html

[icanboogie/routing]: https://github.com/ICanBoogie/Routing
