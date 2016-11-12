# ActiveRecord Models

A _model_ is an object-oriented representation of a database table, or a group of tables.
A model is used to create, update, delete and query records. Models are instances of the [Model][]
class, and usually implement a specific business logic.





## Model collection

Models are managed using a model collection that resolves model attributes (such as database
connections) and instantiate them.





### Defining models

Model definitions can be specified while creating the [ModelCollection][] instance.

Note: You don't have to create the [Connection][] instances used by the models, you can use their
identifier which will get resolved when the model is needed.

Note: If `CONNECTION` is not specified the `primary` connection is used.

```php
<?php

use ICanBoogie\ActiveRecord\Model;
use ICanBoogie\ActiveRecord\ModelCollection;

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

$models = new ModelCollection($connections, [

	'nodes' => [

		// …
		Model::SCHEMA => [

			'nid' => 'serial',
			'title' => 'varchar'
			// …
		]
	],

	'contents' => [

		// …
		Model::EXTENDING => 'nodes'
	]
]);
```

Model definitions can be modified or added after the [ModelCollection][] instance has been created.

```php
<?php

use ICanBoogie\ActiveRecord\Model;

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$models['new'] = [

	// …
	Model::EXTENDING => 'contents'
];
```

You can modify the definition of a model until it is instantiated. A [ModelAlreadyInstantiated][]
exception is thrown in attempt to modify the definition of an already instantiated model.





### Obtaining a model

Use the [ModelCollection][] instance as an array to obtain a [Model][] instance.

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$nodes = $models['nodes'];
```

Models are instantiated on demand, so that you can define a hundred models an they will only by
instantiated, along with their database connection, when needed.

A [ModelNotDefined][] exception is thrown in attempts to obtain a model which is not defined.





### Checking defined models

The `isset()` function checks if a model is defined.

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

if (isset($models['nodes']))
{
	echo "The model 'node' is defined.\n";
}
```

The `definitions` magic property returns the current model definitions. The property is
_read-only_.

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

foreach ($models->definitions as $id => $definition)
{
	echo "The model '$id' is defined.\n";
}
```





### Instantiated models

An array with the instantiated models can be retrieved using the `instances` magic property. The
property is _read-only_.

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

foreach ($models->instances as $id => $model)
{
	echo "The model '$id' has been instantiated.\n";
}
```





### Installing / Uninstalling models

All the models managed by the provider can be installed and uninstalled with a single command
using the `install()` and `uninstall()` methods. The `is_installed()` method returns an array
of key/value pair where _key_ is a model identifier and _value_ `true` if the model is
installed, `false` otherwise.

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$models->install();
var_dump($models->is_installed()); // [ "nodes" => true, "contents" => true ]
$models->uninstall();
var_dump($models->is_installed()); // [ "nodes" => false, "contents" => false ]
```





### Model provider

The `get_model()` helper retrieves models using their identifier. It is used by active records to
retrieve their model when required, and by queries during joins. Models are retrieved using the
model collection returned by [ModelProvider][].

The following example demonstrates how to define a model provider:

```php
<?php

use ICanBoogie\ActiveRecord;
use ICanBoogie\ActiveRecord\ModelProvider;
use function ICanBoogie\ActiveRecord\get_model;

/* @var $models ActiveRecord\ModelCollection */

ModelProvider::define(function($id) use($models) {

	return $models[$id];

});

$nodes = get_model('nodes');
```

> **Note:** A `LogicException` is thrown if no provider is defined when `get_model()` require it.






```php
<?php

namespace App\Modules\Nodes;

use ICanBoogie\ActiveRecord;
use ICanBoogie\ActiveRecord\Model;
use ICanBoogie\ActiveRecord\ModelCollection;

class NodeModel extends Model
{
	// …
}

class Node extends ActiveRecord
{
	public $id;
	public $title;
	public $number;

	// …
}

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

$models = new ModelCollection($connections, [

	'nodes' => [

		Model::ACTIVERECORD_CLASS => Node::class,
		Model::SCHEMA => [

			'id' => 'serial',
			'title' => [ 'varchar', 80 ],
			'number' => [ 'integer', 'unsigned' => true ]

		]
	]
]);

$models->install();

$node_model = $models['nodes'];

$node = Node::from([

	'title' => "My first node",
	'number' => 123

], [ $node_model ]);
// ^ because we don't use a model provider yet, we need to specify the model to the active record

# or

$node = $node_model->new([

	'title' => "My first node",
	'number' => 123

]);

$id = $node->save();

echo "Saved node, got id: $id\n";
```





## Model definition





### Connection

The `CONNECTION` key specifies the database connection, an instance of the [Connection][] class.





### Instance class

The `ACTIVERECORD_CLASS` key specifies the class used to instantiate the active records of the
model.





### Name of the table

The `NAME` key specifies the name of the table. If a table prefix is defined by the connection,
it is used to prefix the table name. The `name` and `unprefixed_name` properties returns the
prefixed name and original name of the table:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

echo "table name: {$model->name}, original: {$model->unprefixed_name}.";
```

The `{self}` placeholder is replaced in queries by the `name` property:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$stmt = $model('SELECT * FROM `{self}` LIMIT 10');
```





### Schema

The columns and properties of the table are defined with a schema, which is specified by the
`SCHEMA` attribute. A key defines the name of the column, and a value defines the properties of
the column. Most column types use the following basic definition pattern:

```
'<identifier>' => '<type_and_default_options>'
# or
'<identifier>' => [ '<type>', <size> ];
```

The following types are available: `blob`, `char`, `integer`, `text`, `varchar`,
`bit`, `boolean`, `date`, `datetime`, `time`, `timestamp`, `year`, `enum`, `double` et `float`. The
`serial` and `foreign` special types are used to define auto incrementing primary keys and foreign
keys:

```php
<?php

[
	'nid' => 'serial', // bigint(20) unsigned NOT NULL AUTO_INCREMENT, PRIMARY KEY (`nid`)
	'uid' => 'foreign' // bigint(20) unsigned NOT NULL, KEY `uid` (`uid`)
];
```

The size of the field can be defined as an integer for the `blob`, `char`, `integer`, `varchar` and
`bit` types:

```php
<?php

[
	'title' => 'varchar', // varchar(255) NOT NULL
	'slug' => [ 'varchar', 80 ], // varchar(80) NOT NULL
	'weight' => 'integer', // int(11) NOT NULL
	'small_count' => [ 'integer', 8 ], // int(8) NOT NULL,
	'price' => [ 'float', [ 10, 3 ] ], // float(10,3) NOT NULL
];
```

The size of the field can be defined using the qualifiers `tiny`, `small`, `medium`,
`big` or `long` for the `blob`, `char`, `integer`, `text` and `varchar` types:

```php
<?php

[
	'body' => [ 'text', 'long' ] // longtext NOT NULL
];
```

The qualifier `null` specifies that a field can be null, by default fields are not capable
of receiving the `null`:

```
[ 'varchar', 'null' => true ] // varchar(255)
[ 'integer', 'null' => true ] // int(11)
[ 'integer' ] // int(11) NOT NULL
```

The qualifier `unsigned` specifies that a numeric value is not signed:

```
[ 'integer' ] // int(11)
[ 'integer', 'unsigned' => true ] // int(10) unsigned
```

The qualifier `indexed` specifies that a field should be indexed:

```php
<?php

[
	'slug' => [ 'varchar', 'indexed' => true ], // varchar(255) NOT NULL, KEY `slug` (`slug`)
	'is_online' => [ 'boolean', 'indexed' => true ], // tinyint(1) NOT NULL, KEY `is_online` (`is_online`),

	'page_id' => [ 'foreign', 'indexed' => 'page-content' ], // bigint(20) unsigned NOT NULL
	'content_id' => [ 'foreign', 'indexed' => 'page-content' ], // bigint(20) unsigned NOT NULL, KEY `page-content` (`page_id`, `content_id`)
];
```

The qualifier `primary` specifies that a column is a primary key. A multi-column primary key is
created if multiple columns have the `primary` qualifier:

```php
<?php

[
	'vtid' => [ 'foreign', 'primary' => true ],
	'nid' => [ 'foreign', 'primary' => true ],
	'weight' => [ 'integer', 'unsigned' => true ]
];

// ADD PRIMARY KEY ( `vtid` , `nid` )
```




## Creating the table associated with a model

Once the model has been defined, its associated table can easily be created with the `install()`
method.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->install();
```

The `is_installed()` method checks if a model has already been installed.

Note: The method only checks if the corresponding table exists, not if its schema is correct.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

if ($model->is_installed())
{
	echo "The model is already installed.";
}
```




## Placeholders

The following placeholders are replaced in model queries:

* `{alias}`: The alias of the table.
* `{prefix}`: The prefix of the table names of the connection.
* `{primary}`: The primary key of the table.
* `{self}`: The name of the table.
* `{self_and_related}`: The escaped name of the table and the possible JOIN clauses.





[Connection]:                   http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Connection.html
[Model]:                        http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Model.html
[ModelAlreadyInstantiated]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelAlreadyInstantiated.html
[ModelNotDefined]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelNotDefined.html
[ModelCollection]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelCollection.html
[ModelProvider]:                http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelProvider.html
