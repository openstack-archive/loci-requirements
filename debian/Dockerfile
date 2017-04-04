FROM debian:jessie

ARG OVERRIDE=override

ADD $OVERRIDE /

RUN set -x \
    # NOTE(SamYaple): backports is only used for liberasurecode-dev
    && echo "deb http://deb.debian.org/debian jessie-backports main" > /etc/apt/sources.list.d/backports.list \
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
        libxml2-dev \
        libxslt1-dev \
        libvirt-dev \
        libyaml-dev \
        libz-dev \
        pkg-config \
        python \
        python-dev \
    && curl -sSL https://bootstrap.pypa.io/get-pip.py -o get-pip.py \
    && python get-pip.py \
    && rm get-pip.py \
    && mkdir /root/packages \
    && curl -sSL https://raw.githubusercontent.com/openstack/requirements/master/global-requirements.txt > /root/packages/global-requirements.txt \
    && curl -sSL https://raw.githubusercontent.com/openstack/requirements/master/upper-constraints.txt > /root/packages/upper-constraints.txt \
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
        libxml2-dev \
        libxslt1-dev \
        libvirt-dev \
        libyaml-dev \
        libz-dev \
        pkg-config \
        python-dev \
    && rm -rf /var/lib/apt/lists/* /tmp/* /root/.cache
