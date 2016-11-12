# Installation

## Server requirements

ICanBoogie has a few system requirements, which are usually covered by any PHP installation:

- PHP >= 5.6.4
- [OpenSSL PHP Extension](http://php.net/manual/en/book.openssl.php)
- [PDO PHP Extension](http://php.net/manual/en/book.pdo.php)
- [Mbstring PHP Extension](http://php.net/manual/en/book.mbstring.php)





## Installing ICanBoogie

ICanBoogie uses [Composer][] to manage its dependencies, before proceeding any further, make sure it
is available on your machine.

Use Composer `create-project` command in your terminal to install the _Hello World!_ starter
project:

```bash
$ composer create-project --prefer-dist -s dev icanboogie/app-hello hello
```





## Local Development Server

You can use [PHP's built-in web server][] to serve your application. Use ICanBoogie `serve` command
to start a development server, the command will automatically find an available port and provide you
the URL to the server:

```bash
$ cd hello
$ ./icanboogie serve
```

Of course Apache or NGIX would be more efficient, but [PHP's built-in web server][] requires zero
setup.





[Composer]:                  https://getcomposer.org/
[PHP's built-in web server]: http://php.net/manual/en/features.commandline.webserver.php
