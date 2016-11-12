# Controllers

Previous examples demonstrated how closures could be used to handle routes. Closures are
perfectly fine when you start building your application, but as soon as it grows you might want
to use controller classes instead to better organize your application. You can map each route to
its [Controller][] class, or use the [ActionTrait][] to group related HTTP
request handling logic into a single controller.





## Controller response

When invoked, the controller should return a result, or `null` if it can't handle the request.
The result of the `action()` method is handled by the `__invoke()` method: if the result is a
[Response][] instance it is returned as is; if the [Response][] instance attached to the
controller has been initialized (through the `$this->response` getter, for instance), the result
is used as the body of the response; otherwise,  the result is returned as is.





## Before the action is executed

The event `ICanBoogie\Routing\Controller::action:before` of class
[Controller\BeforeActionEvent][] is fired before the `action()` method is invoked. Event hooks may
use this event to provide a response and thus cancelling the action. Event hooks may also use
this event to alter the controller before the action is executed.





## After the action is executed

The event `ICanBoogie\Routing\Controller::action:before` of class [Controller\ActionEvent][]
is fired after the `action()` method was invoked. Event hooks may use this event to alter the
result of the method.





## Basic controllers

Basic controllers extend [Controller][] and must implement the `action()` method.

> **Note:** The `action()` method is invoked _from within_ the controller, by the `__invoke()` method,
and should be defined as _protected_. The `__invoke()` method is final, thus cannot be overridden.

```php
<?php

namespace App\Modules\Articles\Routing;

use ICanBoogie\HTTP\Request;
use ICanBoogie\Routing\Controller;

class DeleteController extends Controller
{
	protected function action(Request $request)
	{
		// Your code goes here, and should return a string or a Response instance
	}
}
```

Although any class implementing `__invoke()` is suitable as a controller, it is recommended to
extend [Controller][] as it makes accessing your application features much easier. Also, you might
benefit from prototype methods and event hooks attached to the [Controller][] class, such as the
`view` property added by the [icanboogie/view][] package.

The following properties are provided by the [Controller][] class:

- `name`: The name of the controller, extracted from its class name e.g. "articles_delete".
- `request`: The request being dispatched.
- `route`: The route being dispatched.





## Action controllers

Action controllers are used to group related HTTP request handling logic into a class and use
HTTP methods to separate concerns. An action controller is created by extending the
[Controller][] class and using [ActionTrait][].

The following example demonstrates how an action controller can be used to display a contact
form, handle its submission, and redirect the user to a _success_ page. The action invoked
inside the controller is defined after the "#" character. The action may as well be defined
using the `ACTION` key.

```php
<?php

// routes.php

use ICanBoogie\Routing\RouteDefinition;

return [

	'contact' => [

		RouteDefinition::PATTERN => '/contact',
		RouteDefinition::CONTROLLER => AppController::class . '#contact'

	],
	
	# or
	
	'contact' => [

		RouteDefinition::PATTERN => '/contact',
		RouteDefinition::CONTROLLER => AppController::class,
		RouteDefinition::ACTION => 'contact'

	]

];
```

The HTTP method is used as a prefix for the method handling the action. The prefix `any` is used
for methods that handle any kind of HTTP method, they are a fallback when more accurate methods
are not available. If you don't care about that, you can omit the HTTP method.

```php
<?php

use ICanBoogie\Routing\Controller;

class AppController extends Controller
{
	use Controller\ActionTrait;
	
	protected function action_any_contact()
	{
		return new ContactForm;
	}

	protected function action_post_contact()
	{
		$form = new ContactForm;
		$request = $this->request;

		if (!$form->validate($request->params))
		{
			return $this->redirect($this->routes['contact']);
		}

		// …

		$email = $request['email'];
		$message = $request['message'];

		// …
	}
}
```





## Resource controllers

A resource controller groups the different actions required to handle a resource in a
[RESTful][] fashion. It is created by extending the [Controller][] class and
using [ActionTrait][].

The following table list the verbs/routes and their corresponding action. `{name}` is the
placeholder for the plural name of the resource, while `{id}` is the placeholder for the
resource identifier.

| HTTP verb   | Path                | Action    | Used for                                   |
| ----------- | ------------------- | --------- | ------------------------------------------ |
| `GET`       | `/{name}`           | `index`   | A list of `{resource}`                     |
| `GET`       | `/{name}/new`       | `new`     | A form for creating a new `{resource}`     |
| `POST`      | `/{name}`           | `create`  | Create a new `{resource}`                  |
| `GET`       | `/{name}/{id}`      | `show`    | A specific `{resource}`                    |
| `GET`       | `/{name}/{id}/edit` | `edit`    | A form for editing a specific `{resource}` |
| `PATCH/PUT` | `/{name}/{id}`      | `update`  | Update a specific `{resource}`             |
| `DELETE`    | `/{name}/{id}`      | `delete`  | Delete a specific `{resource}`             |

The routes listed are more of a guideline than a requirement, still the actions are important.

The following example demonstrates how the resource controller for _articles_ may be
implemented. The example implements all actions, but you are free to implement only
some of them.

```php
<?php

use ICanBoogie\Routing\Controller;

class PhotosController extends Controller
{
	use Controller\ActionTrait;

	protected function action_index()
	{
		// …
	}
	
	protected function action_new()
	{
		// …
	}
	
	protected function action_create()
	{
		// …
	}
	
	protected function action_show($id)
	{
		// …
	}
	
	protected function action_edit($id)
	{
		// …
	}
	
	protected function action_update($id)
	{
		// …
	}

	protected function action_delete($id)
	{
		// …
	}
}
```





### Defining resource routes using `RouteMaker`

Given a resource name and a controller, the `RouteMaker::resource()` method makes the various
routes required to handle a resource. Options can be specified to filter the routes to create,
specify the name of the _key_ property and/or it's regex constraint, or name routes.

The following example demonstrates how to create routes for an _article_ resource:

```php
<?php

namespace App;

use ICanBoogie\Routing\RouteMaker as Make;

// create all resource actions definitions
$definitions = Make::resource('articles', ArticlesController::class);

// only create the _index_ definition
$definitions = Make::resource('articles', ArticlesController::class, [

	Make::OPTION_ONLY => Make::ACTION_INDEX

]);

// only create the _index_ and _show_ definitions
$definitions = Make::resource('articles', ArticlesController::class, [

	Make::OPTION_ONLY => [ Make::ACTION_INDEX, Make::ACTION_SHOW ]

]);

// create definitions except _destroy_
$definitions = Make::resource('articles', ArticlesController::class, [

	Make::OPTION_EXCEPT => Make::ACTION_DELETE

]);

// create definitions except _updated_ and _destroy_
$definitions = Make::resource('articles', PhotosController::class, [

	Make::OPTION_EXCEPT => [ Make::ACTION_UPDATE, Make::ACTION_DELETE ]

]);

// specify _key_ property name and its regex constraint
$definitions = Make::resource('articles', ArticlesController::class, [

	Make::OPTION_ID_NAME => 'uuid',
	Make::OPTION_ID_REGEX => '{:uuid:}'

]);

// specify the identifier of the _create_ definition
$definitions = Make::resource('articles', ArticlesController::class, [

	Make::OPTION_AS => [ 

		Make::ACTION_CREATE => 'articles:build' 

	]

]);
```

> **Note:** It is not required to define all the resource actions, only define the one you actually need.





[Response]:                            http://api.icanboogie.org/http/3.0/class-ICanBoogie.HTTP.Response.html
[ActionTrait]:                         http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.ActionTrait.html
[Controller]:                          http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.html
[Controller\BeforeActionEvent]:        http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.BeforeActionEvent.html
[Controller\ActionEvent]:              http://api.icanboogie.org/routing/4.0/class-ICanBoogie.Routing.Controller.ActionEvent.html

[icanboogie/view]:                     https://github.com/ICanBoogie/View
[RESTful]:                             https://en.wikipedia.org/wiki/Representational_state_transfer
