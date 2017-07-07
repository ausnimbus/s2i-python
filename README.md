# AusNimbus Builder for Python [![Build Status](https://travis-ci.org/ausnimbus/s2i-python.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-python) [![Docker Repository on Quay](https://quay.io/repository/ausnimbus/s2i-python/status "Docker Repository on Quay")](https://quay.io/repository/ausnimbus/s2i-python)

[![Python](https://user-images.githubusercontent.com/2239920/27286437-7b386556-5543-11e7-8483-91ae1cdd7c53.jpg)](https://www.ausnimbus.com.au/)

The [AusNimbus](https://www.ausnimbus.com.au/) builder for Python provides a fast, secure and reliable Django, Flask and [Python hosting](https://www.ausnimbus.com.au/languages/python-hosting/) environment.

This document describes the behaviour and environment configuration when running your Python apps on AusNimbus.

## Table of Contents

- [Runtime Environments](#runtime-environments)
- [Web Process](#web-process)
- [Dependency Management](#dependency-management)
- [Advanced](#advanced)
  - [Build Customization](#build-customization)
    - [Configuring pip](#configuring-pip)
  - [Using Gunicorn](#using-gunicorn)
    - [Configuring Gunicorn](#configuring-gunicorn)
    - [Application Concurrency](#application-concurrency)
    - [Forwarded Allow IPs](#forwarded-allow-ips)
  - [Using Django](#using-django)
    - [Serving static files](#serving-static-files)
    - [Disabling Collectstatic](#disabling-collectstatic)
- [Extending](#extending)
  - [Build Stage (assemble)](#build-stage-assemble)
  - [Runtime Stage (run)](#runtime-stage-run)
  - [Persistent Environment Variables](#persistent-environment-variables)
- [Debug Mode](#debug-mode)
- [Troubleshooting](#Troubleshooting)

## Runtime Environments

AusNimbus supports the major Python versions.

The currently supported versions are `2.7` and `3.6`

## Web Process

Your application's web processes must bind to port `8080`.

AusNimbus handles SSL termination at the load balancer.

This builder is optimized for web frameworks such as [Django](https://www.ausnimbus.com.au/apps/django/) and Flask.

The recommended webserver is Gunicorn and supports automatic optimization.

## Dependency Management

The builder uses the `pip` for installing dependencies.

If your Python application contains a `setup.py` file but excludes a `requirements.txt`.

`python setup.py develop` will be used to install your application and dependencies.

## Advanced

### Build Customization

If you have a `requirements.txt` file but would like to run `python setup.py develop`,
you can add the following to your `requirements.txt` file:

```
-e .
```

#### Configuring pip

By default `pip` will be executed with the version packaged in the current builder. If you would like to have `pip` upgraded to the latest version you may set the following environment variable:

NAME          | Description
--------------|-------------
PIP_UPGRADE   | Set to TRUE to upgrade pip before installing dependencies

If you would like to use a custom pip mirror you may use the following environment variable:

NAME          | Description
--------------|-------------
PIP_INDEX_URL | Define a custom pip mirror for downloading dependencies

### Using Gunicorn

The builder will attempt to detect your [Gunicorn GUNICORN_APP_MODULE](http://docs.gunicorn.org/en/latest/run.html#gunicorn) to run your application.

This variable specifies a WSGI callable with the pattern
`MODULE_NAME:VARIABLE_NAME`, where `MODULE_NAME` is the full dotted path
of a module, and `VARIABLE_NAME` refers to a WSGI callable inside the
specified module. Gunicorn will look for a WSGI callable named `application` if not specified.

The builder will attempt to look for a `wsgi.py` file in your project and use it if it exists.

If you are using `setup.py` for installing the application, the `MODULE_NAME` part can be read from there.

The auto detection may be undesirable, so it is recommended you set the following environment variable:

NAME                    | Description
------------------------|-------------
GUNICORN_APP_MODULE     | This variable specifies a WSGI callable with the pattern
`MODULE_NAME:VARIABLE_NAME`

#### Configuring Gunicorn

The builder provides a standard Gunicorn configuration however you may alternatively specify another location for your Gunicorn config:

Name            | Description
----------------|-------------
GUNICORN_CONFIG | Path to your
[Gunicorn configuration](http://docs.gunicorn.org/en/latest/configure.html#configuration-file) file.

#### Application Concurrency

If you are using the builder provided (fallback) Gunicorn config we automatically tune the number of [workers](http://docs.gunicorn.org/en/stable/settings.html#workers) for you based on the app instance size.

Size     | Value
---------|-------------
Small    | 2 Workers
Medium   | 4 Workers
Large    | 6 Workers
2xLarge  | 10 Workers

You may optionally change these default settings by configuring the following environment variable:

NAME             | Description
-----------------|-------------
WEB_CONCURRENCY  | Set the number of Gunicorn workers. By default, it is calculated as above.

If you are not using the default Gunicorn config the calculated `WEB_CONCURRENCY` variable is still made available to you.

#### Forwarded Allow IPs

On [AusNimbus](https://www.ausnimbus.com.au/) your application runs behind a load balancer. To ensure the correct client IP address is received by the application Gunicorn's ForwardedAllowIPS is set to `*`

You may optionally change these default settings by configuring the following environment variable:

NAME                 | Description
---------------------|-------------
FORWARDED_ALLOW_IPS  | Set this to change Gunicorn's ForwardedAllowIPS setting. (Default: `*`)

### Using Django

#### Serving static files

Django does not support serving static files in production. Instead it is recommended to integrate the [WhiteNoise](http://whitenoise.evans.io/en/stable/) package into your Django application.

#### Disabling Collectstatic

Django applications on AusNimbus automatically have `python manage.py collectstatic --noinput` run during the build stage.

If you want to disable the execution of `collectstatic` you may set the following environment variable:

NAME                  | Description
----------------------|-------------
DISABLE_COLLECTSTATIC | Set to "TRUE" to disable the automatic execution of `collectstatic`

## Extending

AusNimbus builders are split into two stages:

- Build
- Runtime

Both stages are completely extensible, allowing you to customize or completely overwrite each stage.

### Build Stage (assemble)

If you want to customize the build stage, you need to add the executable `.s2i/bin/assemble` file in your repository.

This file should contain the logic required to build and install any dependencies your application requires.

If you only want to extend the build stage, you may use this example:

```sh
#!/bin/bash

echo "Logic to include before"

# Run the default builder logic
. /usr/libexec/s2i/assemble

echo "Logic to include after"
```

### Runtime Stage (run)

If you only want to change the executed command for the run stage you may the following environment variable.

NAME        | Description
------------|-------------
APP_RUN     | Define a custom command to start your application. eg. `python wsgi.py`

**NOTE:** `APP_RUN` will overwrite any builder's runtime configuration (including the [Debug Mode](#Debug Mode) section)

Alternatively you may customize or overwrite the entire runtime stage by including the executable file `.s2i/bin/run`

This file should contain the logic required to execute your application.

If you only want to extend the run stage, you may use this example:

```sh
#!/bin/bash

echo "Logic to include before"

# Run the default builder logic
. /usr/libexec/s2i/run
```

As the run script executes every time your application is deployed, scaled or restarted it's recommended to keep avoid including complex logic which may delay the start-up process of your application.

### Persistent Environment Variables

The recommend approach is to set your environment variables in the AusNimbus dashboard.

However it is possible to store environment variables in code using the `.s2i/environment` file.

The file expects a key=value format eg.

```
KEY=VALUE
FOO=BAR
```

## Troubleshooting

### ERROR: don't know how to run your application

The builder will attempt to guess how it can start your web process only if these conditions are met:

  - Execute the command defined in environment variable `APP_RUN`
  - Have a valid `wsgi.py` file for Gunicorn **or**
  - Have a valid `app.py` file

As none of these conditions are met, the easiest way will be to specify your launch command with `APP_RUN`
