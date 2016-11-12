# An introduction to ActiveRecord

__Connections__, __models__ and __active records__ are the foundations of everything that concerns
database access and management. They are used to establish database connections, manage tables and
their possible relationship, as well as manage the records of these tables. Leveraging OOP, the
models and active records are instances which properties, getters/setters and behavior can be
inherited in a business logic.

Using the __query interface__, you won't have to write raw SQL, manage table relationship,
or worry about injection.

Finally, using __providers__ you can define all your connections and models in a single place.
Connections are established and models are instantiated on demand, so feel free the define
hundreds of them.





## Exceptions

The exception classes defined by the package implement the `ICanBoogie\ActiveRecord\Exception`
interface so that they can easily be identified:

```php
<?php

try
{
	// …
}
catch (\ICanBoogie\ActiveRecord\Exception $e)
{
	// an ActiveRecord exception
}
catch (\Exception $e)
{
	// some other exception
}
```

The following exceptions are defined:

- [ConnectionAlreadyEstablished][]: Exception thrown in attempt to set/unset the definition of an
already established connection.
- [ConnectionNotDefined][]: Exception thrown in attempt to obtain a connection that is not defined.
- [ConnectionNotEstablished][]: Exception thrown when a connection cannot be established.
- [ModelAlreadyInstantiated][]: Exception thrown in attempt to set/unset the definition of an
already instantiated model.
- [ModelNotDefined][]: Exception thrown in attempt to obtain a model that is not defined.
- [RecordNotFound][]: Exception thrown when one or several records cannot be found.
- [RelationNotDefined][]: Exception thrown in attempt to obtain a relation that is not defined.
- [ScopeNotDefined][]: Exception thrown in attempt to obtain a scope that is not defined.
- [StatementInvocationFailed][]: Exception thrown when invoking a statement fails (`execute()` returned `false`).
- [StatementNotValid][]: Exception thrown in attempt to execute a statement that is not valid.
- [UnableToSetFetchMode][]: Exception thrown when the fetch mode of a statement fails to be set.





[ConnectionAlreadyEstablished]: http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionAlreadyEstablished.html
[ConnectionNotDefined]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionNotDefined.html
[ConnectionNotEstablished]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ConnectionNotEstablished.html
[ModelAlreadyInstantiated]:     http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelAlreadyInstantiated.html
[ModelNotDefined]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ModelNotDefined.html
[RecordNotFound]:               http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RecordNotFound.html
[RecordNotValid]:               http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RecordNotValid.html
[RelationNotDefined]:           http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.RelationNotDefined.html
[ScopeNotDefined]:              http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.ScopeNotDefined.html
[StatementInvocationFailed]:    http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.StatementInvocationFailed.html
[StatementNotValid]:            http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.StatementNotValid.html
[UnableToSetFetchMode]:         http://api.icanboogie.org/activerecord/4.0/class-ICanBoogie.ActiveRecord.UnableToSetFetchMode.html
