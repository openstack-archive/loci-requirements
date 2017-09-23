ARG FROM=debian:jessie-slim
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
        libkrb5-dev \
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
        python-dev \
        python-pip \
        python-virtualenv \
    && mkdir /root/packages \
    && git init /tmp/requirements \
    && git --git-dir /tmp/requirements/.git fetch --depth 1 $PROJECT_REPO $PROJECT_REF \
    && git --work-tree /tmp/requirements --git-dir /tmp/requirements/.git checkout FETCH_HEAD \
    && mv /tmp/requirements/global-requirements.txt /tmp/requirements/upper-constraints.txt /root/packages/ \
    && python -m virtualenv /builder \
    && pip install -U pip \
    && pip install -U wheel setuptools \
    && pip wheel -w /root/packages/ -r /root/packages/global-requirements.txt -c /root/packages/upper-constraints.txt \
        bindep==2.5.0 \
        uwsgi \
    && apt-get purge -y --auto-remove \
        build-essential \
        ca-certificates \
        git \
        liberasurecode-dev \
        libffi-dev \
        libkrb5-dev \
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
        python-dev \
    && rm -rf /var/lib/apt/lists/* /tmp/* /root/.cache

FROM scratch
COPY --from=0 /root/packages/* /
