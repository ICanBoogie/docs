# Request

A request is represented by a [Request][] instance. The initial request is usually created from
the `$_SERVER` array, while sub requests are created from arrays of `Request::OPTION_*` or 
`RequestOptions::OPTION_*` options.

```php
<?php

use ICanBoogie\HTTP\Request;

$initial_request = Request::from($_SERVER);

# a custom request in the same environment

$request = Request::from('path/to/file.html', $_SERVER);

# a request created from scratch

$request = Request::from([

	Request::OPTION_PATH => 'path/to/file.html',
	Request::OPTION_IS_LOCAL => true,            // or OPTION_IP => '::1'
	Request::OPTION_IS_POST => true,             // or OPTION_METHOD => Request::METHOD_POST
	Request::OPTION_HEADERS => [

		'Cache-Control' => 'no-cache'

	]

]);
```





## Safe and idempotent requests

Safe methods are HTTP methods that do not modify resources. For instance, using `GET` or `HEAD` on a
resource URL, should NEVER change the resource.

The `is_safe` property may be used to check if a request is safe or not.

```php
<?php

use ICanBoogie\HTTP\Request;

Request::from([ Request::OPTION_METHOD => Request::METHOD_GET ])->is_safe; // true
Request::from([ Request::OPTION_METHOD => Request::METHOD_POST ])->is_safe; // false
Request::from([ Request::OPTION_METHOD => Request::METHOD_DELETE ])->is_safe; // false
```

An idempotent HTTP method is a HTTP method that can be called many times without different outcomes.
It would not matter if the method is called only once, or ten times over. The result should be the
same.

The `is_idempotent` property may be used to check if a request is idempotent or not.

```php
<?php

use ICanBoogie\HTTP\Request;

Request::from([ Request::OPTION_METHOD => Request::METHOD_GET ])->is_idempotent; // true
Request::from([ Request::OPTION_METHOD => Request::METHOD_POST ])->is_idempotent; // false
Request::from([ Request::OPTION_METHOD => Request::METHOD_DELETE ])->is_idempotent; // true
```





## A request with changed properties

Requests are for the most part immutable, the `with()` method creates an instance copy with
changed properties.

```php
<?php

use ICanBoogie\HTTP\Request;

$request = Request::from($_SERVER)->with([

	Request::OPTION_IS_HEAD => true,
	Request::OPTION_IS_XHR => true

]);
```




## Request parameters

Whether they are sent as part of the query string, the post body, or the path info, parameters
sent along a request are collected in arrays. The `query_params`, `request_params`,
and `path_params` properties give you access to these parameters.

You can access each type of parameter as follows:

```php
<?php

/* @var $request \ICanBoogie\HTTP\Request */

$id = $request->query_params['id'];
$method = $request->request_params['method'];
$info = $request->path_params['info'];
```

All the request parameters are also available through the `params` property, which merges the
_query_, _request_ and _path_ parameters:

```php
<?php

/* @var $request \ICanBoogie\HTTP\Request */

$id = $request->params['id'];
$method = $request->params['method'];
$info = $request->params['info'];
```

Used as an array, the [Request][] instance provides these parameters as well, but returns `null`
when a parameter is not defined:

```php
<?php

/* @var $request \ICanBoogie\HTTP\Request */

$id = $request['id'];
$method = $request['method'];
$info = $request['info'];

var_dump($request['undefined']); // null
```

Of course, the request is also an iterator:

```php
<?php

/* @var $request \ICanBoogie\HTTP\Request */

foreach ($request as $parameter => $value)
{
	echo "$parameter: $value\n";
}
```





## Request files

Files associated with a request are collected in a [FileList][] instance. The initial request
created with `$_SERVER` obtain its files from `$_FILES`. For custom requests, files are defined
using `OPTION_FILES`.

```php
<?php

use ICanBoogie\HTTP\FileOptions as File;
use ICanBoogie\HTTP\Request;

$request = Request::from($_SERVER);

# or

$request = Request::from([

	Request::OPTION_FILES => [

		'uploaded' => [ File::OPTION_PATHNAME => '/path/to/my/example.zip' ]

	]

]);

#

$files = $request->files;    // instanceof FileList
$file = $files['uploaded'];  // instanceof File
$file = $files['undefined']; // null
```

Uploaded files, and _pretend_ uploaded files, are represented by [File][] instances. The class
tries its best to provide the same API for both. The `is_uploaded` property helps you set
them apart.

The `is_valid` property is a simple way to check if a file is valid. The `move()` method
let's you move the file out of the temporary folder or around the filesystem.

```php
<?php

use ICanBoogie\HTTP\File;

/* @var $file File */

echo $file->name;            // example.zip
echo $file->unsuffixed_name; // example
echo $file->extension;       // .zip
echo $file->size;            // 1234
echo $file->type;            // application/zip
echo $file->is_uploaded;     // false

if ($file->is_valid)
{
	$file->move('/path/to/repository/' . $file->name, File::MOVE_OVERWRITE);
}
```

The `match()` method is used to check if a file matches a MIME type, a MIME class, or a file
extension:

```php
<?php

/* @var $file \ICanBoogie\HTTP\File */

echo $file->match('application/zip');             // true
echo $file->match('application');                 // true
echo $file->match('.zip');                        // true
echo $file->match('image/png');                   // false
echo $file->match('image');                       // false
echo $file->match('.png');                        // false
```

The method also handles sets, and returns `true` if there's any match:

```php
echo $file->match([ '.png', 'application/zip' ]); // true
echo $file->match([ '.png', '.zip' ]);            // true
echo $file->match([ 'image/png', '.zip' ]);       // true
echo $file->match([ 'image/png', 'text/plain' ]); // false
```

[File][] instances implement the [ToArray][] interface and can be converted into arrays
with the `to_array()` method:

```php
$file->to_array();
/*
[
	'name' => 'example.zip',
	'unsuffixed_name' => 'example',
	'extension' => '.zip',
	'type' => 'application/zip',
	'size' => 1234,
	'pathname' => '/path/to/my/example.zip',
	'error' => null,
	'error_message' => null
]
*/
```





## Request context

Because requests may be nested the request context offers a safe place where you can store the state
of your application that is relative to a request, for instance a request relative site, page,
route, dispatcher… The context may be used as an array, but is also a prototyped instance.

The following example demonstrates how to store a value in a request context:

```php
<?php

use ICanBoogie\HTTP\Request;

$request = Request::from($_SERVER);
$request->context['site'] = $app->models['sites']->one;
```

The following example demonstrates how to use the prototype feature to provide a value when it is
requested from the context:

```php
<?php

use ICanBoogie\HTTP\Request;
use ICanBoogie\HTTP\Request\Context;
use ICanBoogie\Prototype;

Prototype::from(Context::class)['lazy_get_site'] = function(Context $context) use ($site_model) {

	return $site_model->resolve_from_request($context->request);

};

$request = Request::from($_SERVER);

$site = $request->context['site'];
# or
$site = $request->context->site;
```





## Obtaining a response

A response is obtained from a request simply by invoking the request, or by invoking one of the
available HTTP methods. Internally, the [`dispatch()`][] helper is used to dispatch the request. A
[Response][] instance is returned if the dispatching is successful, a [NotFound][] exception is
thrown otherwise.

```php
<?php

/* @var $request \ICanBoogie\HTTP\Request */

$response = $request();

# using the POST method and additional parameters

$response = $request->post([ 'param' => 'value' ]);
```





[Request]:                       http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Request.html
[File]:                          http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.File.html
[FileList]:                      http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.FileList.html
[ToArray]:                       http://api.icanboogie.org/common/1.2/class-ICanBoogie.ToArray.html
[NotFound]:                      http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.NotFound.html
[Response]:                      http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Response.html
[`dispatch()`]:                  http://api.icanboogie.org/http/2.6/function-ICanBoogie.HTTP.dispatch.html
