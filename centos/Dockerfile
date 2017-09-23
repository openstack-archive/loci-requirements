ARG FROM=centos:7
FROM ${FROM}

ENV PROJECT=requirements \
    PATH=/builder/bin:$PATH
ARG PROJECT_REPO=https://git.openstack.org/openstack/${PROJECT}
ARG PROJECT_REF=master
ARG OVERRIDE=override

ADD $OVERRIDE /

RUN set -x \
    && yum upgrade -y \
    # NOTE(SamYaple): https://bugs.centos.org/view.php?id=10750
    && yum install -y --setopt=tsflags=docs libffi-devel \
    && yum install -y  \
        gcc \
        gcc-c++ \
        make \
        openssl-devel \
        ca-certificates \
        git \
        bzip2 \
        liberasurecode-devel \
        openldap-devel \
        mariadb-devel \
        nss-devel \
        postgresql-devel \
        cyrus-sasl-devel \
        openssl-devel \
        libxml2-devel \
        libxslt-devel \
        libvirt-devel \
        libyaml-devel \
        zlib-devel \
        pkgconfig \
        python \
        python-devel \
        python-pip \
        python-virtualenv \
        libgcrypt \
        nss-util \
        systemd-devel \
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
    && find . -mindepth 1 -maxdepth 1 -path ./wheels -prune -o -exec rm -rf {} + 2> /dev/null || :

FROM scratch
COPY --from=0 /wheels/* /
