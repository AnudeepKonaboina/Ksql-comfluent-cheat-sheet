Installation of KSQL using docker-compose:
==========================================

1.Run curl --silent --output docker-compose.yml \
  https://raw.githubusercontent.com/confluentinc/cp-all-in-one/6.1.0-post/cp-all-in-one/docker-compose.yml

2.Run **docker-compose up** -d for starting confluent platform and verify the services using **docker-compose ps** 

Output:
=======
```
[ak0107@devopsagent1 ~]$ curl --silent --output docker-compose.yml \
>   https://raw.githubusercontent.com/confluentinc/cp-all-in-one/6.1.0-post/cp-all-in-one/
[ak0107@devopsagent1 ~]$ ll -tr
-rw-rw-r--. 1 ak0107 ak0107 6644 Mar  5 14:07 docker-compose.yml
[ak0107@devopsagent1 ~]$ docker-compose up -d
Creating zookeeper ... done
Creating broker    ... done
Creating schema-registry ... done
Creating connect         ... done
Creating rest-proxy      ... done
Creating ksqldb-server   ... done
Creating control-center  ... done
Creating ksqldb-cli      ... done
Creating ksql-datagen    ... done
[ak0107@devopsagent1 ~]$ docker ps
CONTAINER ID   IMAGE                                             COMMAND                  CREATED        STATUS        PORTS                                                                   NAMES
153de4de9556   confluentinc/cp-enterprise-control-center:6.1.0   "/etc/confluent/dock…"   22 hours ago   Up 22 hours   0.0.0.0:9021->9021/tcp                                                  control-center
2f1db151c30c   confluentinc/ksqldb-examples:6.1.0                "bash -c 'echo Waiti…"   22 hours ago   Up 22 hours                                                                           ksql-datagen
3fffd9c2b0ad   confluentinc/cp-ksqldb-cli:6.1.0                  "/bin/sh"                22 hours ago   Up 22 hours                                                                           ksqldb-cli
420e9435664d   confluentinc/cp-ksqldb-server:6.1.0               "/etc/confluent/dock…"   22 hours ago   Up 22 hours   0.0.0.0:8088->8088/tcp                                                  ksqldb-server
f4a6217c223e   confluentinc/cp-kafka-rest:6.1.0                  "/etc/confluent/dock…"   22 hours ago   Up 22 hours   0.0.0.0:8082->8082/tcp                                                  rest-proxy
8a9c71e18789   cnfldemos/cp-server-connect-datagen:0.4.0-6.1.0   "/etc/confluent/dock…"   22 hours ago   Up 22 hours   0.0.0.0:8083->8083/tcp, 9092/tcp                                        connect
6a52e79650ff   confluentinc/cp-schema-registry:6.1.0             "/etc/confluent/dock…"   22 hours ago   Up 22 hours   0.0.0.0:8081->8081/tcp                                                  schema-registry
146caac7bc45   confluentinc/cp-server:6.1.0                      "/etc/confluent/dock…"   22 hours ago   Up 22 hours   0.0.0.0:9092->9092/tcp, 0.0.0.0:9101->9101/tcp                          broker
cb49844b789a   confluentinc/cp-zookeeper:6.1.0                   "/etc/confluent/dock…"   22 hours ago   Up 22 hours   2888/tcp, 0.0.0.0:2181->2181/tcp, 3888/tcp                              zookeeper
351b4bda525b   16ff4e68d176                                      "./entrypoint.sh nod…"   45 hours ago   Up 45 hours   0.0.0.0:3000->3000/tcp                                                  vigorous_rhodes
d63c1392d129   4239cd2958c6                                      "/usr/bin/docker-qui…"   6 days ago     Up 6 days     0.0.0.0:7180->7180/tcp, 0.0.0.0:8888->8888/tcp, 0.0.0.0:49154->80/tcp   zealous_ramanujan

``` 
3.Navigate to control center UI at http://localhost:9021