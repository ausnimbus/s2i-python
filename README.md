# Python S2I Docker images

[![Build Status](https://travis-ci.org/ausnimbus/s2i-python.svg?branch=master)](https://travis-ci.org/ausnimbus/s2i-python)

This repository contains the source for building various versions of
the Python application as a reproducible Docker image
[source-to-image](https://github.com/openshift/source-to-image)
to be run on [AusNimbus](https://www.ausnimbus.com.au/).

Images are built with Python binaries from python.org
The resulting image can be run using [Docker](http://docker.io).

If you are interested in using SCL-based Python binaries, use [s2i-Python-scl](https://github.com/ausnimbus/s2i-python-scl)

## Versions

The versions currently supported are:

- 2.7
- 3.6
