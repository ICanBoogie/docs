# Configuring event hooks

Event hooks are declared using `event` configuration fragments.

The following example demonstrates how an application can attach event hooks to be notified when
nodes are saved (or nodes subclasses), and when an authentication exception is thrown during the
dispatch of a request.

```php
<?php

// config/event.php

namespace App;

use ICanBoogie;
use Icybee;

$hooks = Hooks::class . '::';

return [

	Icybee\Modules\Nodes\SaveOperation::class . '::process' => $hooks . 'on_nodes_save',
	ICanBoogie\HTTP\AuthenticationRequired::class . '::rescue' => $hooks . 'on_authentication_required_rescue'

];
```

The following example demonstrates how to obtain the `event` config and the event collection.

```php
<?php

namespace ICanBoogie;

require 'vendor/autoload.php';

$app = boot();

$app->configs['event']; // obtain the "event" config.
$app->events;           // obtain an EventCollection instance created with the "event" config.
```





[Application]:           http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Core.html
[icanboogie/icanboogie]: https://github.com/ICanBoogie/ICanBoogie
[icanboogie/event]:      https://github.com/ICanBoogie/Event
[Autoconfig feature]:    https://github.com/ICanBoogie/ICanBoogie#autoconfig
[ICanBoogie]:            https://github.com/ICanBoogie/ICanBoogie
