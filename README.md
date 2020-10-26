# Redis-Sentinel-Docker

This is an example project which implements redis master slave architecture in Python and deploys it using Docker-Compose. Redis Sentinel has been used to increase availability of the infrastructure

## Dependencies

* [Python 3.7](https://www.python.org/downloads/release/python-370/)
* [Redis](https://redis.io/)
* [Redis Sentinel](https://redis.io/topics/sentinel)
* [Docker](https://www.docker.com/)
* [Docker Compose](https://docs.docker.com/compose/)
* [Py-Redis](https://pypi.org/project/redis/)

## Architecture

It runs 1 redis `master` node with 2 `slave` nodes. There are total 2 `sentinel` nodes running in total. There is one node which hosts the python application. This application initially waits 60 seconds for redis and sentinel nodes to be functional. It then tries to write and read from the redis.

Then the applicaton sleeps for 2 minutes, giving the opportunity for the user to manually stop the redis master node. Sentinel should promote one of the slaves to the master when this happens. After 2 minutes, when the application tries to write and read again from the redis, it would be from the new master.

## Running the program

* To run the application locally : `docker-compose up`
* To run in detached mode : `docker-compose up -d`
* To see the logs of a specific, container, first find the container name by running `docker ps -a`, then run : `docker container logs <container_name>`
* To clean up : `docker-compose down -v --rmi all --remove-orphans`
* We can run separate container by : `docker-compose up <container_name>` where `container_name` is one of the service names from the docker-compose.yml.
* To view the sentinel configuration, use : `docker exec -it <sentinel_container_name> bash`. Run `redis-cli -p 26379` to access the sentinel cli. Run `info sentinel` to view the sentinal configuration. It would be of the following form:
```
$ docker exec -it redis-cluster_sentinel-1_1 bash
root@4f16419bb77b:/data# redis-cli -p 26379
127.0.0.1:26379> info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=172.28.0.4:6379,slaves=2,sentinels=3
127.0.0.1:26379>
```

* To view the redis node master-slave configurations, use : `docker exec -it <redis_container_name> bash`. Run `redis-cli` to access the redis cli. Run `info replication` to view the configuration. It will be of the following form:

For master :
```
$ docker exec -it redis-cluster_redis-master_1 bash
root@67347b7522ab:/data# redis-cli
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:172.28.0.4
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:9321
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:9381
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:2
repl_backlog_histlen:9380
127.0.0.1:6379>
```
For slave :
```
$ docker exec -it redis-cluster_redis-slave-2_1 bash
root@6b98845fa6b1:/data# redis-cli
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:redis-master
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:2873
slave_priority:100
slave_read_only:1
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
127.0.0.1:6379>
```
* Once the application has accessed the redis once, kill the master using the command `docker container kill <redis-master-container-name>`
* Monitor the logs for application using : `docker container logs <application-container-name> -ft`. They will be as follows :

```
$ docker container logs redis-cluster_redis_cluster_app_1 -ft
2020-10-26T15:35:07.221841300Z Waiting for redis to run..
2020-10-26T15:36:07.209120900Z *****************
2020-10-26T15:36:07.217727000Z {'success': True}
2020-10-26T15:36:07.218993800Z {'success': True, 'value': 'world'}
2020-10-26T15:36:07.219075000Z ******** SLEEPING *********
2020-10-26T15:38:07.182580000Z ******** AGAIN *********
2020-10-26T15:38:07.184196100Z {'success': True}
2020-10-26T15:38:07.185728100Z {'success': True, 'value': 'world2'}
```
* We can also manually check if the keys `hello` and `hello2` exist in the redis nodes by manually running `get <key>` in the CLIs of the containers.
