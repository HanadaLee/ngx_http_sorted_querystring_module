Nginx Sorted Args Module
===============================

This Nginx module orders the args parameters of an HTTP request alphanumerically and makes the sorted key-value pairs accessible using an Nginx variable.

Requests like `/index.html?b=2&a=1&c=3`, `/index.html?b=2&c=3&a=1`, `/index.html?c=3&a=1&b=2`, `/index.html?c=3&b=2&a=1` will produce the same normalized args `a=1&b=2&c=3` which can be accessed within Nginx using the `$sorted_args` variable.

This is especially useful if you want to normalize the args to be used in a cache key, for example when used with the `proxy_cache_key` directive.

It is also possible to remove one or more undesired query parameters by defining their name with the `sorted_args_filter` directive, like `sorted_args_filter <parameter_name> [<parameter_name> <parameter_name> ...];`.

_This module is not distributed with the Nginx source. See [the installation instructions](#installation)._


Configuration
-------------

An example:

```
pid         logs/nginx.pid;
error_log   logs/nginx-main_error.log debug;

worker_processes    2;

events {
  worker_connections  1024;
  #use                 kqueue; # MacOS
  use                 epoll; # Linux
}

http {
  default_type    text/plain;

  types {
    text/html   html;
  }

  log_format main  '[$time_local] $host "$request" $request_time s '
                 '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
                 'cache_status: "$upstream_cache_status" args: "$args '
                 'sorted_args: "$sorted_args" ';

  access_log       logs/nginx-http_access.log;

  proxy_cache_path /tmp/cache levels=1:2 keys_zone=zone:10m inactive=10d max_size=100m;

  server {
    listen          8080;
    server_name     localhost;

    access_log       logs/nginx-http_access.log main;

    location /filtered {
      sorted_args_filter v _ time b;

      proxy_set_header Host "static_files_server";
      proxy_pass http://localhost:8081;

      proxy_cache zone;
      proxy_cache_key "$sorted_args";
      proxy_cache_valid 200 1m;
    }

    location / {
      proxy_pass http://localhost:8081;

      proxy_cache zone;
      proxy_cache_key "$sorted_args";
      proxy_cache_valid 200 10m;
    }
  }

  server {
    listen          8081;

    location / {
      return 200 "$args\n";
    }
  }
}
```

Variables
---------

* **$sorted_args** - just list the IP considered as remote IP on the connection


Directives
----------

* **sorted_args_filter** - list parameters to be filtered while using the `$sorted_args` variable.


<a id="installation"></a>Installation Instructions
--------------------------------------------------

[Download Nginx Stable](http://nginx.org/en/download.html) source and uncompress it (ex.: to ../nginx). You must then run ./configure with --add-module pointing to this project as usual. Something in the lines of:

    $ ./configure \
        --add-module=../nginx-sorted-args-module \
        --prefix=/home/user/dev-workspace/nginx
    $ make
    $ make install


Running Tests
-------------

This project uses [nginx_test_helper](https://github.com/wandenberg/nginx_test_helper) on the test suite. So, after you've installed the module, you can just download the necessary gems:

    $ cd test
    $ bundle install

And run rspec pointing to where your Nginx binary is (default: /usr/local/nginx/sbin/nginx):

    $ NGINX_EXEC=../path/to/my/nginx rspec .


Changelog
---------

This is still a work in progress. Be the change. And take a look on the Changelog file.
