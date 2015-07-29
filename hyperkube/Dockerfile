FROM google/debian:wheezy

RUN DEBIAN_FRONTEND=noninteractive apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get -yy -q \
    install \
    iptables \
    ca-certificates \
    && DEBIAN_FRONTEND=noninteractive apt-get autoremove -y \
    && DEBIAN_FRONTEND=noninteractive apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
COPY ./hyperkube.tmp /hyperkube
RUN chmod a+rx /hyperkube


COPY ./master-multi.json.tmp /etc/kubernetes/manifests-multi/master.json
COPY ./master.json.tmp /etc/kubernetes/manifests/master.json
COPY ./safe_format_and_mount.tmp /usr/share/google/safe_format_and_mount
RUN chmod a+rx /usr/share/google/safe_format_and_mount
