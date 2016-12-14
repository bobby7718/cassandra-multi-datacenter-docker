#
# Based of abh1nav/cassandra with modifications to work in docker-compose
#
FROM abh1nav/cassandra

ADD . /cassandra_src

RUN \
    cp -rf /cassandra_src/cassandra.yaml /opt/cassandra/conf/; \
    cp /cassandra_src/address.yaml  /opt/agent/conf/; \
    cp /cassandra_src/agent-run     /etc/service/agent/run; \
    cp /cassandra_src/cassandra-run /etc/service/cassandra/run;
    
