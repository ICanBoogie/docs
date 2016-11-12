# Relations between models

## Extending another model

A model can extend another, just like a class can extend another in PHP. Fields are inherited
and the primary key of the parent is used to link records together. When the model is queried, the
tables are joined. When values are inserted or updated, they are split to update the various
tables. Also, the connection of the parent model is inherited.

The `EXTENDING` attribute specifies the model to extend.

```php
<?php

use ICanBoogie\ActiveRecord\Model;
use ICanBoogie\ActiveRecord\ModelCollection;
use ICanBoogie\DateTime;

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

$models = new ModelCollection($connections, [

	'nodes' => [

		Model::SCHEMA => [

			'nid' => 'serial',
			'title' => 'varchar'

		]
	],

	'contents' => [

		Model::EXTENDING => 'nodes',
		Model::SCHEMA => [

			'body' => 'text',
			'date' => 'date'

		]
	],

	'news' => [

		Model::EXTENDING => 'contents'

	]
]);

$models['news']->save([

	'title' => "Testing!",
	'body' => "Testing...",
	'date' => DateTime::now()

]);
```

Contrary to tables, models are not required to define a schema if they extend another model, but
they may end with different parents.

In the following example the parent _table_ of `news` is `nodes` but its parent _model_ is
`contents`. That's because `news` doesn't define a schema and thus inherits the schema and some
properties of its parent model.

```php
<?php

/* @var $news \ICanBoogie\ActiveRecord\Model */

echo $news->parent->id;       // nodes
echo $news->parent_model->id; // contents
```





## One-to-one relation (belongs_to)

Records of a model can belong to records of other models. For instance, a news article belonging
to a user. The relation is specified with the `BELONGS_TO` attribute. When the _belongs to_
relation is specified, a getter is automatically added to the prototype of the records. For
instance, if records of a `news` model belong to records of a `users` model, than the `get_user`
getter is added to the prototype of the records of the `news` model. The user of a news record
can then by obtained using the magic property `user`.

```php
<?php

use ICanBoogie\ActiveRecord\Model;
use ICanBoogie\ActiveRecord\ModelCollection;

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

$models = new ModelCollection($connections, [

	'news' => [

		Model::BELONGS_TO => 'users',
		Model::SCHEMA => [

			'news_id' => 'serial',
			'uid' => 'foreign'
			// …

		]
	],

	'users' => [

		Model::SCHEMA => [
    
            'uid' => 'serial',
            'name' => 'varchar'
            // …

        ]
	]

]);

/* @var $news Model */

$record = $news->one;

echo "{$record->title} belongs to {$record->user->name}.";
```





## One-to-many relation (has_many)

A one-to-many relation can be established between two models. For instance, an article having
many comments. The relation is specified with the `HAS_MANY` attribute or the `has_many()` method
of the parent model. A getter is added to the active record class of the model and returns a
[Query][] instance when it is accessed.

The following example demonstrates how a one-to-many relation can be established between the
"articles" and "comments" models, while creating the models:

```php
<?php

use ICanBoogie\ActiveRecord\Model;
use ICanBoogie\ActiveRecord\ModelCollection;

// …

/* @var $connections \ICanBoogie\ActiveRecord\ConnectionCollection */

$models = new ModelCollection($connections, [

	'comments' => [

		Model::ACTIVERECORD_CLASS => Comment::class,
		Model::SCHEMA => [

			'comment_id' => 'serial',
			'article_id' => 'foreign',
			'body' => 'text'

		]
	],

	'articles' => [

		Model::ACTIVERECORD_CLASS => Article::class,
		Model::HAS_MANY => 'comments',
		Model::SCHEMA => [

			'article_id' => 'serial',
			'title' => 'varchar'

		]
	]

]);
```

The relation can also be established after the models are created using the `has_many` method:

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$articles = $models['articles'];
$comments = $models['comments'];

$articles->has_many($comments);
# or, if the model can be obtained with `get_model()`
$articles->has_many('comments');

# The local and foreign keys can be specified if they cannot be obtained from the models
$articles->has_many('comments', [ 'local_key' => 'article_id', 'related_key' => 'comment_id' ]);

# The name of the magic property can also be specified
$articles->has_many('comments', [ 'as' => 'article_comments' ]);
```

The following example demonstrates how all the comments of the author called "me"
can be retrieved in order from an article:

```php
<?php

/* @var $articles \ICanBoogie\ActiveRecord\Model */

$comments = $articles->one->comments->filter_by_author("me")->ordered->all;
```







[Query]:                        http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Query.html
