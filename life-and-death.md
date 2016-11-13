# The life and death of your application

Thanks to its [Autoconfig][] feature and a few conventions, running your application requires very
few lines of code:

```php
<?php

namespace ICanBoogie;

require 'vendor/autoload.php';

$app = boot();
$app();
```

1\. Pretty common for applications using [Composer][], the auto-loader if the first thing to run.

2\. Then `boot()` helper to instantiate the application with its [Application][] class and
[autoconfig][]. After the application has booted, the `ICanBoogie\Application::boot` event is fired.
At this point ICanBoogie and its low-level components are configured and booted, the application is
ready to process requests.

3\. Finally, the application is ran, which involves the following:

3.1\. The HTTP response code is set to 500 because we don't want to return a 200 (Ok) if a fatal error occurs.

3.2\. The initial request is obtained and is used to fire the `ICanBoogie\Application::run` event.

3.3\. The request is executed to obtain a response.

3.4\. The response is executed to respond to the request. It should set the HTTP code to the
appropriate value.

3.5\. The `ICanBoogie\Application::terminate` event is fired at which point the application should be
terminated.




## Events

During the execution of the application, the following events are fired:





### The application is configured

The `ICanBoogie\Application::configure` event of class [ConfigureEvent][] is fired once the
application is configured. Event hooks may use this event to alter the application configuration or
configure components.





### The application has booted

The `ICanBoogie\Application::boot` event of class [BootEvent][] is fired once the application has
booted. Event hooks may use this event to bootstrap components before the application is ran.





### The application is running

The `ICanBoogie\Application::run` event of class [RunEvent][] is fired when the application is
running. Event hooks may use this event to alter various states of the application, starting with
the initial request.





### The application is terminated

The `ICanBoogie\Application::terminate` event of class [TerminateEvent][] is fired after the
response to the initial request was sent and the application is about to be terminated. Event hooks
may use this event to cleanup loose ends.





[BootEvent]:           http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.BootEvent.html
[ConfigureEvent]:      http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.ConfigureEvent.html
[RunEvent]:            http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.RunEvent.html
[TerminateEvent]:      http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Application.TerminateEvent.html
[Application]:         ./the-application-class.md
[Autoconfig]:          ./autoconfig.md
[autoconfig]:          ./autoconfig.md
[Composer]:            https://getcomposer.org/
