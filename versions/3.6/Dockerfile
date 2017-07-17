#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "./hack/run update"
#
# DO NOT EDIT IT DIRECTLY.
#
FROM python:3.6
# <pre>

MAINTAINER AusNimbus <support@ausnimbus.com.au>

LABEL summary="Platform for running Python 3.6 applications" \
      maintainer="AusNimbus <support@ausnimbus.com.au>" \
      io.k8s.description="Platform for running Python 3.6 applications" \
      io.k8s.display-name="Python 3.6"

ENV USER=999 \
    HOME=/opt/app-root/src

USER 0

RUN apt-get update && apt-get install -y \
      rsync nano vim curl && \
      rm -rf /var/lib/apt/lists/* && \
    ln -sf /bin/bash /bin/sh && \
    mkdir -p $HOME && \
    useradd -u 1001 -g 0 -d $HOME -s /sbin/nologin \
      -c "Default Application User" default && \
    chown -R 1001:0 /opt/app-root

# Copy common executable utilities.
COPY ./common/bin/ /usr/bin/

# Each container image can have 'contrib' a directory with extra files needed to
# run and build the application.
COPY ./contrib/ /opt/app-root

# </pre>
# <s2i>

RUN apt-get update && apt-get install -y git --no-install-recommends && rm -rf /var/lib/apt/lists/*

ENV STI_SCRIPTS_PATH=/usr/libexec/s2i

LABEL io.openshift.expose-services="8080:http" \
    io.openshift.tags="builder,python" \
    io.openshift.s2i.scripts-url=image:///usr/libexec/s2i

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Directory with the sources is set as the working directory so all S2I scripts
# can execute relative to this path.
WORKDIR ${HOME}

EXPOSE 8080
CMD /usr/bin/base-usage

# </s2i>
# <template>

RUN apt-get update && apt-get install -yq \
      wget \
      locales \
      # Dependenies for third-party Python packages
      # with C-extensions
      build-essential \
      libcurl4-openssl-dev \
      libffi-dev \
      libjpeg-dev \
      libmysqlclient-dev \
      libpng12-dev \
      libpq-dev \
      libssl-dev \
      libxml2-dev \
      libxslt1-dev \
      swig \
      zlib1g-dev \
      # Needed by scipy/numpy
      gfortran \
      libatlas-dev \
      libblas-dev \
      libfreetype6-dev \
      liblapack-dev \
      libquadmath0 \
      # Needed by pylibmc
      libmemcached-dev \
      libsasl2-2 \
      libsasl2-dev \
      libsasl2-modules \
      sasl2-bin \
      # Apache for mod_wsgi
      apache2 \
      apache2-dev && \
    rm -rf /var/lib/apt/lists/*

ENV PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off \
    LIBRARY_PATH=/lib:/usr/lib

# Fix locale
RUN localedef -i en_US -f UTF-8 en_US.UTF-8

# Create a Python virtual environment for use by any application to avoid
# potential conflicts with Python packages preinstalled in the main Python
# installation.
RUN pip install virtualenv && \
    virtualenv /opt/app-root

# </template>
# <post>

# Standardize on UID $USER
RUN usermod -o -u $USER default

# Drop the root user and make the content of /opt/app-root owned by user default
RUN chown -R $USER:0 /opt/app-root && chmod -R ug+rwx /opt/app-root
USER $USER

ENTRYPOINT ["container-entrypoint"]

# </post>
