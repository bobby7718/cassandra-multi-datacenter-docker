#
# Based of abh1nav/cassandra with modifications to work in docker-compose
#
FROM abh1nav/opscenter

ADD . /opscenter_src

RUN \
    mkdir -p /opt/opscenter/conf/cluster; \
    cp /opscenter_src/cluster.conf /opt/opscenter/conf/cluster/;
    
