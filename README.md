# Python S2I Builder

[![Build Status](https://travis-ci.org/ausnimbus/s2i-python.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-python)
[![Docker Repository on Quay](https://quay.io/repository/ausnimbus/s2i-python/status "Docker Repository on Quay")](https://quay.io/repository/ausnimbus/s2i-python)

This repository contains the source for the [source-to-image](https://github.com/openshift/source-to-image)
builders used to deploy [Python applications](https://www.ausnimbus.com.au/languages/python/)
on [AusNimbus](https://www.ausnimbus.com.au/).

## Environment variables
---------------------

The following ENV variables are made available:

* **APP_SCRIPT**

    Used to run the application from a script file.
    This should be a path to a script file (defaults to `app.sh`) that will be
    run to start the application.

* **APP_FILE**

    Used to run the application from a Python script.
    This should be a path to a Python file (defaults to `app.py`) that will be
    passed to the Python interpreter to start the application.

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
    can be read from there. For an example, see
    [setup-test-app](https://github.com/openshift/s2i-python/tree/master/3.5/test/setup-test-app).

* **APP_HOME**

    This variable can be used to specify a sub-directory in which the application to be run is contained.
    The directory pointed to by this variable needs to contain `wsgi.py` (for Gunicorn) or `manage.py` (for Django).

    If `APP_HOME` is not provided, the `assemble` and `run` scripts will use the application's root
    directory.

* **APP_CONFIG**

    Path to a valid Python file with a
    [Gunicorn configuration](http://docs.gunicorn.org/en/latest/configure.html#configuration-file) file.

* **DISABLE_COLLECTSTATIC**

    Set this variable to a non-empty value to inhibit the execution of
    'manage.py collectstatic' during the build. This only affects Django projects.

* **DISABLE_MIGRATE**

    Set this variable to a non-empty value to inhibit the execution of 'manage.py migrate'
    when the produced image is run. This only affects Django projects.

* **PIP_INDEX_URL**

    Set this variable to use a custom index URL or mirror to download required packages
    during build process. This only affects packages listed in requirements.txt.

* **UPGRADE_PIP_TO_LATEST**

    Set this variable to a non-empty value to have the 'pip' program be upgraded
    to the most recent version before any Python packages are installed. If not
    set it will use whatever the default version is included by the platform
    for the Python version being used.

* **WEB_CONCURRENCY**

    Set this to change the default setting for the number of
    [workers](http://docs.gunicorn.org/en/stable/settings.html#workers). By
    default, this is set to the number of available cores times 2.

## Versions

The versions currently supported are:

- 2.7
- 3.6

## Variants

Two different variants are made available:

- Default
- Alpine
  - mod_wsgi is not supported on the Alpine variant
  - uwsgi is not supported on Python 2.7 Alpine variant
