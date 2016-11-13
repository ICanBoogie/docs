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




## Configuration

### Public directory

After installing ICanBoogie, you should configure your web server's document root to be the `web`
directory. The index.php in this directory serves as the front controller for all HTTP requests
entering your application.





### Configuration files

All of the configuration files for ICanBoogie are stored in the `config` of your application and its
components. What you define in the configuration files of your application overrides the
configuration of its components.





### Directory permissions

After installing ICanBoogie, you may need to change the permissions of the `repository` folder so
that its content is writable your web server. The directory is used to store framework variables and
cache files. ICanBoogie will complain if you activate caching but the directory is either missing or
not writable.





[Composer]:                  https://getcomposer.org/
[PHP's built-in web server]: http://php.net/manual/en/features.commandline.webserver.php
