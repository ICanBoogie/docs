# Configuring protoypes

Using `prototype` configuration fragments, components may bind prototypes methods.

The following example demonstrates how an application may bind a `url()` method and a `url` property to instances of `ActiveRecord`:

```php
<?php

// config/prototype.php

namespace App;

$hooks = Hooks::class . '::';

return [

	Article::class . '::url' => $hooks . '::url',
	Article::class . '::get_url' => $hooks . '::url'

];
```

The following examples demonstrates how to retrieve the `prototype` configuration: 

```php
<?php

namespace ICanBoogie;

require 'vendor/autoload.php';

$app = boot();
$app->configs['prototype'];
```






[Core]:          http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Core.html
[documentation]: http://api.icanboogie.org/bind-prototype/3.0/

[icanboogie/icanboogie]: https://github.com/ICanBoogie/ICanBoogie
[icanboogie/prototype]:  https://github.com/ICanBoogie/Prototype
[Autoconfig feature]:    https://github.com/ICanBoogie/ICanBoogie#autoconfig
[ICanBoogie]:            https://github.com/ICanBoogie/ICanBoogie
