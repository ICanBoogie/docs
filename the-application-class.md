# The Application class

The documentation and the code base often refer to an `ICanBoogie\Application` class, and if you
check the source code you'll see that it's used in many places, but you might be surprise that the
class itself is nowhere to be found. That's because the class must be defined by _your_ application.
Whether it is defined in your bootstrap file, or as a stand-alone auto-loadable file, you have to
define the class and it must extend `ICanBoogie\Core`.

When you start writing your application, it might seem redundant to define a class that extends
another without adding features, but as your application grows you'll soon add bindings to the
class, and because the code base uses `ICanBoogie\Application` instead of `ICanBoogie\Core`, they
will be readily available to your IDE.

The following example demonstrates how to add bindings from the [icanboogie/bind-render][] package
to your application.

```php
<?php

// app/all/Application.php

namespace ICanBoogie;

class Application extends Core
{
	use Binding\Render\ApplicationBindings;
}
```





[icanboogie/bind-render]: https://github.com/ICanBoogie/bind-render
