# Multi-site support

ICanBoogie has built-in multi-site support and can be configured for different domains or
environments. Even if you are dealing with only one domain, this feature can be used to provide
different configuration for the _dev_, _staging_, and _production_ versions of a same application.

The intended location for your custom application code is in a separate `app` directory, but another
directory can be defined with the `app-root` _autoconfig_ directive, which is relative to the `root`
directive.

ICanBoogie searches for `config` directories to add to the _autoconfig_. Packages may alter the
_autoconfig_ as well. For instance, [icanboogie/module][] searches for `modules` directories.





## Instance name

The instance name of the application is used to resolve the application paths. It is usually
defined by the `ICANBOOGIE_INSTANCE` environment variable but can be retrieved from `PHP_SAPI` or
`$_SERVER`.

If the variable is not defined, the instance name defaults as follows:

- The application runs from the CLI (e.g. `PHP_SAPI` == "cli"), "cli" is used as instance name.
- `$_SERVER['SERVER_NAME']` is defined, it is used as instance name.





## Resolving applications paths

Consider an application root directory with the following directories:

```
all
cli
default
dev
icanboogie.org
org
```

The directory `all` contains resources that are common to all instances. It is always added if
present. The directory `default` is only added if there no directory matches the instance name.

To resolve the matching directory, the instance name is first broken into parts and the most
specific ones are removed until a corresponding directory is found. For instance, given the
server name `www.icanboogie.dev`, the following directories are tried:
`www.icanboogie.dev`, `icanboogie.dev`, and finally `dev`.

If the instance name cannot be resolved into a directory, `default` is used instead.





[icanboogie/module]:            https://github.com/ICanBoogie/Module
