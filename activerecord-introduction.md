# An introduction to ActiveRecord

__Connections__, __models__ and __active records__ are the foundations of everything that concerns
database access and management. They are used to establish database connections, manage tables and
their possible relationship, as well as manage the records of these tables. Leveraging OOP, the
models and active records are instances which properties, getters/setters and behavior can be
inherited in a business logic.

Using the __query interface__, you won't have to write raw SQL, manage table relationship,
or worry about injection.

Finally, using __providers__ you can define all your connections and models in a single place.
Connections are established and models are instantiated on demand, so feel free the define
hundreds of them.





## Bindings

The [icanboogie/bind-activerecord][] package binds the [icanboogie/activerecord][] package to
ICanBoogie, using its _Autoconfig_ feature. It provides configuration synthesizers for connections
and models, as well as getters for connection collection and model collection. A model provider
is also declared:

```php
<?php
namespace ICanBoogie;

$app = boot();

$connections_config = $app->configs['activerecord_connections'];
$models_config = $app->configs['activerecord_models'];

echo get_class($app->connections);               // ICanBoogie\ActiveRecord\ConnectionCollection
echo get_class($app->models);                    // ICanBoogie\ActiveRecord\ModelCollection

$primary_connection = $app->connections['primary'];
$primary_connection === $app->db;                // true

get_models('nodes') === $app->models['nodes'];   // true
```





### Autoconfig

ICanBoogie's _Autoconfig_ is used to provide the following features:

- A synthesizer for the `activerecord_connections` config, created from
the `activerecord#connections` fragments.
- A synthesizer for the `activerecord_models` config, created from
the `activerecord#models` fragments.
- A lazy getter for the `ICanBoogie\Application::$connections` property, that returns
a [ConnectionCollection][] instance created with the `activerecord_connections` config.
- A lazy getter for the `ICanBoogie\Application::$models` property, that returns
a [ModelCollection][] instance created with the `activerecord_models` config.
- A lazy getter for the `ICanBoogie\Application::$db` property, that returns the connection named
`primary` from the `ICanBoogie\Application::$connections` property.





### The `activerecord` config fragment

Currently `activerecord` fragments are used to synthesize `activerecord_connections` and
`activerecord_models` configurations, which are suitable to create [ConnectionCollection][] and
[ModelCollection][] instances.

The following example demonstrates how to define connections and models. Two connections
are defined: `primary` is a connection to the MySQL server;`cache` is a connection to a SQLite
database. The `nodes` model is also defined.

```php
<?php

// config/activerecord.php

use ICanBoogie\ActiveRecord\ConnectionOptions;
use ICanBoogie\ActiveRecord\Model;

return [

	'connections' => [

		'primary' => [

			'dsn' => 'mysql:dbname=mydatabase',
			'username' => 'root',
			'password' => 'root',
			'options' => [

				ConnectionOptions::TIMEZONE => '+02:00',
				ConnectionOptions::TABLE_NAME_PREFIX => 'myprefix'

			]
		],

		'cache' => 'sqlite:' . ICanBoogie\REPOSITORY . 'cache.sqlite'

	],

	'models' => [

		'nodes' => [

			Model::SCHEMA => [

				'id' => 'serial',
				'title' => 'varchar'

			]
		]
	]
];
```





## Exceptions

ActiveRecord exceptions implement the `ICanBoogie\ActiveRecord\Exception` interface so that they can
easily be identified:

```php
<?php

try
{
	// …
}
catch (\ICanBoogie\ActiveRecord\Exception $e)
{
	// an ActiveRecord exception
}
catch (\Exception $e)
{
	// some other exception
}
```

The following exceptions are defined:

- [ConnectionAlreadyEstablished][]: Exception thrown in attempt to set/unset the definition of an
already established connection.
- [ConnectionNotDefined][]: Exception thrown in attempt to obtain a connection that is not defined.
- [ConnectionNotEstablished][]: Exception thrown when a connection cannot be established.
- [ModelAlreadyInstantiated][]: Exception thrown in attempt to set/unset the definition of an
already instantiated model.
- [ModelNotDefined][]: Exception thrown in attempt to obtain a model that is not defined.
- [RecordNotFound][]: Exception thrown when one or several records cannot be found.
- [RelationNotDefined][]: Exception thrown in attempt to obtain a relation that is not defined.
- [ScopeNotDefined][]: Exception thrown in attempt to obtain a scope that is not defined.
- [StatementInvocationFailed][]: Exception thrown when invoking a statement fails (`execute()` returned `false`).
- [StatementNotValid][]: Exception thrown in attempt to execute a statement that is not valid.
- [UnableToSetFetchMode][]: Exception thrown when the fetch mode of a statement fails to be set.





[ConnectionAlreadyEstablished]: http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionAlreadyEstablished.html
[ConnectionNotDefined]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionNotDefined.html
[ConnectionNotEstablished]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionNotEstablished.html
[ModelAlreadyInstantiated]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelAlreadyInstantiated.html
[ModelNotDefined]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelNotDefined.html
[RecordNotFound]:               http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RecordNotFound.html
[RecordNotValid]:               http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RecordNotValid.html
[RelationNotDefined]:           http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RelationNotDefined.html
[ScopeNotDefined]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ScopeNotDefined.html
[StatementInvocationFailed]:    http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.StatementInvocationFailed.html
[StatementNotValid]:            http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.StatementNotValid.html
[UnableToSetFetchMode]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.UnableToSetFetchMode.html
[ConnectionCollection]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionCollection.html
[ModelCollection]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelCollection.html
[icanboogie/activerecord]:      https://github.com/ICanBoogie/activerecord
[icanboogie/bind-activerecord]: https://github.com/ICanBoogie/bind-activerecord
