# Query Interface

The query interface provides different ways to retrieve data from the database. Using the query
interface you can find records using a variety of methods and conditions; specify the order,
fields, grouping, limit, or the tables to join; use dynamic or scoped filters; check the existence
or particular records; perform various calculations.

Queries often starts from a model, in the following examples the `$model` variable is a reference
to a model managing _nodes_.





## Retrieving records from the database

To retrieve objects and values from the database several finder methods are provided. Each of these
methods defines the fragments of the database query. Complex queries can be created without having
to write any raw SQL.

The methods are:

* where
* select
* group
* having
* order
* limit
* offset
* join

All of the above methods return a [Query][] instance, allowing you to chain them.

Records can be retrieved in various ways, especially using the `all`, `one`, `pairs` or `rc`
magic properties. The `find()` method—used to retrieve a single record or a set of records—is the
most simple of them.





### Retrieving a single record

Retrieving a single record using its primary key is really simple. You can either use the `find()`
method of the model, or use the model as an array.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$article = $model->find(10);

# or

$article = $model[10];
```





### Retrieving a set of records

Retrieving a set or records using their primary key is really simple too:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$articles = $model->find([ 10, 32, 89 ]);

# or

$articles = $model->find(10, 32, 89);
```

The [RecordNotFound][] exception is thrown when a record could not be found. Its `records` property
can be used to know which records could be found and which could not.

Note: The records of the set are returned in the same order they are requested, this also applies
to the `records` property of the [RecordNotFound][] exception.





### Records caching

Records retrieved using `find()` are cached, they are reused by subsequent calls. This also
applies to the array notation.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$article = $model[12]; // '12' retrieved from database
$articles = $model->find(11, 12, 13); // '11' and '13' retrieved from database, '12' is reused.
```





## Conditions

The `where()` method specifies the conditions used to filter the records. It represents
the `WHERE`-part of the SQL statement. Conditions can either be specified as a string, as a
list of arguments or as an array.





### Conditions specified as a string

Adding a condition to a query can be as simple as `$model->where('is_online = 1');`. This
would return all the records where the `is_online` field equals "1".

__Warning:__ Building you own conditions as string can leave you vulnerable to SQL injection
exploits. For instance, `$model->where('is_online = ' . $_GET['online']);` is not safe. Always
use placeholders when you can't trust the source of your inputs:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where('is_online = ?', $_GET['online']);
```

Of course you can use multiple conditions:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where('is_online = ? AND is_home_excluded = ?', $_GET['online'], false);
```

`and()` is alias to `where()` and should be preferred when linking adding conditions:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where('is_online = ?', $_GET['online'])->and('is_home_excluded = ?', false);
```





### Conditions specified as an array (or list of arguments)

Conditions can also be specified as arrays:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where([ 'is_online' => $_GET['online'], 'is_home_excluded' => false ]);
```





### Subset conditions

Records belonging to a subset can be retrieved using an array as condition value:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where([ 'orders_count' => [ 1,3,5 ] ]);
```

This generates something like: `... WHERE (orders_count IN (1,3,5))`.





### Modifiers

When conditions are specified as an array it is possible to modify the comparing function.
Prefixing a field name with an exclamation mark uses the _not equal_ operator.

The following example demonstrates how to search for records where the `order_count` field is
different than "2":

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where([ '!order_count' => 2 ]);
```

```
… WHERE `order_count` != 2
```

This also works with subsets:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where([ '!order_count' => [ 1,3,5 ] ]);
```

```
… WHERE `order_count` NOT IN(1, 3, 5)
```





### Dynamic filters

Conditions can also be specified as methods, prefixed by `filter_by_` and separated by `_and_`:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->filter_by_slug('creer-nuage-mots-cle');
$model->filter_by_is_online_and_uid(true, 3);
```

Is equivalent to:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where([ 'slug' => 'creer-nuage-mots-cle' ]);
$model->where([ 'is_online' => true, 'uid' => 3 ]);
```





### Scopes

Scopes can be viewed as model defined filters. Models can define their own filters,
inherit filters from their parent class and override them. For instance, this is how a
`similar_site`, `similar_language` and `visible` scopes could be defined:

```php
<?php

namespace Website\Nodes;

use ICanBoogie\ActiveRecord\Query;

class Model extends \ICanBoogie\ActiveRecord\Model
{
	// …

	protected function scope_similar_site(Query $query, $site_id = null)
	{
		return $query->and('site_id = 0 OR site_id = ?', $site_id !== null ? $site_id : $this->current_site_id);
	}

	protected function scope_similar_language(Query $query, $language = null)
	{
		return $query->and('language = "" OR language = ?', $language !== null ? $language : $this->current_language);
	}

	protected function scope_visible(Query $query, $visible = true)
	{
		return $query->similar_site->similar_language->filter_by_is_online($visible);
	}

	// …
}
```

Now you can easily retrieve the first ten records that are visible on your website:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->visible->limit(10);
```

Or retrieve the first ten French records:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->similar_language('fr')->limit(10);
```





## Ordering

The `order()` method retrieves records in a specific order.

The following example demonstrates how to get records in the ascending order of their creation
date:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->order('created');
```

A direction can be specified:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->order('created ASC');
# or
$model->order('created DESC');
```

Multiple fields can be used while ordering:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->order('created DESC, title');
```

Records can also be ordered by field:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where([ 'nid' => [ 1, 2, 3 ] ])->order('nid', [ 2, 3, 1 ]);
# or
$model->where([ 'nid' => [ 1, 2, 3 ] ])->order('nid', 2, 3, 1);
```





## Grouping data

The `group()` method specifies the `GROUP BY` clause.

The following example demonstrates how to retrieve the first record of records grouped by day:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->group('date(created)')->order('created');
```





### Filtering groups

The `having()` method specifies the `HAVING` clause, which specifies the conditions of the
`GROUP BY` clause.

The following example demonstrates how to retrieve the first record created by day for the past
month:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->group('date(created)')->having('created > ?', new DateTime('-1 month'))->order('created');
```





## Limit and offset

The `limit()` method limits the number of records to retrieve.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->limit(10); // retrieves the first 10 records
```

With two arguments, an offset can be specified:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->limit(5, 10); // retrieves records from the 6th to the 16th
```

The offset can also be defined using the `offset()` method:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->offset(5); // retrieves records from the 6th to the last
$model->limit(10)->offset(5);
```





## Selecting specific fields

By default all fields are selected (`SELECT *`) and records are instances of the [ActiveRecord][]
class defined by the model. The `select()` method selects only a subset of fields from
the result set, in which case each row of the result set is returned as an array, unless a fetch
mode is defined.

The following example demonstrates how to get the identifier, creation date and title of records:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->select('nid, created, title');
```

Because the `SELECT` string is used _as is_ to build the query, complex SQL statements can be
used:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->select('nid, created, CONCAT_WS(":", title, language)');
```





## Joining tables

The `join()` method specifies the `JOIN` clause. A raw string or a model identifier
can be used to specify the join. The method can be used multiple times to create multiple joints.





### Joining tables using a subquery

A [Query][] instance can be joined as a subquery. The following options are available:

- `mode`: Specifies the join mode. Default: `INNER`.
- `as`: Alias for the subquery. Default: The alias of the model associated with the query.
- `on`: The column used for the conditional expression. Depending on the columns available, the
method tries to determine the best solution between `ON` and `USING`.

The following example demonstrates how to fetch users and order them by the number
of online article they published since last year. We use the join mode `LEFT` so that users
that did not publish articles are fetched as well.

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$online_article_count = $models['articles']
->select('user_id, COUNT(node_id) AS online_article_count')
->filter_by_type_and_created_at('articles', new DateTime('-1 year'))
->online
->group('user_id');

$users = $models['users']
->join($online_article_count, [ 'on' => 'user_id', 'mode' => 'LEFT' ])
->order('online_article_count DESC');
```







### Joining tables using a model

A join can be specified using a model or a model identifier, in which case the relationship
between that model and the model associated with the query is used to create the join. The
following options are available:

- `mode`: Specifies the join mode. Default: `INNER`.
- `as`: Alias for the joining model. Default: The alias of the joining model.

The column character ":" is used to distinguish a model identifier from a raw fragment.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */
/* @var $contents_model \ICanBoogie\ActiveRecord\Model */

$model->join($contents_model);
# or
$model->join(':contents');

$model->join(':contents', [ 'mode' => 'LEFT', 'as' => 'cnt' ]);
```

> **Note:** If a model identifier is provided, the model collection associated with the
query's model is used to obtain the model.





### Joining tables using a raw string

Finally, a join can be specified using a raw string, which will be included _as is_ in the
final SQL statement.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->join('INNER JOIN `contents` USING(`nid`)');
```





## Retrieving data

There are many ways to retrieve data. We have already seen the `find()` method, which can be used
to retrieve records using their identifier. The following methods or magic properties work with
conditions.





### Retrieving data by iteration

Instances of [Query][] are traversable, it's the easiest way the retrieve the rows
of a result set:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

foreach ($model->where('is_online = 1') as $node)
{
	// …
}
```





### Retrieving the complete result set

The magic property `all` retrieves the complete result set as an array:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$array = $model->all;
$array = $model->visible->order('created DESC')->all;
```

The `all()` method retrieves the complete result set using a specific fetch mode:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$array = $model->all(\PDO::FETCH_ASSOC);
$array = $model->visible->order('created DESC')->all(\PDO::FETCH_ASSOC);
```





### Retrieving a single record

The `one` magic property retrieves a single record:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$record = $model->one;
$record = $model->visible->order('created DESC')->one;
```

The `one()` method retrieves a single record using a specific fetch mode:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$record = $model->one(\PDO::FETCH_ASSOC);
$record = $model->visible->order('created DESC')->one(\PDO::FETCH_ASSOC);
```

Note: The number of records to retrieve is automatically limited to 1.





### Retrieving key/value pairs

The `pairs` magic property retrieves key/value pairs when selecting two columns, the first column
is the key and the second its value.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->select('nid, title')->pairs;
```

Results are similar to the following example:

```
array
  34 => string 'Créer un nuage de mots-clé' (length=28)
  57 => string 'Générer à la volée des miniatures avec mise en cache' (length=56)
  307 => string 'Mes premiers pas de développeur sous Ubuntu 10.04 (Lucid Lynx)' (length=63)
  ...
```





### Retrieving the first column of the first row

The `rc` magic property retrieves the first column of the first row.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$title = $model->select('title')->rc;
```

Note: The number of records to retrieve is automatically limited to 1.





## Defining the fetch mode

The fetch mode is usually selected by the query interface but the `mode` can be used to specify
it.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->select('nid, title')->mode(\PDO::FETCH_NUM);
```

The `mode()` method accepts the same arguments as the
[PDOStatement::setFetchMode](http://php.net/manual/fr/pdostatement.setfetchmode.php) method.

As we have seen in previous examples, the fetch mode can also be specified when fetching data
with the `all()` and `one()` methods.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$array = $model->order('created DESC')->all(\PDO::FETCH_ASSOC);
$record = $model->order('created DESC')->one(\PDO::FETCH_ASSOC);
```





## Checking the existence of records

The `exists()` method checks the existence of a record, it queries the database just like `find()`
but returns `true` when a record is found and `false` otherwise.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->exists(1);
```

The method accepts multiple identifiers in which case it returns `true` when all the
records exist, `false` when all the record don't exist, and an array otherwise.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->exists(1, 2, 999);
# or
$model->exists([ 1, 2, 999 ]);
```

The method would return the following result if records "1" and "2" exist but not record "999".

```
array
  1 => boolean true
  2 => boolean true
  999 => boolean false
```

The `exists` magic property is `true` if at least one record matching the specified conditions
exists, `false` otherwise.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->filter_by_author('Madonna')->exists;
```

The `exists` magic property of the model is `true` if the modal has at least one record, `false`
otherwise.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->exists;
```





## Counting

The `count` magic property is the number of records in a model or matching a query.

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->count;
```

Or on a query:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->filter_by_firstname('Ryan')->count;
```

Of course, all query methods can be combined:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->filter_by_firstname('Ryan')->join(':content')->where('YEAR(date) = 2011')->count;
```

The `count()` method returns an array with the number of recond for each value of a field:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->count('is_online');
```

```
array
  0 => string '35' (length=2)
  1 => string '145' (length=3)
```

In this example, there are 35 record online and 145 offline.





## Calculations

The `average()`, `minimum()`, `maximum()` and `sum()` methods are respectively used, for a column,
to compute its average value, its minimum value, its maximum value and its sum.

All calculation methods work directly on the model:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->average('price');
```

And on a query:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->filter_by_category('Toys')->average('price');
```

Of course, all query methods can be combined:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->filter_by_category('Toys')->join(':content')->where('YEAR(date) = 2011')->average('price');
```





## Some useful properties

The following properties might be helpful, especially when you are using the Query interface
to create a query string to be used in the subquery of another query:

- `conditions`: The conditions rendered as a string.
- `conditions_args`: The arguments to the conditions.
- `model`: The model associated with the query.





## Using a query as a subquery

The following example demonstrates how a query on some taxonomy models can be used as a subquery
to obtain only the online articles in a "music" category:

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$taxonomy_query = $models['taxonomy.terms/nodes']
->join(':taxonomy.vocabulary')
->join(':taxonomy_vocabulary/scopes')
->where([

	'termslug' => "music",
	'vocabularyslug' => "category",
	'constructor' => "articles"

])
->select('nid');

$articles = $models['articles']
->filter_by_is_online(true)
->and("nid IN ($taxonomy_query)", $taxonomy_query->conditions_args)
->all;

# or

$articles = $models['articles']
->filter_by_is_online_and_nid(true, $taxonomy_query)
->all;
```





## Deleting the records matching a query

The records matching a query can be deleted using the `delete()` method:

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$models['nodes']
->filter_by_is_deleted_and_uid(true, 123)
->limit(10)
->delete();
```

You might need to join tables to decide which record to delete, in which case you might
want to define in which tables the records should be deleted. The following example demonstrates
how to delete the nodes and comments of nodes belonging to user 123 and marked as deleted:

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$models['comments']
->filter_by_is_deleted_and_uid(true, 123)
->join(':nodes')
->delete('comments, nodes');
```

When using `join()` the table associated with the query is used by default. The following
example demonstrates how to delete nodes that lack content:

```php
<?php

/* @var $models \ICanBoogie\ActiveRecord\ModelCollection */

$models['nodes']
->join(':contents', [ 'mode' => 'LEFT' ])
->where('content.nid IS NULL')
->delete();
```





## Query interface summary

Retrieving records:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$record = $model[10];
# or
$record = $model->find(10);

$records = $model->find(10, 15, 19);
# or
$records = $model->find([ 10, 15, 19 ]);
```

Conditions:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->where('is_online = ?', true);
$model->where([ 'is_online' => true, 'is_home_excluded' => false ]);
$model->where('site_id = 0 OR site_id = ?', 1)->and('language = "" OR language = ?', "fr");

# Sets

$model->where([ 'order_count' => [ 1, 2, 3 ] ]);
$model->where([ '!order_count' => [ 1, 2, 3 ] ]); # NOT

# Dynamic filters

$model->filter_by_nid(1);
$model->filter_by_site_id_and_language(1, 'fr');

# Scopes

$model->visible;
$model->own->visible->ordered;
```

Grouping and ordering:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->group('date(created)')->order('created');
$model->group('date(created)')->having('created > ?', new DateTime('-1 month'))->order('created');
```

Limits and offsets:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->limit(10); // first 10 records
$model->limit(5, 10); // 6th to the 16th records

$model->offset(5); // from the 6th to the last
$model->offset(5)->limit(10);
```

Fields selection:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->select('nid, created, title');
$model->select('nid, created, CONCAT_WS(":", title, language)');
```

Joins:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->join($subquery, [ 'on' => 'nid' ]);
$model->join(':contents');
$model->join('INNER JOIN contents USING(nid)');
```

Retrieving data:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->all;
$model->order('created DESC')->all(PDO::FETCH_ASSOC);
$model->order('created DESC')->mode(PDO::FETCH_ASSOC)->all;
$model->order('created DESC')->one;
$model->select('nid, title')->pairs;
$model->select('title')->rc;
```

Testing object existence:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->exists;
$model->exists(1, 2, 3);
$model->exists([ 1, 2, 3 ]);
$model->where('author = ?', 'madonna')->exists;
```

Calculations:

```php
<?php

/* @var $model \ICanBoogie\ActiveRecord\Model */

$model->count;
$model->count('is_online'); // count is_online = 0 and is_online = 1
$model->filter_by_is_online(true)->count; // count is_online = 1
$model->average('score');
$model->minimum('age');
$model->maximum('age');
$model->sum('comments_count');
```





[ActiveRecord]:                 http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.html
[Query]:                        http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.Query.html
[RecordNotFound]:               http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RecordNotFound.html
