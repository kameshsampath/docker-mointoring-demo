# Docker Monitoring Demo
A basic docker monitoring demo application using Axibase and Google cAdvisor.

### Vagrant boxes
The Demo application will have 3 hosts, refer to [Vagrantfile](../Vagrantfile) for more details,
 * rhel7-cdkv2 - [RHEL 7 Container Development Kit] (http://developers.redhat.com/containers/adoption/) - the machine which will host Axibase, Grafana etc., and its a host where all docker image building, registry setup etc., its optional to have RHEL7 CDK, you can use any Linux host you want which can support docker 
 * node[1,2] - the docker container hosts, which is based on [Centos Atomic Host](https://wiki.centos.org/SpecialInterestGroup/Atomic), which will host cAdvisor and some demo docker containers
 
#### Environment Variables
`_AXIBASE_DB_HOST=192.168.56.100`

# Setting up containers

### Docker Registry (Vagrant Box: rhel7-cdkv2)
A docker registry will come handy if there is need to create/destory child nodes multiple times to get your hands dirty and ablity to quickly pull the images 

sudo docker create -p 5000:5000 \
 -v /var/lib/cdkv2-docker-registry:/srv/registry \
-e STANDALONE=false \
-e MIRROR_SOURCE=https://registry-1.docker.io \
-e MIRROR_SOURCE_INDEX=https://index.docker.io \
-e STORAGE_PATH=/srv/registry \
--name=cdkv2-docker-registry registry:2

### Create ATSD container (Vagrant Box: rhel7-cdkv2)

The ATSD container will host the Axibase Time series Database, where the [cadvisor](https://github.com/axibase/cadvisor-atsd) from the docker container hosts will push the metrics

```
sudo docker run \
  -d \
  -p 5022:22 \
  --privileged=true  \
  -p 8088:8088 \
  -p 8081:8081 \
  -p 8443:8443 \
  -p 8082:8082/udp \
  -e AXIBASE_USER_PASSWORD=redhat \
  -e ATSD_USER_NAME=redhat \
  -e ATSD_USER_PASSWORD=redhat \
  -h atsd \
  --name=atsd \
  axibase/atsd
```

### Create cAdvisor (Vagrant Box: node1,node2)
[cadvisor](https://github.com/axibase/cadvisor-atsd) is responsible for pushing docker container host metrics to ATSD, use the following commands to create the [cadvisor](https://github.com/axibase/cadvisor-atsd) containers 

``` 
sudo docker run \
    --volume=/:/rootfs:ro \
    --volume=/var/run:/var/run:rw \
    --volume=/sys:/sys:ro \
    --volume=/var/lib/docker/:/var/lib/docker:ro \
    --publish=8080:8080 \
    --detach=true \
    --privileged=true \
    --name=cadvisor \
    axibase/cadvisor:latest \
    --storage_driver=atsd \
    --storage_driver_user=redhat \
    --storage_driver_password=redhat \
    --atsd_storage_url=http://$_AXIBASE_DB_HOST:8088 \
    --atsd_storage_write_host=$_AXIBASE_DB_HOST:8082 \
    --atsd_storage_write_protocol=udp \
    --atsd_storage_docker_host="`hostname`" \
    --atsd_storage_property_interval=15s \
    --housekeeping_interval=15s \
    --storage_driver_buffer_duration=15s
```

### Create Grafana (Optional)(Vagrant Box: rhel7-cdkv2)
Axibase does provide some native visualizations for cadvisor, if you want to pull the same on to Grafana you can have it installed using the following docker commands:

```
sudo docker create -v /data --name influxdb-data busybox /bin/true
```

```
sudo docker create -p 3000:3000 \
  -e INFLUXDB_HOST=localhost \
  -e INFLUXDB_PORT=8086 \
  -e INFLUXDB_NAME=cadvisor \
  -e INFLUXDB_USER=root \
  -e INFLUXDB_PASS=root \
  --privileged=true \
  --link atsd:atsd \
  --name grafana grafana/grafana
```
# License
[LICENSE][../LICENSE.md]

# References

  * [Axibase] (https://axibase.com/products/axibase-time-series-database/writing-data/docker-cadvisor/)
  * [Axibase cadvisor] (https://github.com/axibase/cadvisor-atsd)
  * [Google cAdvisor] (https://github.com/google/cadvisor)
