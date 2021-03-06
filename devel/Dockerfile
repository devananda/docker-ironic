FROM ubuntu

MAINTAINER Devananda van der Veen <devananda.vdv@gmail.com>

EXPOSE 6385
EXPOSE 35357

# Install all system prerequisites
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        libssl-dev libffi-dev libxslt1-dev libzmq1 \
        redis-server gettext sqlite supervisor git autoconf libtool pkg-config \
        python2.7 python-setuptools python-dev python-pip python-zmq python-redis && \
    apt-get -y autoremove && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Install necessary python packages from PIP
RUN pip install -U pip wheel setuptools

# Clone ironic and keystone & install in editable mode
RUN mkdir /opt/source && cd /opt/source && \
    git clone --depth=1 https://github.com/openstack/ironic && \
    git clone --depth=1 https://github.com/openstack/keystone && \
    cd /opt/source/ironic && pip install -e . && \
    cd /opt/source/keystone && pip install -e . && \
    pip install -U python-ironicclient python-openstackclient && \
    rm -rf /root/.cache/pip/*

# Remove packages we won't need after the installation
# Saves about
RUN apt-get -y remove m4 make perl autotools-dev cpp cpp-5 gcc gcc-5 git-man && \
    apt-get -y autoremove

# Create service users and relevant directories
RUN useradd --user-group stack && \
    mkdir -p /etc/ironic /etc/keystone \
             /var/log/ironic /var/log/keystone /var/run/openstack /var/log/supervisord \
             /tftpboot /httpboot && \
    chown -R stack:stack /etc/ironic /var/log/ironic /etc/keystone /var/log/keystone /var/run/openstack \
             /tftpboot /httpboot

# Copy config files
RUN cp -rp /opt/source/ironic/etc/ironic/* /etc/ironic/ && \
    cp -rp /opt/source/keystone/etc/* /etc/keystone/

COPY config/ironic.conf /etc/ironic/ironic.conf
COPY config/keystone.conf /etc/keystone/keystone.conf
COPY config/supervisord.conf /etc/supervisord.conf
COPY register.sh /etc/init.d/

# Initialize databases
RUN ironic-dbsync create_schema && \
    chown stack:stack /etc/ironic/ironic.sqlite && \
    keystone-manage db_sync && \
    chown stack:stack /etc/keystone/keystone.sqlite

# Initialize keystone accounts
RUN OS_BOOTSTRAP_USERNAME=admin \
    OS_BOOTSTRAP_PASSWORD=admin_pass \
    OS_BOOTSTRAP_PROJECT_NAME=demo \
    OS_BOOTSTRAP_ADMIN_URL=http://127.0.0.1:35357/v3 \
    OS_BOOTSTRAP_PUBLIC_URL=http://127.0.0.1:35357/v3 \
    OS_BOOTSTRAP_INTERNAL_URL=http://127.0.0.1:35357/v3 \
    OS_BOOTSTRAP_SERVICE_NAME=keystone \
    keystone-manage bootstrap


#ENTRYPOINT ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
