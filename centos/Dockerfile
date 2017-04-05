FROM centos:7

ENV PROJECT=requirements
ARG PROJECT_REPO=https://git.openstack.org/openstack/${PROJECT}
ARG PROJECT_REF=master
ARG OVERRIDE=override

ADD $OVERRIDE /

ENV OPENSTACK_VERSION=newton

RUN set -x \
    # NOTE(Pete Birley): CentOS-OpenStack repo is only used for liberasurecode-devel
    && curl -sSL https://raw.githubusercontent.com/rdo-infra/centos-release-openstack/${OPENSTACK_VERSION}-rdo/CentOS-OpenStack.repo > /etc/yum.repos.d/CentOS-OpenStack.repo \
    && sed -i "s/OPENSTACK_VERSION/${OPENSTACK_VERSION}/g" /etc/yum.repos.d/CentOS-OpenStack.repo \
    && sed -i "/\[centos-openstack-${OPENSTACK_VERSION}\]/s/.*/&\nincludepkgs=liberasurecode*/" /etc/yum.repos.d/CentOS-OpenStack.repo \
    && curl -sSL https://raw.githubusercontent.com/rdo-infra/centos-release-openstack/newton-rdo/RPM-GPG-KEY-CentOS-SIG-Cloud > /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-Cloud \
    # NOTE(Pete Birley): libffi-devel does not like being uninstalled on CentOS so we install it seperatly, all of its dependencies are already installed in the docker base image.
    && yum install --setopt=tsflags=nodocs -y \
        libffi-devel \
    && yum install --setopt=tsflags=nodocs -y  \
        gcc \
        gcc-c++ \
        make \
        openssl-devel \
        ca-certificates \
        curl \
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
        libgcrypt \
        nss-util \
    && curl -sSL https://bootstrap.pypa.io/get-pip.py -o get-pip.py \
    && python get-pip.py \
    && rm get-pip.py \
    && mkdir /root/packages \
    && git init /tmp/requirements \
    && git --git-dir /tmp/requirements/.git fetch --depth 1 $PROJECT_REPO $PROJECT_REF \
    && git --work-tree /tmp/requirements --git-dir /tmp/requirements/.git checkout FETCH_HEAD \
    && mv /tmp/requirements/global-requirements.txt /tmp/requirements/upper-constraints.txt /root/packages/ \
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
    && yum history -y undo $(yum history list gcc | tail -2 | head -1 | awk '{ print $1}') \
    && yum clean all \
    && rm -rf /tmp/* /root/.cache
