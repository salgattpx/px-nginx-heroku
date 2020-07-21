# nginx buildpack for Heroku

This buildpack allows you to run nginx (w/lua module) on a Heroku dyno.

## Features

* Downloads and builds nginx 1.16.1 from source
* Downloads and builds LuaJIT 2.1-20200102 from source
* Downloads and installs luarocks 3.3.1 from source
* Builds nginx with SSL, PCRE, and Lua extensions
* Installs the PerimeterX Lua module via Luarocks
* Caches all builds for lightning-fast deploys

Note: SSL and PCRE packages are [already available](https://devcenter.heroku.com/articles/cedar-ubuntu-packages) on Heroku, so they don't need to be downloaded before building nginx.

## Before buildpack installation

This buildpack requires a few "Config Vars" to be setup within the Heroku application before it is installed. 

1. Login to your Heroku account.
2. Open the application that this buildpack will be added to
3. Navigate to the application's *Settings* page
4. Click on the *Reveal Config Vars* button within the *Config Vars* section of the *Settings* page
5. Add the following Config Vars using the below table's Key : Value entries.
   | KEY | VALUE |
   | --- | ----- |
   | PX_APP_ID | This is the PerimeterX Application ID found within your application in the PerimeterX Portal |
   | PX_AUTH_TOKEN | This is the PerimeterX Server Token found within your application in the PerimeterX Portal |
   | PX_COOKIE_KEY | This is the PerimeterX Risk Cookie found within your policy in the PerimeterX Portal |
   | PX_BLOCKING_ENABLED | This controls whether the PerimeterX Enforcer is in blocking mode or not. This should be set to false initially |
6. After these Config Vars have been added, you can then add this buildpack to your application

## Using this buildpack

After adding the buildpack, `nginx` will be available on your path. But here comes the tricky part: Heroku sends HTTP traffic to a random port on any given dyno, indicated by the `PORT` environment variable. So in order for nginx to receive incoming requests, the `web` process in your `Procfile` needs to do the following:

1. Determine the current port.
1. Dynamically generate an `nginx.conf` file that listens on this port

  ```
  server {
    listen <%= ENV['PORT'] %>;
    ...
  }
  ```
1. Starts nginx with this generated config file

  ```
  nginx -c config/nginx.conf
  ```

Any application using this buildpack must handle all of this itself.


## Development

### Ubuntu

For maximum parity with Heroku's dynos, we recommend using Ubuntu or an Ubuntu virtual machine. See this article for all the details on Heroku's current stack:

https://devcenter.heroku.com/articles/stack

### OS X

In OS X, you can `bin/compile` locally like this:

```
BUILD_PREFIX=$PWD ./bin/compile $PWD/build $PWD/cache
```

If you try to build on OS X and have issues related to OpenSSL, make sure you've installed OpenSSL using Homebrew and then you could try the following command:
```
BUILD_PREFIX=$PWD NGINX_OPTIONS="--with-cc-opt=-I$(brew --prefix)/opt/openssl/include --with-ld-opt=-L$(brew --prefix)/opt/openssl/lib" bin/compile $PWD/build $PWD/cache
```
It's not the prettiest, but it _should_ sort out your issues. Basically, it provides an extra option to the nginx compilation which points it to the paths where OpenSSL is located, resolving the errors.


## Credits

This buildpack is based on ryandotsmith's [nginx-buildpack](https://github.com/ryandotsmith/nginx-buildpack). It has since been heavily adapted by PerimeterX in order to provide support for the Lua Enforcer Module

## Support

If there are any support related questions with this buildpack, contact [PerimeterX Support](mailto:support@perimeterx.com)
