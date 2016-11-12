# Render

The [icanboogie/render][] package provides an API to render templates with a variety of template
engines.





## Render engines

Templates may be rendered with a variety of template engines. A shared collection is provided by
the `get_engines` helper. When it is first created the `EngineCollection::alter` event of class
[EngineCollection\AlterEvent][] is fired. Event hooks may use this event to add rendering engines
or replace the engine collection altogether.

> **Note:** Currently, the package only provides an engine to render PHP templates with the extension
`.phtml`, but third parties, such as the [Patron engine][], can easily provide others.

The following example demonstrates how the **Patron** engine can be added to handle `.patron`
extensions:

```php
<?php

use ICanBoogie\Render\EngineCollection;
use Patron\RenderSupport\PatronEngine;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(EngineCollection\AlterEvent $event, EngineCollection $target) {

	$event->instance['.patron'] = PatronEngine::class;

});
```

The following example demonstrates how to replace the engine collection with a decorator:

```php
<?php

use ICanBoogie\Render\EngineCollection;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(EngineCollection\AlterEvent $event, EngineCollection $target) {

	$event->instance = new MyEngineCollection($event->instance);

});
```




### The pathname of the template being rendered

Engines should use the `Engine::VAR_TEMPLATE_PATHNAME` variable to define the pathname of the
template being rendered, so that it is easy to track which template is being rendered and from
which location.





## Template resolver

A template resolver tries to match a template name with an actual template file. A set of paths
can be defined for the resolver to search in.

The `BasicTemplateResolver::alter` event of class [TemplateResolver\AlterEvent][] is fired on the
shared template resolver when it is created. Event hooks may use this event to add template paths
or replace the template resolver.

The following example demonstrates how to add template paths:

```php
<?php

use ICanBoogie\Render\TemplateResolver;
use ICanBoogie\Render\BasicTemplateResolver;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(TemplateResolver\AlterEvent $event, BasicTemplateResolver $target) {

	$target->add_paths(__DIR__ . '/my/templates/path);

});
```





### Decorating a template resolver

Decorating the basic template resolver allows you to use more complex resolving mechanisms than
its simple name/file mapping. The [ModuleTemplateResolver][] or
the [ApplicationTemplateResolver][] decorators are great examples. The [TemplateResolverTrait][]
trait may provide support  for implementing such a decorator.

> **Note:** The decorator must implement the [TemplateResolver][] interface.

The following example demonstrates how to replace the template resolver with a decorator:

```php
<?php

use ICanBoogie\Render\BasicTemplateResolver;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(BasicTemplateResolver\AlterEvent $event, BasicTemplateResolver $target) {

	$event->instance = new MyTemplateResolverDecorator($event->instance);

});
```





## Renderer

A [Renderer][] instance is used to render a template with a subject and options. An engine
collection and a template resolver are used to find suitable templates for the rendering.

A shared [Renderer][] instance is provided by the `get_renderer()` helper. When it is first
created the `Renderer::alter` event of class [Renderer\AlterEvent][] is fired. Event hooks may use
this event to alter the renderer or replace it.

The following example demonstrates how to replace the renderer:

```php
<?php

use ICanBoogie\Render\Renderer;

/* @var $events \ICanBoogie\EventCollection */

$events->attach(function(Renderer\AlterEvent $event, Renderer $target) {

	$event->instance = new MyRenderer($event->instance->engines, $event->instance->template_resolver);

});
```





## Helpers

The following helpers are defined:

- `get_engines()`: Returns a shared engine collection.
- `get_template_resolver()`: Returns a shared template resolver.
- `get_renderer()`: Returns a shared renderer.
- `render()`: Renders using the default renderer.





# Bindings

The [icanboogie/bind-render][] package binds [icanboogie/render][] to [ICanBoogie][], using its
autoconfig feature. It adds various getters and methods to the [Application][] instance and a
template resolver that uses the application paths to look for templates.

```php
<?php

$app = ICanBoogie\boot();

echo get_class($app->template_engines);  // ICanBoogie\Render\EngineCollection
echo get_class($app->template_resolver); // ICanBoogie\Binding\Render\ApplicationTemplateResolver
echo get_class($app->renderer);          // ICanBoogie\Render\Renderer

$app->render($app->models['articles']->one);
```

The shared [BasicTemplateResolver][] instance is replaced by an [ApplicationTemplateResolver][]
instance during the `TemplateResolver::alter` event of class [TemplateResolver\AlterEvent].





## Enhanced template resolver

[ApplicationTemplateResolver][] extends the template resolver used by [icanboogie/render][]
and [icanboogie/view][] to search templates in the application paths (see [Multi-site support](https://github.com/ICanBoogie/ICanBoogie#multi-site-support)).
Also, the "//" prefix can be used to search for templates from these paths .e.g.
"//my/special/templates/_form".





[ApplicationTemplateResolver]: http://api.icanboogie.org/bind-render/0.5/class-ICanBoogie.Binding.Render.ApplicationTemplateResolver.html
[Application]:                 http://api.icanboogie.org/icanboogie/4.0/class-ICanBoogie.Core.html
[ModuleTemplateResolver]:      http://api.icanboogie.org/module/2.3/class-ICanBoogie.Module.ModuleTemplateResolver.html
[BasicTemplateResolver]:       http://api.icanboogie.org/render/0.5/class-ICanBoogie.Render.BasicTemplateResolver.html
[EngineCollection\AlterEvent]: http://api.icanboogie.org/render/0.6/class-ICanBoogie.Render.EngineCollection.AlterEvent.html
[TemplateResolver\AlterEvent]: http://api.icanboogie.org/render/0.6/class-ICanBoogie.Render.TemplateResolver.AlterEvent.html
[Renderer]:                    http://api.icanboogie.org/render/0.6/class-ICanBoogie.Render.Renderer.AlterEvent.html
[Renderer\AlterEvent]:         http://api.icanboogie.org/render/0.6/class-ICanBoogie.Render.Renderer.AlterEvent.html
[TemplateResolver]:            http://api.icanboogie.org/render/0.6/class-ICanBoogie.Render.TemplateResolver.AlterEvent.html
[TemplateResolverTrait]:       http://api.icanboogie.org/render/0.6/class-ICanBoogie.Render.TemplateResolverTrait.AlterEvent.html
[ICanBoogie]:                  https://github.com/ICanBoogie\ICanBoogie
[icanboogie/render]:           https://github.com/ICanBoogie\Render
[icanboogie/bind-render]:      https://github.com/ICanBoogie\bind-render
[icanboogie/view]:             https://github.com/ICanBoogie\View
[Patron engine]:               https://github.com/Icybee/PatronViewSupport
