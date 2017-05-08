#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "./hack/run update"
#
# DO NOT EDIT IT DIRECTLY.
#
# <pre>

FROM python:2.7-alpine
MAINTAINER AusNimbus <support@ausnimbus.com.au>

LABEL summary="Platform for running Python 2.7 applications" \
      maintainer="AusNimbus <support@ausnimbus.com.au>" \
      io.k8s.description="Platform for running Python 2.7 applications" \
      io.k8s.display-name="Python 2.7"

ENV HOME=/opt/app-root/src

RUN apk --no-cache add bash rsync git && rm -rf /var/cache/apk/* && \
    mkdir -p ${HOME}/.pki/nssdb && \
    chown -R 1001:0 ${HOME}/.pki && \
    adduser -S -u 1001 -G root -h ${HOME} -s /sbin/nologin \
      -g "Default Application User" default && \
    chown -R 1001:0 /opt/app-root

# Copy common executable utilities.
COPY ./common/bin/ /usr/bin/

# Each container image can have 'contrib' a directory with extra files needed to
# run and build the application.
COPY ./contrib/ /opt/app-root

# </pre>
# <s2i>

EXPOSE 8080

ENV STI_SCRIPTS_PATH=/usr/libexec/s2i

LABEL io.openshift.expose-services="8080:http" \
    io.openshift.tags="builder,python" \
    io.openshift.s2i.scripts-url=image:///usr/libexec/s2i

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Directory with the sources is set as the working directory so all S2I scripts
# can execute relative to this path.
WORKDIR ${HOME}

CMD /usr/bin/base-usage

# </s2i>
# <template>

ENV PATH=$HOME/.local/bin/:$PATH \
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off \
    LIBRARY_PATH=/lib:/usr/lib

RUN apk add --no-cache \
      build-base \
      libxml2-dev \
      libxslt-dev \
      postgresql-dev \
      mariadb-dev \
      linux-headers \
      libtool \
      libffi-dev \
      apache2 \
      apache2-dev && \
    rm -rf /tmp/* && \
    rm -rf /var/cache/apk/* && \
    # Fix numpy
    ln -s /usr/include/locale.h /usr/include/xlocale.h

# Create a Python virtual environment for use by any application to avoid
# potential conflicts with Python packages preinstalled in the main Python
# installation.
RUN pip install virtualenv && \
    virtualenv /opt/app-root

# </template>
# <post>

# Drop the root user and make the content of /opt/app-root owned by user 1001
RUN chown -R 1001:0 /opt/app-root && chmod -R ug+rwx /opt/app-root
USER 1001

ENTRYPOINT ["container-entrypoint"]

# </post>
