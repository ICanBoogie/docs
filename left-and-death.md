# The life and death of your application

Thanks to its Autoconfig system and a few conventions, running your application only requires
three lines:

```php
<?php

require 'vendor/autoload.php';

$app = ICanBoogie\boot();
$app();
```

1\. The first line is pretty common for applications using [Composer][], it creates and runs
its autoloader.

2\. On the second line the [Application][] instance is created with the _autoconfig_, its `boot()`
method is invoked, and the `ICanBoogie\Application::boot` event is fired. At this point ICanBoogie
and low-level components are configured and booted. Your application is ready to process requests.

3\. On the third line the application is run, which implies the following:

3.1\. The HTTP response code is set to 500, so that if a fatal error occurs the error message
won't be sent with the HTTP code 200 (Ok).

3.2\. The initial request is obtained and the `ICanBoogie\Application::run` event is fired with it.

3.3\. The request is executed to obtain a response.

3.4\. The response is executed to respond to the request. It should set the HTTP code to the
appropriate value.

3.5\. The `ICanBoogie\Application::terminate` event is fired at which point the application should be
terminated.

During the execution of the application, the following events are emitted:





## The application is configured

The `ICanBoogie\Application::configure` event of class [ConfigureEvent][] is fired once the
application is configured. Event hooks may use this event to alter the application configuration or
configure components.





## The application has booted

The `ICanBoogie\Application::boot` event of class [BootEvent][] is fired once the application has
booted. Event hooks may use this event to bootstrap components before the application is ran.





## The application is running

The `ICanBoogie\Application::run` event of class [RunEvent][] is fired when the application is
running. Event hooks may use this event to alter various states of the application, starting with
the initial request.





## The application is terminated

The `ICanBoogie\Application::terminate` event of class [TerminateEvent][] is fired after the
response to the initial request was sent and the application is about to be terminated. Event hooks
may use this event to cleanup loose ends.





[BootEvent]:           http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.BootEvent.html
[RunEvent]:            http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.RunEvent.html
[TerminateEvent]:      http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.TerminateEvent.html
