# HTTP Headers

HTTP headers are represented by a [Headers][] instance. They are used by requests and
responses, and may be used to create the headers string of the `mail()` command as well.





## Content-Type header

The `Content-Type` header is represented by a [ContentType][] instance.

```php
<?php

/* @var $response \ICanBoogie\HTTP\Response */

$response->headers['Content-Type'] = 'text/html; charset=utf-8';

echo $response->headers['Content-Type']->type; // text/html
echo $response->headers['Content-Type']->charset; // utf-8

$response->headers['Content-Type']->type = 'application/xml';

echo $response->headers['Content-Type']; // application/xml; charset=utf-8
```





## Content-Disposition header

The `Content-Disposition` header is represented by a [ContentDisposition][] instance. Of course,
utf-8 file names are supported.

```php
<?php

/* @var $response \ICanBoogie\HTTP\Response */

$response->headers['Content-Disposition'] = 'attachment; filename="été.jpg"';

echo $response->headers['Content-Disposition']->type; // attachment
echo $response->headers['Content-Disposition']->filename; // été.jpg

echo $response->headers['Content-Disposition']; // attachment; filename="ete.jpg"; filename*=UTF-8''%C3%A9t%C3%A9.jpg
```





## Cache-Control header

The `Cache-Control` header is represented by a [CacheControl][] instance. Directives can be set at
once using a plain string, or individually using the properties of the [CacheControl][] instance.
Directives of the [rfc2616](http://www.w3.org/Protocols/rfc2616/rfc2616.html) are supported.

```php
<?php

/* @var $response \ICanBoogie\HTTP\Response */

$response->headers['Cache-Control'] = 'public, max-age=3600, no-transform';

echo $response->headers['Cache-Control']; // public, max-age=3600, no-transform
echo $response->headers['Cache-Control']->cacheable; // public
echo $response->headers['Cache-Control']->max_age; // 3600
echo $response->headers['Cache-Control']->no_transform; // true

$response->headers['Cache-Control']->no_transform = false;
$response->headers['Cache-Control']->max_age = 7200;

echo $response->headers['Cache-Control']; // public, max-age=7200
```





## Date, Expires, If-Modified-Since, If-Unmodified-Since and Retry-After headers

All date related headers can be specified as Unix timestamp, strings or `DateTime` instances.

```php
<?php

use ICanBoogie\HTTP\Response;

$response = new Response('{ "message": "Ok" }', Response::STATUS_OK, [

	'Content-Type' => 'application/json',
	'Date' => 'now',
	'Expires' => '+1 hour'

]);
```




[Headers]:                       http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Headers.html
[ContentType]:                   http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Headers.ContentType.html
[ContentDisposition]:            http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Headers.ContentDisposition.html
[CacheControl]:                  http://api.icanboogie.org/http/2.6/class-ICanBoogie.HTTP.Headers.CacheControl.html
