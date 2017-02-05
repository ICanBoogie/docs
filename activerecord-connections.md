# ActiveRecord connections

ActiveRecord connections (or database connections) are managed by a [ConnectionCollection][]
instance. Because the class implements [ArrayAccess][], instances are used like arrays.





## Defining connections

Connection definitions can be specified while creating the [ConnectionCollection][]
instance.

```php
<?php

use ICanBoogie\ActiveRecord\ConnectionCollection;

$connections = new ConnectionCollection([

	'temporary' => [

		'dsn' => 'sqlite::memory:'

	],

	'bad_connection' => [

		'dsn' => 'mysql:dbname=bad_database' . uniqid()

	]

]);
```

Or after:

```php
<?php

$connections['primary'] = [

	'dsn' => 'mysql:dbname=example',
	'username' => getenv('ACTIVERECORD_CONNECTION_PRIMARY_USERNAME'),
	'password' => getenv('ACTIVERECORD_CONNECTION_PRIMARY_PASSWORD'),

];
```

You can modify a connection definition until it is established. A [ConnectionAlreadyEstablished][]
exception is thrown in attempt to modify the definition of an already established connection.





### Defining the prefix of the database tables

The `ConnectionOptions::TABLE_NAME_PREFIX` option specifies the prefix for all the tables name
of the connection. Thus, if the `icanboogie` prefix is defined, the `nodes` table is
renamed as `icanboogie_nodes`.

The `{table_name_prefix}` placeholder is replaced in queries by the prefix:

```php
<?php

/* @var $connection \ICanBoogie\ActiveRecord\Connection */

$statement = $connection('SELECT * FROM `{table_name_prefix}nodes` LIMIT 10');
```





### Defining the charset and collate to use

The `ConnectionOptions::CHARSET_AND_COLLATE` option specify the charset and the collate of the
connection in a single string e.g. "utf8/general_ci" for the "utf8" charset and the
"utf8_general_ci" collate.

The `{charset}` and `{collate}` placeholders are replaced in queries:

```php
<?php

/* @var $connection \ICanBoogie\ActiveRecord\Connection */

$connection('ALTER TABLE nodes CHARACTER SET "{charset}" COLLATE "{collate}"');
```




### Specifying a time zone

The `ConnectionOptions::TIMEZONE` option specifies the time zone of the connection.





## Checking defined connections

The `isset()` function is used to check if a connection is defined.

```php
<?php

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

if (isset($connections['one']))
{
	echo "The connection 'one' is defined.\n";
}
```

The read-only property `definitions` holds the connection definitions.

```php
<?php

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

foreach ($connections->definitions as $id => $definition)
{
	echo "The connection '$id' is defined.\n";
}
```





## Establishing a connection

A database connection is obtained by reading the connection identifier from the connection
collection. The connection is established on the first read and reused on subsequent reads. A
[ConnectionNotDefined][] exception is thrown in attempt to obtain a connection that is not defined,
and a [ConnectionNotEstablished][] exception is thrown if the connection could not be established.

A database connection is represented by a [Connection][] instance. Because it extends [PDO][], takes
the same parameters. Custom options can also be provided with the driver options to specify a prefix
to table names, specify the charset and collate of the connection or its timezone.

The following example demonstrates how to establish a connection using a connection collection:

```php
<?php

use ICanBoogie\ActiveRecord\ConnectionNotDefined;
use ICanBoogie\ActiveRecord\ConnectionNotEstablished;

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

try
{
	$one = $connections['one'];
}
catch (ConnectionNotDefined $e)
{
	// ..
}
catch (ConnectionNotEstablished $e)
{
	// ..
}
```

The following example demonstrates how connections can be established to MySQL and SQLite by
creating [Connection][] instances directly:

```php
<?php

use ICanBoogie\ActiveRecord\Connection;

# a connection to a MySQL database
$connection = new Connection('mysql:dbname=example', 'username', 'password');

# a connection to a SQLite temporary database stored in memory
$connection = new Connection('sqlite::memory:');
```





## Established connections

The `established` read-only property holds an array of established connections:

```php
<?php

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

foreach ($connections->established as $id => $connection)
{
	echo "The connection '$id' is established.\n";
}
```

The [ConnectionCollection][] instance itself can be used to traverse established connections.

```php
<?php

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

foreach ($connections as $id => $connection)
{
	echo "The connection '$id' is established.\n";
}
```





[Connection]:                   http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Connection.html
[ConnectionAlreadyEstablished]: http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionAlreadyEstablished.html
[ConnectionCollection]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionCollection.html
[ConnectionNotDefined]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionNotDefined.html
[ConnectionNotEstablished]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionNotEstablished.html
[ArrayAccess]:                  http://php.net/manual/en/class.arrayaccess.php
[PDO]:                          http://php.net/manual/en/book.pdo.php
