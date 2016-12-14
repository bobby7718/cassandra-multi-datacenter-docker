# Multi datacenter Cassandra cluster with Docker-compose

## Overview

This is a multi datacenter dockerized Cassandra cluster inspired by [cassandra-docker-compose](https://github.com/jdgoldie/cassandra-docker-compose) with the following improvements.

* use Docker embedded dns discovery for server name discovery 
* docker-compose Compose file version updated to version 2
* Ops Center is now supported - work in progress
* sample applications demonstrating failover of cluster doesn't impact write/read. - work in progress

## Prerequisites

* Docker 1.10+ for Embedded DNS server
* Docker-compose

```
$ docker -v
Docker version 1.12.3, build 6b644ec

$ docker-compose -version
docker-compose version 1.8.1, build 878cff1
```


## The Docker Image

The docker image is based off [abh1nav/cassandra](https://registry.hub.docker.com/u/abh1nav/cassandra/) 
with changes to support multiple data centers.  

## Try it out

### Start Cluster

Starting this cluster is as simple as 

```
docker-compose -p cluster up -d 
```

The `-p cluster` specifies the cluster name. This command should start the following:

```
      Name             Command      State                                                 Ports                                                
----------------------------------------------------------------------------------------------------------------------------------------------
cluster_nodedc1_1   /sbin/my_init   Up      7000/tcp, 7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                                                   
cluster_nodedc2_1   /sbin/my_init   Up      7000/tcp, 7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                                                   
cluster_nodeops_1   /sbin/my_init   Up      0.0.0.0:61620->61620/tcp, 7000/tcp, 7001/tcp, 7199/tcp, 0.0.0.0:8888->8888/tcp, 9042/tcp, 9160/tcp 
cluster_seeddc1_1   /sbin/my_init   Up      7000/tcp, 7001/tcp, 7199/tcp, 0.0.0.0:9042->9042/tcp, 0.0.0.0:9160->9160/tcp                       
cluster_seeddc2_1   /sbin/my_init   Up      7000/tcp, 7001/tcp, 7199/tcp, 0.0.0.0:9043->9042/tcp, 0.0.0.0:9161->9160/tcp      
```

seed nodes is specified in cassandra.yaml, note servce name defined in docker-compose(e..g seeddc1) is resolved thanks to Docker embedded dns server.

```
- seeds:  "seeddc1,seeddc2"
```

### Run nodetool to check cluster status

```
docker exec -it cluster_seeddc1_1 /opt/cassandra/bin/nodetool status

Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.20.0.5  67.89 KB   256     41.4%             4c314bea-eba7-42da-b6ed-31b31e921598  RACK1
UN  172.20.0.2  102.12 KB  256     42.6%             7e8fdefe-4418-42e8-80df-f7d28911a4a5  RACK1
Datacenter: DC2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.20.0.6  67.79 KB   256     38.1%             eec2b7d8-944c-4893-ac8f-5a781548762d  RACK1
UN  172.20.0.4  109.94 KB  256     39.1%             1cdba952-d2ff-4fd5-966a-6e52f3ca9ac9  RACK1
Datacenter: 
============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.20.0.3  82.34 KB   256     38.8%             617b8e2c-3157-41f6-a301-52010694a1cc  RACK1
```

### Scale data center nodes now.

```
docker-compose -p cluster scale nodedc1=2 nodedc2=2
```

Check `nodetool` again and you will see another two nodes joined datacenters.

```
docker exec -it cluster_seeddc1_1 /opt/cassandra/bin/nodetool status

Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UJ  172.20.0.7  9.64 KB    256     ?                 6af03e20-d577-4cea-9369-3001060b515d  RACK1
UN  172.20.0.5  67.89 KB   256     41.4%             4c314bea-eba7-42da-b6ed-31b31e921598  RACK1
UN  172.20.0.2  102.12 KB  256     42.6%             7e8fdefe-4418-42e8-80df-f7d28911a4a5  RACK1
Datacenter: DC2
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UJ  172.20.0.8  9.63 KB    256     ?                 15c118d1-4088-43fa-bf80-5afaf8aa7ebf  RACK1
DN  172.20.0.6  67.79 KB   256     38.1%             eec2b7d8-944c-4893-ac8f-5a781548762d  RACK1
UN  172.20.0.4  100.58 KB  256     39.1%             1cdba952-d2ff-4fd5-966a-6e52f3ca9ac9  RACK1
Datacenter: 
============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens  Owns (effective)  Host ID                               Rack
UN  172.20.0.3  82.34 KB   256     38.8%             617b8e2c-3157-41f6-a301-52010694a1cc  RACK1
```

### Stop the cluster

```
docker-compose -p cluster down
```




