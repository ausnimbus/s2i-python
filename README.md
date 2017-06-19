# AusNimbus Builder for Python [![Build Status](https://travis-ci.org/ausnimbus/s2i-python.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-python) [![Docker Repository on Quay](https://quay.io/repository/ausnimbus/s2i-python/status "Docker Repository on Quay")](https://quay.io/repository/ausnimbus/s2i-python)

[![Python](https://user-images.githubusercontent.com/2239920/27286437-7b386556-5543-11e7-8483-91ae1cdd7c53.jpg)](https://www.ausnimbus.com.au/)

[AusNimbus](https://www.ausnimbus.com.au/) builder for Python provides a fast, secure and reliable Django, Flask and [Python hosting](https://www.ausnimbus.com.au/languages/python-hosting/) environment.

This builder is optimized for web frameworks such as [Django](https://www.ausnimbus.com.au/apps/django/) and Flask.
The recommended webserver is Gunicorn. Web processes must bind to port `8080`,
and only the HTTP protocol is permitted for incoming connections.

For your application to run you need to either have:

  - Define the environment variable `APP_RUN` **or** `APP_MODULE` **or**
  - Have a valid `wsgi.py` file for Gunicorn **or**
  - Have a valid `app.py` file

## Environment variables

* **APP_RUN**

    Used to define the application file to run. This should be a command to start the application. eg.
    `python app.py` or `gunicorn myapp.wsgi --bind=0.0.0.0:8080 --forwarded-allow-ips="*" --access-logfile=-`

* **APP_MODULE**

    Used to run the application with Gunicorn, as documented
    [here](http://docs.gunicorn.org/en/latest/run.html#gunicorn).
    This variable specifies a WSGI callable with the pattern
    `MODULE_NAME:VARIABLE_NAME`, where `MODULE_NAME` is the full dotted path
    of a module, and `VARIABLE_NAME` refers to a WSGI callable inside the
    specified module.
    Gunicorn will look for a WSGI callable named `application` if not specified.

    If `APP_MODULE` is not provided, the `run` script will look for a `wsgi.py`
    file in your project and use it if it exists.

    If using `setup.py` for installing the application, the `MODULE_NAME` part
    can be read from there.

* **APP_HOME**

    This variable can be used to specify a sub-directory in which the application to be run is contained.
    The directory pointed to by this variable needs to contain `wsgi.py` (for Gunicorn) or `manage.py` (for Django).

    If `APP_HOME` is not provided, the `assemble` and `run` scripts will use the application's root directory.

* **PIP_INDEX_URL**

    Set this variable to use a custom index URL or mirror to download required packages
    during build process. This only affects packages listed in requirements.txt.

* **PIP_UPGRADE**

    Set this variable to TRUE to have the 'pip' program upgraded
    to the most recent version before any Python packages are installed. If not
    set it will use whatever the default version is included by the platform
    for the Python version being used.

If you are using Gunicorn, you may use the following environment variables.

* **APP_CONFIG**

    Path to a valid Python file with a
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

    Set this variable to "TRUE" to disable the execution of
    'manage.py collectstatic' during the build process.

## Versions

The versions currently supported are:

- 2.7
- 3.6
