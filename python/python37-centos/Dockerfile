# Python 3.7 builder ===========================================================
ARG CENTOS_VERSION=7.7.1908
FROM centos:$CENTOS_VERSION AS builder

# Set utf-8 encoding for Python
RUN localedef --quiet -c -i en_US -f UTF-8 en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib

# Install development tools
RUN yum -y update && \
    yum install -y epel-release && \
    yum install -y \
        curl \
        gcc \
        hostname \
        libffi \
        make \
        openssl \
        perl \
        sudo \
        tar \
        which && \
    yum install -y \
        bzip2-devel \
        libffi-devel \
        libxml2-devel \
        openssl-devel \
        sqlite-devel \
        zlib-dev && \
    yum clean -y all

# Install openssl
RUN cd /tmp && \
    curl -s -O -J -L https://www.openssl.org/source/openssl-1.1.1h.tar.gz && \
    tar -xf openssl-1.1.1h.tar.gz && \
    cd openssl-1.1.1h && \
    ./config shared --prefix=/usr/local/ssl --openssldir=/usr/local/ssl && \
    make && \
    make install

# Download and install Python 3.7
RUN cd /tmp && \
    curl -s -O -J -L https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tgz && \
    tar xzf Python-3.7.0.tgz -C /usr/src/ && \
    cd /usr/src/Python-3.7.0 && \
    # Fix SSL headers
    sed -i '211,214 s/^##*//' Modules/Setup.dist && \
    LDFLAGS="-Wl,-rpath=/usr/local/ssl/lib" \
    /usr/src/Python-3.7.0/configure \
        --enable-optimizations \
        --enable-shared \
        --with-system-ffi \
        --prefix=/usr/local \
        --with-openssl=/usr/local/ssl && \
    make altinstall && \
    ln -s /usr/local/bin/python3.7 /usr/bin/python3.7 && \
    echo "/usr/local/lib" >> /etc/ld.so.conf && \
    ldconfig && \
    pip3.7 install --upgrade pip pipenv && \
    rm /tmp/Python-3.7.0.tgz

# Setup python venv
COPY ./requirements.txt /opt
WORKDIR /opt/py37venv
RUN PIPENV_VENV_IN_PROJECT=1 \
    pipenv --python 3.7 install --deploy --skip-lock -r /opt/requirements.txt


# Python 3.7 container =========================================================
FROM centos:$CENTOS_VERSION AS py37app

# Set utf-8 encoding for Python
RUN localedef --quiet -c -i en_US -f UTF-8 en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV PIP_DISABLE_PIP_VERSION_CHECK=1
ENV PATH="/usr/local/bin:$PATH"
ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib
WORKDIR /opt/py37venv

# Copy over the compiled stuff
COPY --from=builder /usr/local /usr/local
COPY --from=builder /opt/py37venv /opt/py37venv
RUN echo "/usr/local/lib" >> /etc/ld.so.conf && \
    ldconfig

CMD ["/bin/bash", "-c", "(pipenv shell)"]
