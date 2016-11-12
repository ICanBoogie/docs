# Configuring components

The application and its components are configured using a set of files called _configuration
fragments_. These fragments are used to synthesize the configuration of one or more components. When
the same fragments are used to synthesize different configurations, these configurations are
qualified as _derived_. Fragments are synthesized using callback functions called _synthesizers_.
The synthesized configurations can be cached, which cancel the cost of the synthesis. Configurations
are managed by a [Config][] instance.





## Configuration fragments

A configuration fragment is a PHP file returning an array. Multiple fragments are used to synthesize
a configuration, or _derived_ configuration. They are usually located in `config` directories and
are usually named after the config they are used to synthesize.

The following code is an example of a `app` config fragment, which is used to configure the core of
the application:

```php
<?php

namespace ICanBoogie;

return [

	'cache catalogs' => false,
	'cache configs' => false,
	'cache modules' => false,

	'storage_for_configs' => Hooks::class . '::create_storage_for_configs',
	'storage_for_vars' => Hooks::class . '::create_storage_for_vars',

	'error_handler' => Debug::class. '::error_handler',
	'exception_handler' => Debug::class. '::exception_handler',

	'session' => [

		SessionOptions::OPTION_NAME => 'ICanBoogie'

	]

];
```





## Synthesizing a configuration

The `synthesize()` method of the [Config][] instance is used to synthesize a configuration.

The following example demonstrates how a closure can be used to synthesize multiple fragments:

```php
<?php

use function ICanBoogie\array_merge_recursive;

/* @var $config \ICanBoogie\Config */

$app_config = $config->synthesize('app', function(array $fragments) {

	return array_merge_recursive(...$fragments);

});
```





### Magic constructors

The `synthesize()` methods supports the _magic constructors_ `merge` and `recursive merge` which
respectively use the `array_merge()` and `ICanBoogie\array_merge_recursive()` functions. Thus,
the following code examples are equivalent:

```php
<?php

use function ICanBoogie\array_merge_recursive;

/* @var $config \ICanBoogie\Config */

$app_config = $config->synthesize('app', function(array $fragments) {

	return array_merge_recursive(...$fragments);

});
```

```php
<?php

/* @var $config \ICanBoogie\Config */

$app_config = $config->synthesize('app', 'merge recursive');
```





## Configuration synthesizers

Synthesizers can be defined for each configuration, they are used when the config collection is
used as an array:
 
```php
<?php

use ICanBoogie\Config;

/* @var array $paths */

$synthesizers = [ 'app' => 'merge recursive' ];
$config = new Config($paths, $synthesizers);
$app_config = $config['app'];
```





## _Derived_ configurations

It is possible to synthesize a configuration from the same fragments as another configuration,
such a configuration is qualified as _derived_.

For instance, connections, models, and facets, for the [icanboogie/bind-activerecord][] package are
defined in `activerecord` fragments, the synthesizer for the `activerecord_connections`
configuration filters the fragments data to extract what is relevant.

The following example demonstrates how to obtain the `activerecord_connections` configuration using
the `synthesize()` method:

```php
<?php

use ICanBoogie\Binding\ActiveRecord\Hooks;

/* @var $config \ICanBoogie\Config */

$events_config = $config->synthesize(
	'activerecord_connections', 
	Hooks::class . '::synthesize_connections_config',
	'activerecord'
);
```

The following example demonstrates how to define the synthesizer of the `activerecord_connections`
configuration:

```php
<?php

use ICanBoogie\Binding\ActiveRecord\Hooks;
use ICanBoogie\Config;

/* @var $paths array */

$config = new Config($paths, [

	'activerecord_connections' => [ 

		Hooks::class . '::synthesize_connections_config', 'activerecord' 

	]

]);
```

## Declaring synthesizers

Configuration synthesizers are declared using the the `extra` section of the `composer.json` file.

The following example demonstrates how the synthesizers can be declared for an
`activerecord_connections` and an `activerecord_models` configuration. Notice how the `#` sign is
used to separate the callable and the configuration fragment name:

```json
{
	"extra": {
		"icanboogie": {
			"config-constructor": {
				"activerecord_connections": "ICanBoogie\\Binding\\ActiveRecord\\Hooks::synthesize_connections_config#activerecord",
				"activerecord_models": "ICanBoogie\\Binding\\ActiveRecord\\Hooks::synthesize_models_config#activerecord"
			}
		}
	}
}
```






## Caching synthesized configurations

Caching synthesized configurations removes the cost of synthesizing configurations by reusing the
result of a previous synthesis. To enable caching, you just need to provide an instance implementing
[Storage][].

```php
<?php

use ICanBoogie\Config;
use ICanBoogie\Storage\Storage;

/* @var $paths array */
/* @var $synthesizers array */
/* @var Storage $cache */

$config = new Config($paths, $synthesizers, $cache);
```





[icanboogie/bind-activerecord]: https://github.com/ICanBoogie/bind-activerecord
[Config]:                       http://api.icanboogie.org/config/1.1/class-ICanBoogie.Config.html
[Storage]:                      http://api.icanboogie.org/storage/2.0/class-ICanBoogie.Storage.Storage.html
