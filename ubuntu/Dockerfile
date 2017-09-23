ARG FROM=ubuntu:xenial
FROM ${FROM}

ENV PROJECT=requirements \
    PATH=/builder/bin:$PATH
ARG PROJECT_REPO=https://git.openstack.org/openstack/${PROJECT}
ARG PROJECT_REF=master
ARG OVERRIDE=override

ADD $OVERRIDE /

RUN set -x \
    && apt-get update \
    && apt-get dist-upgrade -y \
    && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        git \
        liberasurecode-dev \
        libffi-dev \
        libldap2-dev \
        libmysqlclient-dev \
        libnss3-dev \
        libpq-dev \
        libsasl2-dev \
        libssl-dev \
        libsystemd-dev \
        libxml2-dev \
        libxslt1-dev \
        libvirt-dev \
        libyaml-dev \
        libz-dev \
        pkg-config \
        python \
        python-dev \
        python-pip \
        python-virtualenv \
    && mkdir /wheels/ \
    && git init /tmp/requirements \
    && git --git-dir /tmp/requirements/.git fetch --depth 1 $PROJECT_REPO $PROJECT_REF \
    && git --work-tree /tmp/requirements --git-dir /tmp/requirements/.git checkout FETCH_HEAD \
    && mv /tmp/requirements/global-requirements.txt /tmp/requirements/upper-constraints.txt /wheels/ \
    && python -m virtualenv /builder \
    && pip install -U pip \
    && pip install -U wheel setuptools \
    && pip wheel -w /wheels/ -r /wheels/global-requirements.txt -c /wheels/upper-constraints.txt \
        bindep==2.5.0 \
        uwsgi \
    && find . -mindepth 1 -maxdepth 1 -path /wheels -prune -o -exec rm -rf {} + || :

FROM scratch
COPY --from=0 /wheels/* /
