# Autoconfig

The _Autoconfig_ feature automatically generates a configuration file used to instantiate the
application and configure some of its low-level components. Currently, it defines configuration
constructors, paths to components configurations, paths to locale message catalogs, and paths to
modules.





## Generating the autoconfig file

The autoconfig file is generated at `vendor/icanboogie/autoconfig.php` after the autoloader is
dumped, during the [`post-autoload-dump`](https://getcomposer.org/doc/articles/scripts.md) event
emitted by [Composer][]. In order for the _autoconfig_ feature to work, a script for the event must
be declared in the _root_ package of the application:

```json
{
	"scripts": {
		"post-autoload-dump": "ICanBoogie\\Autoconfig\\Hooks::on_autoload_dump"
	}
}
```





## Obtaining the autoconfig

The autoconfig is obtained using the `get_autoconfig()` function. The function also updates the
`app-root` and `app-paths` values with the resolved application root and the resolved application
paths respectively. Of course, the autoconfig can be used _as is_ to instantiate the application:

```php
<?php

namespace ICanBoogie;

$app = boot(get_autoconfig());
```

Additionally, the `ICanBoogie\AUTOCONFIG_PATHNAME` constant defines the absolute pathname to the
autoconfig file.

> **Note:** A fatal error is triggered if the autoconfig file does not exists, which might happen if
the `post-autoload-dump` hook is missing from the `composer.json` file of the application.





## Altering the autoconfig at runtime

Filters defined with `autoconfig-filters` are invoked to alter the autoconfig before it is returned
by the `get_autoconfig()` helper. For instance, ICanBoogie uses this feature to add `config`
directories found in the application paths (using the [multi-site support][]).

```json
{
	"extra": {
		"icanboogie": {
			"autoconfig-filters": [ "ICanBoogie\\Autoconfig\\Hooks::filter_autoconfig" ]
		}
	}
}
```





## Participating in the autoconfig process

To participate in the autoconfig process, packages need to define their autoconfig fragment in the
`extra/icanboogie` section of their `composer.json` file. The file must match the
[composer-schema.json][] schema.

The following example demonstrates how a package can specify the path to its configuration and
locale messages.

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





[multi-site support]:   ./multi-site.md
[Composer]:             https://getcomposer.org/
[composer-schema.json]: https://github.com/ICanBoogie/ICanBoogie/blob/master/lib/Autoconfig/composer-schema.json
