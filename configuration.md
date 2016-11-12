# Configuration

ICanBoogie and its components are usually very configurable and come with sensible defaults and a
few conventions. Configurations are usually located in `config` folders, while locale messages are
usually located in `locale` folders. Components configure themselves thanks to ICanBoogie's
_Autoconfig_ feature, and won't require much of you other than a line in the
`composer.json` file of your application.





## Configuring the _core_

The [Core][] instance is configured with _core_ configuration fragments. The fragment used by your
application is usually located in the `/app/all/config/core.php` file.

The following example demonstrates how to enable configs caching and how to specify the name
of the session and its scope.

```php
<?php

// protected/all/config/core.php

return [

	'cache configs' => true,

	'session' => [

		'name' => "ICanBoogie",
		'domain' => ".example.org"

	]

];
```

> **Note:** Check ICanBoogie's `config/core.php` for a list of available options and their default values.





### Specify a storage engine for synthesized configurations

A storage engine for synthesized configurations may be specified with the
`storage_for_configs` option. You may specify a class name or a callable that would return a
[Storage][] instance.

> The default implementation returns a [FileStorage][] instance, or if APC is available a
[StorageCollection][] made of an [APCStorage][] instance and a [FileStorage][] instance.





### Specify a storage engine for variables

A storage engine for variables may be specified with the
`storage_for_variables` option. You may specify a class name or a callable that would return a
[Storage][] instance.

> The default implementation returns a [FileStorage][] instance, or if APC is available a
[StorageCollection][] made of an [APCStorage][] instance and a [FileStorage][] instance.





[Composer]: http://getcomposer.org/
[Core]:                http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Core.html
[APCStorage]:          http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.APCStorage.html
[FileStorage]:         http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.FileStorage.html
[Storage]:             http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.Storage.html
[StorageCollection]:   http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.StorageCollection.html
[composer-schema.json]: https://github.com/ICanBoogie/ICanBoogie/blob/master/lib/Autoconfig/composer-schema.json
