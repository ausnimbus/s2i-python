# AusNimbus Builder for Python [![Build Status](https://travis-ci.org/ausnimbus/s2i-python.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-python) [![Docker Repository on Quay](https://quay.io/repository/ausnimbus/s2i-python/status "Docker Repository on Quay")](https://quay.io/repository/ausnimbus/s2i-python)

[![Python](https://user-images.githubusercontent.com/2239920/27286437-7b386556-5543-11e7-8483-91ae1cdd7c53.jpg)](https://www.ausnimbus.com.au/)

The [AusNimbus](https://www.ausnimbus.com.au/) builder for Python provides a fast, secure and reliable Django, Flask and [Python hosting](https://www.ausnimbus.com.au/languages/python-hosting/) environment.

It uses pip for dependency management.

This builder is optimized for web frameworks such as [Django](https://www.ausnimbus.com.au/apps/django/) and Flask.
The recommended webserver is Gunicorn. Web processes must bind to port `8080` and only the HTTP protocol is permitted for incoming connections.

For your application to run you need to either have:

  - Define the environment variable `APP_RUN` **or** `GUNICORN_APP_MODULE` **or**
  - Have a valid `wsgi.py` file for Gunicorn **or**
  - Have a valid `app.py` file

## Environment variables

* **APP_RUN**

    Define the application command to be run. This can be a command to start the application.

    **NOTE:** This overwrites any builder dynamic run configuration.

* **PIP_INDEX_URL**

    Set this variable to use a custom index URL or mirror to download required packages during build process. This only affects packages listed in requirements.txt.

* **PIP_UPGRADE**

    Set this variable to TRUE to have the 'pip' program upgraded
    to the most recent version before any Python packages are installed. If not
    set it will use whatever the default version is included by the platform
    for the Python version being used.

If you are using Gunicorn, you may use the following environment variables.

* **GUNICORN_APP_MODULE**

    Used to run the application with [Gunicorn](http://docs.gunicorn.org/en/latest/run.html#gunicorn)

    This variable specifies a WSGI callable with the pattern
    `MODULE_NAME:VARIABLE_NAME`, where `MODULE_NAME` is the full dotted path
    of a module, and `VARIABLE_NAME` refers to a WSGI callable inside the
    specified module. Gunicorn will look for a WSGI callable named `application` if not specified.

    If `GUNICORN_APP_MODULE` is not provided:

    The builder will attempt to look for a `wsgi.py` file in your project and use it if it exists.
    If you are using `setup.py` for installing the application, the `MODULE_NAME` part can be read from there.

* **GUNICORN_CONFIG**

    Optional path to your
    [Gunicorn configuration](http://docs.gunicorn.org/en/latest/configure.html#configuration-file) file.

* **WEB_CONCURRENCY**

    Set this to change the default setting for the number of
    [workers](http://docs.gunicorn.org/en/stable/settings.html#workers). By
    default, this is auto configured based on the `MEMORY_LIMIT`

* **FORWARDED_ALLOW_IPS**

    Set this to change Gunicorn's ForwardedAllowIPS setting. On [AusNimbus](https://www.ausnimbus.com.au/) your
    application runs behind a loadbalancer, so by default, this is set to `*`
    This ensures the correct client IP address is received by the application.

If you are using [Django](https://www.ausnimbus.com.au/apps/django/), you may use the following environment variables.

* **DISABLE_COLLECTSTATIC**

    Set this variable to "TRUE" to disable the execution of 'manage.py collectstatic' during the build process.

## Traditional Applications

If your Python application contains a `setup.py` file but excludes a `requirements.txt`. `python setup.py develop`
will be used to install your application and dependencies.

If you have a `requirements.txt` file but would like to run `python setup.py develop`,
you can add the following to your `requirements.txt` file:

```
-e .
```

## Versions

The versions currently supported are:

- 2.7
- 3.6
