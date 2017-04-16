FROM centos:7

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
    && mkdir /root/packages \
    && git init /tmp/requirements \
    && git --git-dir /tmp/requirements/.git fetch --depth 1 $PROJECT_REPO $PROJECT_REF \
    && git --work-tree /tmp/requirements --git-dir /tmp/requirements/.git checkout FETCH_HEAD \
    && mv /tmp/requirements/global-requirements.txt /tmp/requirements/upper-constraints.txt /root/packages/ \
    && python -m virtualenv /builder \
    && pip install -U pip \
    && pip install -U wheel setuptools \
    # NOTE(SamYaple): There is a bug with python-nss, this is a workaround. https://bugzilla.redhat.com/show_bug.cgi?id=1389739
    && sed -i '/dogtag-pki/d' /root/packages/global-requirements.txt \
    && pip download -d /tmp -c /root/packages/upper-constraints.txt python-nss \
    && mkdir /tmp/python-nss \
    && tar xf /tmp/python-nss-*.tar.bz2 -C /tmp/python-nss --strip-components=1 \
    && sed -i "s/if arg in ('-d', '--debug'):/if arg == '--debug':/g" /tmp/python-nss/setup.py \
    && pip wheel -w /root/packages/ $(grep dogtag-pki /root/packages/upper-constraints.txt) /tmp/python-nss/ \
    # NOTE(SamYaple): end bug workaround
    && pip wheel -w /root/packages/ -r /root/packages/global-requirements.txt -c /root/packages/upper-constraints.txt \
        uwsgi \
    && yum history -y undo 3 \
    && yum clean all \
    && rm -rf /tmp/* /root/.cache
