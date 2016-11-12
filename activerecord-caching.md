# Records caching

By default, each model uses an instance of [RuntimeActiveRecordCache][] to cache its records.
This cache stores the records for the duration of the request, it is brand new with each HTTP
request. The cache is obtained using the prototype features of the model, through the
`activerecord_cache` magic property.

Third parties can provide a different cache instance simply by overriding the
`lazy_get_activerecord_cache` method:

```php
<?php

use ICanBoogie\ActiveRecord\Model;
use ICanBoogie\Prototype;

Prototype::from(Model::class)['lazy_get_activerecord_cache'] = function(Model $model) {

	return new MyActiveRecordCache($model);

};
```

Or using a `prototype` configuration fragment:

```php
<?php

// config/prototype.php

return [

	'ICanBoogie\ActiveRecord\Model::lazy_get_activerecord_cache' => 'my_activerecord_cache_provider'

];
```





[RuntimeActiveRecordCache]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ActiveRecordCache.RuntimeActiveRecordCache.html
