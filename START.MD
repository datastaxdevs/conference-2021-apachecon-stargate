
## Start with Docker


**✅ 1a. Start node1 with `docker-compose`**

```bash
export CASSTAG=3.11.8
export SGTAG=v1.0.34
docker-compose up -d backend-1
```

**✅ 1b. When the bootstrap is completed, Start node2 with `docker-compose`**

```bash
docker-compose up -d backend-2
```

**✅ 1c. When the bootstrap is completed, Start node3 with `docker-compose`**
```
docker-compose up -d backend-3
````

**✅ 1d. Check status if your nodes with `nodetool`**

```bash
docker exec -it `docker ps | grep backend-1 | cut -b 1-12` nodetool status
```

**✅ 1e. Get IP of backend-1 to use `cqlsh`**
```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' `docker ps | grep backend-1 | cut -b 1-12`
```

**✅ 1f. Connect as `CQLSH` to backend-1**
```bash
docker exec -it `docker ps | grep backend-1 | cut -b 1-12` cqlsh 172.19.0.2 -u cassandra -p cassandra
```

**✅ 1g. Create expected data model**

The stargate node executes query with `LOCAL_QUORUM`, the table handling the token `data_endpoint_auth` must have a replication factory of 3 in the 

```cql
DROP KEYSPACE IF EXISTS data_endpoint_auth;

CREATE KEYSPACE data_endpoint_auth WITH replication = {'class': 'NetworkTopologyStrategy', 'datacenter1': '3'}  AND durable_writes = true;

CREATE KEYSPACE ks1 WITH replication = {'class': 'NetworkTopologyStrategy', 'datacenter1': '3'}  AND durable_writes = true;

use ks1;

CREATE TYPE IF NOT EXISTS video_format (
  width   int,
  height  int
);

CREATE TABLE IF NOT EXISTS videos (
 videoid   uuid,
 title     text,
 upload    timestamp,
 email     text,
 url       text,
 tags      set <text>,
 frames    list<int>,
 formats   map <text,frozen<video_format>>,
 PRIMARY KEY (videoid)
);

INSERT INTO videos(videoid, email, title, upload, url, tags, frames, formats)
VALUES(uuid(), 'clu@sample.com', 'sample video', 
     toTimeStamp(now()), 'http://google.fr',
     { 'cassandra','accelerate','2020'},
     [ 1, 2, 3, 4], 
     { 'mp4':{width:1,height:1},'ogg':{width:1,height:1}});
```





docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' `docker ps | grep stargateio | cut -b 1-12`

docker-compose up -d stargate


docker exec -it `docker ps | grep backend-1 | cut -b 1-12` cqlsh 172.18.0.5 -u cassandra -p cassandra


curl -L -X POST 'http://localhost:8081/v1/auth' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "username": "cassandra",
    "password": "cassandra"
}'
