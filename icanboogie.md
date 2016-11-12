# An introduction to ICanBoogie

**ICanBoogie** is a high-performance micro-framework that provides the
essential features to build web applications quickly and easily. It is extensible and a
variety of packages are available to complement its features with [views](https://github.com/icanboogie/view), [routing](https://github.com/icanboogie/routing),
[operations](https://github.com/icanboogie/operation), [internationalization](https://github.com/icanboogie/cldr), [translation](https://github.com/icanboogie/i18n), [ActiveRecord](https://github.com/icanboogie/activerecord), [facets](https://github.com/icanboogie/facets), [mailer](https://github.com/icanboogie/mailer)…





### What does _micro_ mean?

_"micro"_ means that the core features of ICanBoogie are kept to the essential, its core is simple
but greatly extensible. For instance, ICanBoogie won't force an ORM on you, although its
[ActiveRecord](https://github.com/ICanBoogie/ActiveRecord) implementation is pretty nice. In the
same fashion, its routing mechanisms are quite agnostic and let you use your very own
dispatcher if you want to.





### Configuration and conventions

ICanBoogie and its components are usually very configurable and come with sensible defaults and a
few conventions. Configurations are usually located in "config" folders, while locale messages are
usually located in "locale" folders. Components configure themselves thanks to ICanBoogie's
_Autoconfig_ feature, and won't require much of you other than a line in your
`composer.json` file.





## Working with ICanBoogie

ICanBoogie tries to leverage the magic features of PHP as much as possible: getters/setters,
invokable objects, array access, stringifiable objects, closures… to create a coherent framework
which requires less typing and most of all less guessing. As a result, applications created with
ICanBoogie have readable concise code and fluid flow.





### Getters and setters

Magic properties are used in favor of explicit getters and setters such as `getMinute()` and
`setMinute()`. For instance,  [DateTime][] instances provide a `minute` magic property instead of
`getMinute()` and `setMinute()` methods:

```php
<?php

use ICanBoogie\MutableDateTime;

$time = new MutableDateTime('2013-05-17 12:30:45', 'utc');
echo $time;         // 2013-05-17T12:30:45Z
echo $time->minute; // 30

$time->minute += 120;
echo $time;         // 2013-05-17T14:30:45Z
```

The getter/setter feature provided by [icanboogie/accessor][], and extended by
[icanboogie/prototype][], allows you to create read-only or write-only properties, façades
properties, fallbacks to generate default values. Getters are setters are also often used to control
type, lazy load resources, and inversion of control.





### Dependency injection, inversion of control

[icanboogie/prototype][] allows methods to be defined at runtime, and since getters and setters are
methods as well, this feature in often used as a mean to reverse control to provide dependencies.
What's great about it is that dependencies are only provided when they are required, not when to
instance is created.

The following example demonstrates how a shared database connection is obtained through a `db`
property:

```php
<?php

use ICanBoogie\ActiveRecord\Connection;
use ICanBoogie\Prototype;
use ICanBoogie\PrototypeTrait;

/**
 * @property-read Connection $db
 */
class A
{
	use PrototypeTrait;

	public function truncate()
	{
		$this->db("TRUNCATE my_table");
	}
}

Prototype::from(A::class)['get_db'] = function(A $a) {

	static $db;

	return $db ?: $db = new Connection("sqlite::memory:");

};
```





### Objects as strings

If a string represents a serialized set of data ICanBoogie usually provides a class to make its
manipulation easy. Instances can be created from strings, and in turn they can be used as strings.
This applies to dates and times, time zones, time zone locations, HTTP headers, HTTP responses,
database queries, and many more.

```php
<?php

use ICanBoogie\DateTime;

$time = new DateTime('2013-05-17 12:30:45', 'Europe/Paris');

echo $time;                           // 2013-05-17T12:30:45+0200
echo $time->minute;                   // 30
echo $time->zone;                     // Europe/Paris
echo $time->zone->offset;             // 7200
echo $time->zone->location;           // FR,48.86667,2.3333348
echo $time->zone->location->latitude; // 48.86667

use ICanBoogie\HTTP\Headers;

$headers = new Headers;
$headers['Cache-Control'] = 'no-cache';
echo $headers['Cache-Control'];       // no-cache

$headers['Cache-Control']->cacheable = 'public';
$headers['Cache-Control']->no_transform = true;
$headers['Cache-Control']->must_revalidate = false;
$headers['Cache-Control']->max_age = 3600;
echo $headers['Cache-Control'];       // public, max-age=3600, no-transform

use ICanBoogie\HTTP\Response;

$response = new Response('ok', 200);
echo $response;                       // HTTP/1.0 200 OK\r\nDate: Fri, 17 May 2013 15:08:21 GMT\r\n\r\nok

/* @var $app ICanBoogie\Application */

echo $app->models['pages']->own->visible->filter_by_nid(12)->order('created_on DESC')->limit(5);
// SELECT * FROM `pages` `page` INNER JOIN `nodes` `node` USING(`nid`) WHERE (`constructor` = ?) AND (`is_online` = ?) AND (site_id = 0 OR site_id = ?) AND (language = "" OR language = ?) AND (`nid` = ?) ORDER BY created_on DESC LIMIT 5
```





### Invokable objects 

Objects performing a main action are simply invoked to perform that action. For instance, a
prepared database statement is invoked to perform a command:

```php
<?php

# DB statements

/* @var $app ICanBoogie\Application */

$statement = $app->models['nodes']->prepare('UPDATE {self} SET title = ? WHERE nid = ?');
$statement("Title 1", 1);
$statement("Title 2", 2);
$statement("Title 3", 3);
```

This applies to database connections, models, requests, responses, translators… and many more.

```php
<?php

/* @var $app ICanBoogie\Application */

$pages = $app->models['pages'];
$pages('SELECT * FROM {self_and_related} WHERE YEAR(created_on) = 2013')->all;

# HTTP

use ICanBoogie\HTTP\Request;

$request = Request::from($_SERVER);
$response = $request();
$response();

# I18n translator

use ICanBoogie\I18n\Locale;

$translator = Locale::from('fr')->translator;
echo $translator('I can Boogie'); // Je sais danser le Boogie
```





### Collections as arrays

Collections of objects always provide an array interface, whether they are records in the
database, database connections, models, modules, header fields…

```php
<?php

/* @var $app ICanBoogie\Application */

$app->models['nodes'][123];   // fetch record with key 123 in nodes
$app->modules['nodes'];       // obtain the Nodes module
$app->connections['primary']; // obtain the primary database connection

/* @var $request ICanBoogie\HTTP\Request */

$request['param1'];            // fetch param of the request named `param1`, returns `null` if it doesn't exists

/* @var $response ICanBoogie\HTTP\Response */

$response->headers['Cache-Control'] = 'no-cache';
$response->headers['Content-Type'] = 'text/html; charset=utf-8';
```




### Creating an instance from data

Most classes provide a `from()` static method that creates instances from various data types.
This is especially true for sub-classes of the [Prototyped][] class, which can create instances
from arrays of properties. ActiveRecords are a perfect example of this feature:

```php
<?php

use Icybee\Modules\Nodes\Node;

$node = new Node;
$node->uid = 1;
$node->title = "Title";

#or

$node = Node::from([ 'uid' => 1, 'title' => "Title" ]);
```

Some classes don't even provide a public constructor and rely solely on the `from()` method. For
instance, [Request][] instances can only by created using the `from()` method:

```php
<?php

use ICanBoogie\HTTP\Request;

# Creating the initial request from the $_SERVER array

$initial_request = Request::from($_SERVER);

# Creating a local XHR post request with some parameters

$custom_request = Request::from([

	Request::OPTION_URI => '/path/to/controller',
	Request::OPTION_IS_POST => true,
	Request::OPTION_IS_LOCAL => true,
	Request::OPTION_IS_XHR => true,
	Request::OPTION_REQUEST_PARAMS => [

		'param1' => 'value1',
		'param2' => 'value2'

	]

]);
```





## Events

### Cache must be cleared

The `clear_cache` event of class [ClearCacheEvent][] is fired when the various caches of the
application must be cleared. Event hooks may use this event to clear their own cache. For instance,
ICanBoogie clears its configurations cache when this event is fired. 






## Prototype methods

### `ICanBoogie\Prototyped::get_app`

The `app` magic property of [Prototyped][] instances returns the instance of the application. The
property is read-only and is only available after the [Application][] instance has been created.

The [PrototypedBindings][] trait may be used to type hint instances.

```php
<?php

namespace ICanBoogie;

use ICanBoogie\Binding\PrototypedBindings;

/* @var $o Prototyped|PrototypedBindings */

$o = new Prototyped;
$o->app;
// throw ICanBoogie\PropertyNotDefined;

$app = boot();
$app === $o->app;
// true
```





[icanboogie/accessor]:          https://github.com/ICanBoogie/Accessor
[icanboogie/bind-activerecord]: https://github.com/ICanBoogie/bind-activerecord
[icanboogie/bind-cldr]:         https://github.com/ICanBoogie/bind-cldr
[icanboogie/bind-render]:       https://github.com/ICanBoogie/bind-render
[icanboogie/bind-view]:         https://github.com/ICanBoogie/bind-view
[icanboogie/module]:            https://github.com/ICanBoogie/Module
[icanboogie/operation]:         https://github.com/ICanBoogie/Operation
[icanboogie/prototype]:         https://github.com/ICanBoogie/Prototype
[icanboogie/render]:            https://github.com/ICanBoogie/Render
[icanboogie/view]:              https://github.com/ICanBoogie/View
[Prototype package]:            https://github.com/ICanBoogie/Prototype

[Composer]: http://getcomposer.org/

[DateTime]:                   http://api.icanboogie.org/datetime/2.0/class-ICanBoogie.DateTime.html
[TimeZone]:                   http://api.icanboogie.org/datetime/2.0/class-ICanBoogie.TimeZone.html
[Request]:                    http://api.icanboogie.org/http/3.0/class-ICanBoogie.HTTP.Request.html
[BootEvent]:                  http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.BootEvent.html
[ClearCacheEvent]:            http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.ClearCacheEvent.html
[ConfigureEvent]:             http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.ConfigureEvent.html
[Application]:                http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Core.html
[ApplicationNotInstantiated]: http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.ApplicationNotInstantiated.html
[PrototypedBindings]:         http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Binding.PrototypedBindings.html
[RunEvent]:                   http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.RunEvent.html
[TerminateEvent]:             http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.TerminateEvent.html
[Prototyped]:                 http://api.icanboogie.org/prototype/2.3/class-ICanBoogie.Prototyped.html
[APCStorage]:                 http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.APCStorage.html
[FileStorage]:                http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.FileStorage.html
[Storage]:                    http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.Storage.html
[StorageCollection]:          http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.StorageCollection.html
