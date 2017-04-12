FROM ubuntu:xenial

ENV PROJECT=requirements
ARG PROJECT_REPO=https://git.openstack.org/openstack/${PROJECT}
ARG PROJECT_REF=master
ARG OVERRIDE=override

ADD $OVERRIDE /

RUN set -x \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        curl \
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
    && mkdir /root/packages \
    && git init /tmp/requirements \
    && git --git-dir /tmp/requirements/.git fetch --depth 1 $PROJECT_REPO $PROJECT_REF \
    && git --work-tree /tmp/requirements --git-dir /tmp/requirements/.git checkout FETCH_HEAD \
    && mv /tmp/requirements/global-requirements.txt /tmp/requirements/upper-constraints.txt /root/packages/ \
    # NOTE(SamYaple): There is a bug with python-nss, this is a workaround. https://bugzilla.redhat.com/show_bug.cgi?id=1389739
    && sed -i '/dogtag-pki/d' /root/packages/global-requirements.txt \
    && pip install -U pip wheel setuptools \
    && pip download -d /tmp -c /root/packages/upper-constraints.txt python-nss \
    && mkdir /tmp/python-nss \
    && tar xf /tmp/python-nss-*.tar.bz2 -C /tmp/python-nss --strip-components=1 \
    && sed -i "s/if arg in ('-d', '--debug'):/if arg == '--debug':/g" /tmp/python-nss/setup.py \
    && pip wheel -w /root/packages/ $(grep dogtag-pki /root/packages/upper-constraints.txt) /tmp/python-nss/ \
    # NOTE(SamYaple): end bug workaround
    && pip wheel -w /root/packages/ -r /root/packages/global-requirements.txt -c /root/packages/upper-constraints.txt \
        uwsgi \
    && apt-get purge -y --auto-remove \
        build-essential \
        ca-certificates \
        curl \
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
        python-dev \
    && rm -rf /var/lib/apt/lists/* /tmp/* /root/.cache
