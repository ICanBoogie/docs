# Getting started

## Server requirements

ICanBoogie has a few system requirements, which are usually covered by any PHP installation:

- PHP >= 5.6.4
- [OpenSSL PHP Extension](http://php.net/manual/en/book.openssl.php)
- [PDO PHP Extension](http://php.net/manual/en/book.pdo.php)
- [Mbstring PHP Extension](http://php.net/manual/en/book.mbstring.php)





## Installing ICanBoogie

ICanBoogie uses [Composer][] to manage its dependencies, before proceeding any further, make sure it
is available on your machine. Use its `create-project` command to install the _Hello World!_ starter
project:

```bash
$ composer create-project --prefer-dist -s dev icanboogie/app-hello hello
```





### Web directory

After installing ICanBoogie, you should configure your web server's document root to be the `web`
directory. The `index.php` in this directory serves as the front controller for all HTTP requests
entering your application.





### Directory permissions

After installing ICanBoogie, you may need to change the permissions of the `repository` folder so
that its content is writable your web server. The directory is used to store framework variables and
cache files. ICanBoogie will complain if you activate caching but the directory is either missing or
not writable.





### Configuration files

Configuration files are stored in `config` directories. The main configuration directory of your
application is `app/all/config`. What you define here overrides the configuration of the
application's components.





## Running ICanBoogie

You can use [PHP's built-in web server][] to serve your application. Use ICanBoogie's `run` command
to start a development server, the command will automatically find an available port and provide you
the URL to the server:

```bash
$ cd hello
$ ./icanboogie run
```

A similar message is displayed when the development server is running.

```text
ICanBoogie development server started on http://localhost:8006/
```

Of course [Apache](https://www.apache.org/) or [NGIX](https://www.nginx.com/) would be more
efficient, but [PHP's built-in web server][] requires zero setup.





[Composer]:                  https://getcomposer.org/
[PHP's built-in web server]: http://php.net/manual/en/features.commandline.webserver.php
