# Autoconfig

The _Autoconfig_ feature automatically generates a configuration file used to instantiate the
application and configure some of its low-level components. Currently, it is used to define
configuration constructors, paths to components configurations, paths to locale message catalogs,
and paths to modules.





## Participating in the _autoconfig_ process

To participate in the _autoconfig_ process, packages need to define their _autoconfig_ fragment
in the `extra/icanboogie` section of their `composer.json` file. The file must match the
[composer-schema.json][] schema. The following example
demonstrates how an application can specify the path to its configuration and locale messages.

```json
{
	"extra": {
		"icanboogie": {
			"config-path": "path/to/config",
			"locale-path": "path/to/locale"
		}
	}
}
```





## Generating the _autoconfig_ file

The _autoconfig_ file is generated at `vendor/icanboogie/autoconfig.php` after the autoloader is
dumped, during the [`post-autoload-dump`](https://getcomposer.org/doc/articles/scripts.md) event
emitted by [Composer][]. Thus, in order for the _autoconfig_ feature to work, a script for the event
must be declared in the _root_ package of the application:

```json
{
	"scripts": {
		"post-autoload-dump": "ICanBoogie\\Autoconfig\\Hooks::on_autoload_dump"
	}
}
```





## Obtaining the _autoconfig_

The _autoconfig_ is obtained using the `get_autoconfig()` function. The function also updates the
`app-root` and `app-paths` values with the resolved application root and the resolved application
paths respectively.

```php
<?php

namespace ICanBoogie;

$app = boot(get_autoconfig());
```

Additionally, the `ICanBoogie\AUTOCONFIG_PATHNAME` constant defines the absolute pathname to the
_autoconfig_ file.

> **Note:** A fatal error is triggered if the _autoconfig_ file does not exists, which might
happen if the `post-autoload-dump` hook is missing from the `composer.json` file of the application.





## Altering the _autoconfig_ at runtime

Filters defined with the `autoconfig-filters` key are invoked to alter the _autoconfig_ before
the `get_autoconfig()` function returns it. For instance, ICanBoogie uses this feature to add
"config" directories found in the application paths (using the multi-site support).

```json
{
	"extra": {
		"icanboogie": {
			"autoconfig-filters": [ "ICanBoogie\\Autoconfig\\Hooks::filter_autoconfig" ]
		}
	}
}
```





[Composer]:             http://getcomposer.org/
[Application]:          http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Core.html
[composer-schema.json]: https://github.com/ICanBoogie/ICanBoogie/blob/master/lib/Autoconfig/composer-schema.json
