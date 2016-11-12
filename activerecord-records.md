# Records & Validation

An active record is an object-oriented representation of a record in a database. Usually, the
table columns are its public properties and it is not unusual that getters/setters and business
logic methods are implemented by its class.

The model managing the record needs to be specified when the instance is created. It can be
specified through the `__construct` method as a model identifier or a [Model][] instance, or by
defining the `MODEL_ID` constant, which is the favorite method since it can be easily
introspected.

```php
<?php

namespace App;

use ICanBoogie\ActiveRecord;

class Node extends ActiveRecord
{
	const MODEL_ID = "nodes";

	// …

	protected function get_next()
	{
		return $this->model->own->visible->where('date > ?', $this->date)->order('date')->one;
	}

	protected function get_previous()
	{
		return $this->model->own->visible->where('date < ?', $this->date)->order('date DESC')->one;
	}

	// …
}
```





## Instantiating an active record

Active record are instantiated just like any other object, but the `from()` method is
usually preferred for its shorter notation:

```php
<?php

$record = Article::from([

	'title' => "An example",
	'body' => "My first article",
	'language' => "en",
	'is_online' => true

]);
```





## Validating an active record

The `validate()` method validates an active record and returns a [ValidationErrors][] instance on
failure or an empty array on success. Your active record class should implement the
`create_validation_rules()` method to provide validation rules.

For following example demonstrates how a `User` active record class could implement the
`create_validation_rules()` method to validate its properties.

```php
<?php

use ICanBoogie\ActiveRecord;

class User extends ActiveRecord
{
	use ActiveRecord\Property\CreatedAtProperty;
	use ActiveRecord\Property\UpdatedAtProperty;

	public $id;
	public $username;
	public $email;

	// …

	/**
	 * @inheritdoc
	 */
	public function create_validation_rules()
	{
		return [

			'username' => 'required|max-length:32|unique',
			'email' => 'required|email|unique',
			'created_at' => 'required|datetime',
			'updated_at' => 'required|datetime',

		];
	}
}

// ...

$user = new User;
$errors = $user->validate();

if ($errors)
{
	// …
}
```





## Saving an active record

The `save()` method is used to save the active record to the database. Most properties of an active
record are persisted.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$record = $model[10];
$record->is_online = false;
$record->save();
```

Before a record is saved it is validated with the `validate()` method and if the validation fails a
[RecordNotValid][] exception is thrown with the validation errors.

```php
<?php

use ICanBoogie\ActiveRecord\RecordNotValid;

/* @var $record \ICanBoogie\ActiveRecord */

try
{
	$record->save();
}
catch (RecordNotValid $e)
{
	$errors = $e->errors;
	
	// …
}
```

The validation may be skipped using the `SAVE_SKIP_VALIDATION` option, of course the outcome is then unpredictable so use this option carefully:

```php
<?php

use ICanBoogie\ActiveRecord;

$record->save([ ActiveRecord::SAVE_SKIP_VALIDATION => true ]);
```

The `alter_persistent_properties()` is invoked to alter the properties
that will be sent to the model. One may extend the method to add, remove or alter properties
without altering the instance itself.





## Deleting an active record

The `delete()` method deletes the active record from the database:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$record = $model[190];
$record->delete();
```





## Date time properties

The package comes with three trait properties especially designed to handle [DateTime][]
instances: [DateTimeProperty][], [CreatedAtProperty][], [UpdatedAtProperty][]. Using this
properties you are guaranteed to always get a [DateTime][] instance, no matter what value type
is used to set the date and time.

```php
<?php

namespace App;

use ICanBoogie\ActiveRecord;
use ICanBoogie\ActiveRecord\Property\CreatedAtProperty;
use ICanBoogie\ActiveRecord\Property\UpdatedAtProperty;

class Node extends ActiveRecord
{
	public $title;

	use CreatedAtProperty;
	use UpdatedAtProperty;
}

$node = new Node;

echo get_class($node->created_at);   // ICanBoogie\Datetime
echo $node->created_at->is_empty;    // true
$node->created_at = 'now';
echo $node->created_at;              // 2014-02-21T15:00:00+0100
```





[CreatedAtProperty]:            http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Property.CreatedAtProperty.html
[DateTimeProperty]:             http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Property.DateTimeProperty.html
[Model]:                        http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Model.html
[RecordNotValid]:               http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RecordNotValid.html
[UpdatedAtProperty]:            http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Property.UpdatedAtProperty.html
[DateTime]:                     http://api.icanboogie.org/datetime/1.2/class-ICanBoogie.DateTime.html
[ValidationErrors]:             http://api.icanboogie.org/validate/latest/class-ICanBoogie.Validate.ValidationErrors.html
