## yugabytedb-5-nodes-cluster-docker-sample

We will create (locally with Docker) a [YugabyteDB](https://www.yugabyte.com/) cluster with a replication factor of 5 that allows a fault tolerance of 2.
This means the cluster will remain available for both reads and writes even if two nodes fail.

[YugabyteDB Docs](https://docs.yugabyte.com/latest/)

There is another repository with a 3 nodes cluster solution [here](https://github.com/wagnerjfr/yugabytedb-docker-sample).

### 1. Install YugabyteDB
Pull the YugabyteDB Docker image.
```
docker pull yugabytedb/yugabyte
```
Pull Download the YugabyteDB workload generator.
```
docker pull yugabytedb/yb-sample-apps
```

### 2. Docker network
Let's create a Docker network named "universe"
```
docker network create universe
```

### 3. Create the YB-Masters
We are going to use **"yb-master"** binary and its flags to configure the YB-Master server in 5 containers [[ref.](https://docs.yugabyte.com/latest/reference/configuration/yb-master/)].
```
for N in 1 2 3 4 5
do docker run -d --name yb-master-n$N --net=universe --hostname=yb-master-n$N -p700$N:7000 \
  yugabytedb/yugabyte:latest ./bin/yb-master \
  --master_addresses yb-master-n1:7100,yb-master-n2:7100,yb-master-n3:7100,yb-master-n4:7100,yb-master-n5:7100 \
  --rpc_bind_addresses yb-master-n$N:7100 \
  --fs_data_dirs "/tmp/ybd" \
  --replication_factor=5
done
```

### 4. Create the YB-TServers
We are going to use **"yb-tserver"** binary and its flags to configure the YB-TServer server in 5 containers [[ref.](https://docs.yugabyte.com/latest/reference/configuration/yb-tserver/)].
```
for N in 1 2 3 4 5
do docker run -d --name yb-tserver-n$N --net=universe --hostname=yb-tserver-n$N -p900$N:9000 -p543$N:5433 -p904$N:9042 \
  yugabytedb/yugabyte:latest ./bin/yb-tserver \
  --tserver_master_addrs yb-master-n1:7100,yb-master-n2:7100,yb-master-n3:7100,yb-master-n4:7100,yb-master-n5:7100 \
  --rpc_bind_addresses yb-tserver-n$N:9100 \
  --start_pgsql_proxy \
  --fs_data_dirs "/tmp/tserver"
done
```

### 5. Check cluster status
#### 5.1 Docker
Run:
```
docker ps -a
```
Result:
```
docker ps -a
CONTAINER ID   IMAGE                        COMMAND                  CREATED          STATUS          PORTS                                                                                                                                                                                                                NAMES
fc2b9f7e2003   yugabytedb/yugabyte:latest   "./bin/yb-tserver --…"   8 seconds ago    Up 3 seconds    6379/tcp, 7000/tcp, 7100/tcp, 7200/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:5435->5433/tcp, :::5435->5433/tcp, 0.0.0.0:9005->9000/tcp, :::9005->9000/tcp, 0.0.0.0:9045->9042/tcp, :::9045->9042/tcp   yb-tserver-n5
3574f4893717   yugabytedb/yugabyte:latest   "./bin/yb-tserver --…"   14 seconds ago   Up 9 seconds    6379/tcp, 7000/tcp, 7100/tcp, 7200/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:5434->5433/tcp, :::5434->5433/tcp, 0.0.0.0:9004->9000/tcp, :::9004->9000/tcp, 0.0.0.0:9044->9042/tcp, :::9044->9042/tcp   yb-tserver-n4
d68cc449cc9f   yugabytedb/yugabyte:latest   "./bin/yb-tserver --…"   19 seconds ago   Up 14 seconds   6379/tcp, 7000/tcp, 7100/tcp, 7200/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 0.0.0.0:5433->5433/tcp, :::5433->5433/tcp, 12000/tcp, 0.0.0.0:9003->9000/tcp, :::9003->9000/tcp, 0.0.0.0:9043->9042/tcp, :::9043->9042/tcp   yb-tserver-n3
9590e4dd36d3   yugabytedb/yugabyte:latest   "./bin/yb-tserver --…"   25 seconds ago   Up 19 seconds   6379/tcp, 7000/tcp, 7100/tcp, 7200/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:9042->9042/tcp, :::9042->9042/tcp, 0.0.0.0:5432->5433/tcp, :::5432->5433/tcp, 0.0.0.0:9002->9000/tcp, :::9002->9000/tcp   yb-tserver-n2
84e7577472cd   yugabytedb/yugabyte:latest   "./bin/yb-tserver --…"   32 seconds ago   Up 25 seconds   6379/tcp, 7000/tcp, 7100/tcp, 7200/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:5431->5433/tcp, :::5431->5433/tcp, 0.0.0.0:9001->9000/tcp, :::9001->9000/tcp, 0.0.0.0:9041->9042/tcp, :::9041->9042/tcp   yb-tserver-n1
e42c4307ce45   yugabytedb/yugabyte:latest   "./bin/yb-master --m…"   39 seconds ago   Up 35 seconds   5433/tcp, 6379/tcp, 7100/tcp, 7200/tcp, 9000/tcp, 9042/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:7005->7000/tcp, :::7005->7000/tcp                                                                     yb-master-n5
e748797d9676   yugabytedb/yugabyte:latest   "./bin/yb-master --m…"   42 seconds ago   Up 39 seconds   5433/tcp, 6379/tcp, 7100/tcp, 7200/tcp, 9000/tcp, 9042/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:7004->7000/tcp, :::7004->7000/tcp                                                                     yb-master-n4
74c26171b575   yugabytedb/yugabyte:latest   "./bin/yb-master --m…"   46 seconds ago   Up 42 seconds   5433/tcp, 6379/tcp, 7100/tcp, 7200/tcp, 9000/tcp, 9042/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:7003->7000/tcp, :::7003->7000/tcp                                                                     yb-master-n3
619cea2c2d8a   yugabytedb/yugabyte:latest   "./bin/yb-master --m…"   49 seconds ago   Up 46 seconds   5433/tcp, 6379/tcp, 7100/tcp, 7200/tcp, 9000/tcp, 9042/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:7002->7000/tcp, :::7002->7000/tcp                                                                     yb-master-n2
857828be2120   yugabytedb/yugabyte:latest   "./bin/yb-master --m…"   53 seconds ago   Up 49 seconds   5433/tcp, 6379/tcp, 7100/tcp, 7200/tcp, 9000/tcp, 9042/tcp, 9100/tcp, 10100/tcp, 11000/tcp, 12000/tcp, 0.0.0.0:7001->7000/tcp, :::7001->7000/tcp                                                                     yb-master-n1
```
#### 5.2 Admin UI
We don't know which `yb-master` was selected to be the leader so the Admin UI is available at:
[http://localhost:7001](http://localhost:7001), [http://localhost:7002](http://localhost:7002), [http://localhost:7003](http://localhost:7003), [http://localhost:7004](http://localhost:7004), [http://localhost:7005](http://localhost:7005)

and the `yb-tserver` the Admin UI is available at:
[http://localhost:9001](http://localhost:9001), [http://localhost:9001](http://localhost:9002), [http://localhost:9003](http://localhost:9003), [http://localhost:9004](http://localhost:9004), [http://localhost:9005](http://localhost:9005).

##### YB-Master status
In this execution the leader is `yb-master-n3`
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/master1.png)

##### YB-TServer status
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/tserver1.png)

### 6. Emulate workloads against YugabyteDB
Run the command below and two new containers `yb-client-n4` and `yb-client-n5` will be create and start to write and read some data accesing the `yb-tserver-n4` and `yb-tserver-n5`, through the port `5433`:
```
for N in 4 5
do docker run -d --rm --name yb-client-n$N --net=universe yugabytedb/yb-sample-apps:latest \
  --workload SqlInserts --nodes yb-tserver-n$N:5433 --num_threads_write 1 --num_threads_read 2
done
```

##### YB-TServer status
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/tserver2.png)

##### Tablet splitting
The resharding of `postgres` table data in the cluster. List of all tablets managed by this specific instance `tserver`. 
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/tablets.png)


### 7. Fault tolarence with node kill
Let's kill the leader and another (`yb-master` and `yb-tserver` containers) to observe new leader election and continuous write/ready availability.

From the screenshot above the leader is `yb-master-n3`. Let's also kill `yb-master-n2`.
```
for N in 2 3
do docker rm -f yb-master-n$N yb-tserver-n$N
done
```
After some seconds, a new leader was elected and 2 nodes are down.
##### YB-Master status
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/master2.png)

##### YB-TServer status
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/tserver3.png)

### 8. Recreating the node and rejoinging
Creating the containers again:
```
for N in 2 3
do docker run -d --name yb-master-n$N --net=universe --hostname=yb-master-n$N -p700$N:7000 \
  yugabytedb/yugabyte:latest ./bin/yb-master \
  --master_addresses yb-master-n1:7100,yb-master-n2:7100,yb-master-n3:7100,yb-master-n4:7100,yb-master-n5:7100 \
  --rpc_bind_addresses yb-master-n$N:7100 \
  --fs_data_dirs "/tmp/ybd" \
  --replication_factor=5
done

for N in 2 3
do docker run -d --name yb-tserver-n$N --net=universe --hostname=yb-tserver-n$N -p900$N:9000 -p543$N:5433 -p904$N:9042 \
  yugabytedb/yugabyte:latest ./bin/yb-tserver \
  --tserver_master_addrs yb-master-n1:7100,yb-master-n2:7100,yb-master-n3:7100,yb-master-n4:7100,yb-master-n5:7100 \
  --rpc_bind_addresses yb-tserver-n$N:9100 \
  --start_pgsql_proxy \
  --fs_data_dirs "/tmp/tserver"
done
```
After some seconds/minutes, we have the cluster completely restored.
##### YB-Master status
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/master3.png)

##### YB-TServer status
![alt text](https://github.com/wagnerjfr/yugabytedb-5-nodes-cluster-docker-sample/blob/main/figures/tserver4.png)

### 9. Clean up
To clean up everything:
```
for N in 1 2 3 4 5
do docker rm -f yb-master-n$N yb-tserver-n$N
done

for N in 4 5
do docker rm -f yb-client-n$N
done

docker network rm universe

docker rmi yugabytedb/yugabyte yugabytedb/yb-sample-apps
```
