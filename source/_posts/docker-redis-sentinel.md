title: Redis Sentinel Docker
date: 2019-03-06 00:23:08
tags:
  - redis 
  - docker
---


I am a big fan of TDD, and I use docker a lot to build the dependencies(APIs, Database, etc.) for the unit test in my local development environment and CI. Everything goes well until one day our lovely DevOps guys asked me to use [Redis sentinel](https://redis.io/topics/sentinel) which provides high availability, it's a good practice, and I like the automatic failover capability. Since we always try to align the test environment to the production one, even for the local development environment, so I plan to build the Redis sentinel with docker.

# Challenge
Redis Sentinel is a typical distributed architecture, the significant difference of using sentinel is that you should ask "redis-sentinel" for "redis-master" first, then issue redis command to the "redis-master" sentinel told you.
Long story short, I need to orchestrate 1 master/ 1 slave/ 1 sentinel with docker properly.

<!-- more -->  

# Understand the docker network
Before starting to build the instances, we need to choose which docker network mode we are going to use.
A quick going through the following links you will get an idea that we have a few options, but since 
I am building a bunch of standalone containers, `bridge` or `host` drivers seem reasonable to me(others seem to overkilling).
- https://docs.docker.com/network/
- https://docs.docker.com/network/network-tutorial-standalone/

# Issue with bridge
Let me chose the default network driver bridge, it's simple and straightforward, i.e.
- My Application -> running on host
- Sentinel -> docker 
- Master -> docker
- Slave -> docker
```
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ 
                                                                              
│   bridge network                                                          │ 
                                                                              
│          ┌─────────────────────────────────────────────────────┐          │ 
           │                                                     │            
│          ▼                                                     ▼          │ 
   ┌───────────────┐           ┌───────────────┐         ┌───────────────┐    
│  │               │           │               │         │               │  │ 
   │               │           │               │         │               │    
│  │    Master     │           │     Slave     │         │   Sentinel    │  │ 
   │  172.19.0.2   │◀─────────▶│  172.19.0.3   │◀───────▶│  172.19.0.4   │    
│  │               │           │               │         │               │  │ 
   │               │           │               │         │               │    
│  └───────────────┘           └───────────────┘         └───────────────┘  │ 
           ▲                                                     △            
│          │                                                     │          │ 
           └──────────X───────────────┐┌─────────────────────────┘            
└ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┼│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┘ 
                                      ││                                      
                                      │▽  master is 172.19.0.2                
 ┌───────────────────────────────────────────────────────────────────────────┐
 │                                Application                                │
 │                                192.169.0.2                                │
 │                                                                           │
 └───────────────────────────────────────────────────────────────────────────┘
```

Oops, you see? When the application asks sentinel process for the master, it returns the *internal* IP of master,
because our application is not in the bridge network, 172.19.x.x is invisible.

# Switch to host network?
If I use host network, all boxes in the above diagram use the host's network directly which means all are under 192.168.x.x and they can talk to each other without any obstacle. Unfortunately, it doesn't look suitable to me just because I develop within OSX system; there is a prohibitive [limitation](https://docs.docker.com/network/host/).
> The host networking driver only works on Linux hosts, and is not supported on Docker Desktop for Mac, Docker Desktop for Windows, or Docker EE for Windows Server.

# Back to bridge
I need to find a way to let those 4 boxes be able to talk to each other, yes, there is a way, i.e. **port forwarding**.
Exposing the master, slave,  and sentinel to the external network by different ports, now they are in one world!

```
┌ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ┐ 
                                                                              
│         bridge network                                                    │ 
                                                                              
│                                                                           │ 
   ┌──────────────────┐      ┌─────────────────┐       ┌─────────────────┐    
│  │                  │      │                 │       │                 │  │ 
   │                  │      │                 │       │                 │    
│  │      Master      │      │      Slave      │       │    Sentinel     │  │ 
   │    172.19.0.2    │      │   172.19.0.3    │       │   172.19.0.4    │    
│  │                  │      │                 │       │                 │  │ 
   │                  │      │                 │       │                 │    
│  └─────────┬────────┘      └────────┬────────┘       └────────┬────────┘  │ 
 ─ ─ ─ ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│─ ─ ─ ─ ─ ─  
             ●                        ●                         │             
    ┌─────────────────┐      ┌─────────────────┐        ┌───────●─────────┐   
    │192.168.0.2:6379 │      │192.168.0.2:6380 │        │192.168.0.2:26379│   
    │                 │◀────▶│                 │◀──────▶│                 │   
    └─────────────────┘      └─────────────────┘        └─────────────────┘   
             ▲        ▲                                 ▲        △            
             │        │                                 │        │            
             │        └─────────────────────────────────┘        │            
             │                                                   │            
             │                                                   │            
             │                         master is 192.168.0.2:6379│            
 ┌───────────┴───────────────────────────────────────────────────▽───────────┐
 │                                Application                                │
 │                                192.169.0.2                                │
 │                                                                           │
 └───────────────────────────────────────────────────────────────────────────┘
```

#  Tweak the redis configuration
The next step is that I need to set the redis configuration carefully, I can't pass the internal IP in the config; everything should be the host's IP.

Here is the docker-compose.yaml,
```yaml
version: '2.2'
services:
  redis_master:
    image: redis:3
    container_name: redis_master
    ports:
      - '6379:6379'
  redis_slave:
    image: redis:3
    container_name: redis_slave
    command: redis-server --port 6380 --slaveof "${EXTERNAL_HOST}" 6379 --slave-announce-ip "${EXTERNAL_HOST}"
    ports:
      - '6380:6380'
  sentinel:
    build: ./sentinel
    container_name: redis_sentinel
    ports:
      - '26379:26379'
    environment:
      - SENTINEL_NAME=mysentinel
      - HOST_IP="${EXTERNAL_HOST}"

```
- master: nothing needs to be taken care, just expose the 6379 port.
- slave: 2 important options, `--salveof` follows with master's IP, and `--slave-announce-ip` is like `whoami` announcement for slave itself, see details [here](https://redis.io/topics/replication#configuring-replication-in-docker-and-nat), both should be set with the host's IP. Here I use a variable, you will see where the variable is from later. 
- sentinel: a bit complecated, but it will be clear as you continue reading.(**Declaration**: how to build the sentinel docker is heavily borrowed from https://github.com/mustafaileri/redis-cluster-with-sentinel)


### Sentinel Dockefile
```
FROM redis:3

EXPOSE 26379
COPY sentinel.conf /etc/redis/sentinel.conf
RUN chown redis:redis /etc/redis/sentinel.conf
ENV SENTINEL_QUORUM 2
ENV SENTINEL_NAME mysentinel
ENV SENTINEL_DOWN_AFTER 30000
ENV SENTINEL_FAILOVER 180000
COPY sentinel-entrypoint.sh /usr/local/bin/
ENTRYPOINT ["sentinel-entrypoint.sh"]
```
Nothing special except the `sentinel.conf` and sh script, no worries, they are elaborated as follows,

### sentinel.conf
```
port 26379

dir /tmp

sentinel monitor $SENTINEL_NAME $HOST_IP 6379 $SENTINEL_QUORUM

sentinel down-after-milliseconds $SENTINEL_NAME $SENTINEL_DOWN_AFTER

sentinel parallel-syncs $SENTINEL_NAME 1

sentinel failover-timeout $SENTINEL_NAME $SENTINEL_FAILOVER
```
A few basic settings, the significant two are `SENTINEL_NAME` and `HOST_IP`, they are actually from the docker-compose.yml, scroll up and check the environment part if you want.

### sentinel-entrypoint
```sh
sed -i "s/\$SENTINEL_QUORUM/$SENTINEL_QUORUM/g" /etc/redis/sentinel.conf
sed -i "s/\$SENTINEL_DOWN_AFTER/$SENTINEL_DOWN_AFTER/g" /etc/redis/sentinel.conf
sed -i "s/\$SENTINEL_FAILOVER/$SENTINEL_FAILOVER/g" /etc/redis/sentinel.conf
sed -i "s/\$SENTINEL_NAME/$SENTINEL_NAME/g" /etc/redis/sentinel.conf
sed -i "s/\$HOST_IP/$HOST_IP/g" /etc/redis/sentinel.conf

exec docker-entrypoint.sh redis-server /etc/redis/sentinel.conf --sentinel
```
Replacing the variables in the sentinel.conf and start the sentinel, that's it.
The 4 `SENTINEL_XXX` variables are from the Dockfile, the `HOST_IP` is set as `EXTERNAL_HOST` which is still from the docker-compose.yaml, the last question is, where does `EXTERNAL_HOST` come from?


### Last piece 
Remember the diagram with 4 box in one IP, the `EXTERNAL_HOST` is the IP of host, apparently `localhost` won't work here. Here I craft a script to start the dockers instead of `docker-compose up`,
the main difference is that I feed the IP to the environment variable.
```sh
#!/bin/bash
set -ev

IP=`ifconfig | grep -Eo 'inet (addr:)?([0-9]*\.){3}[0-9]*' | grep -Eo '([0-9]*\.){3}[0-9]*' | grep -v '127.0.0.1'`
pushd `dirname $0` # make sure at the same folder as docker-compose.yml

echo "EXTERNAL_HOST=$IP" > .env

# Start the services and wait for it.
docker-compose up -d --build
docker-compose ps
```

The line with `ifconfig` seems to be scaring but it's only to get the IP, the key point here is that I write the IP to the `.env` where docker-compose will reload environment variables.

# Feel free to clone and try it!
https://github.com/xavierchow/docker-redis-sentinel



