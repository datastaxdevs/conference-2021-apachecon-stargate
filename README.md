## 🎓🔥 An OSS Api Layer for your Cassandra  🔥🎓

[![License Apache2](https://img.shields.io/hexpm/l/plug.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Discord](https://img.shields.io/discord/685554030159593522)](https://discord.com/widget?id=685554030159593522&theme=dark)

![image](pics/splash.png?raw=true)

## Table of content

1. [Prerequisites](#1-prerequisite-install-docker-and-docker-compose)
2. [Start the Demo](#2-start-the-demo)
3. [Use CQL API](#3-use-cql-api)
4. [Use REST API](#4-use-rest-api)
5. [Use Document API](#5-use-document-api-swagger)
6. [Use GraphQL API](#6-use-graphql-api-portal)
7. [Create an Astra Instance](#7-create-your-astra-instance)

## 1. Prerequisite, Install docker and docker-compose ##

To run the demo you need `docker` and `docker-compose`. Skip this steps if you have them available already

### 1.1 Install Docker

Docker is an open-source project that automates the deployment of software applications inside containers by providing an additional layer of abstraction and automation of OS-level virtualization on Linux.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : To install on **windows** please use the following installer [Docker Desktop for Windows Installer](https://download.docker.com/win/stable/Docker%20Desktop%20Installer.exe)

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : To install on **MAC OS**  use [Docker Desktop for MAC Installer](https://download.docker.com/mac/stable/Docker.dmg) or [homebrew](https://docs.brew.sh/Installation) with following commands:
```bash
# Fetch latest version of homebrew and formula.
brew update              
# Tap the Caskroom/Cask repository from Github using HTTPS.
brew tap caskroom/cask                
# Searches all known Casks for a partial or exact match.
brew search docker                    
# Displays information about the given Cask
brew cask info docker
# Install the given cask.
brew cask install docker              
# Remove any older versions from the cellar.
brew cleanup
# Validate installation
docker -v
```

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) : To install on linux (centOS) you can use the following commands
```bash
# Remove if already install
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
# Utils
sudo yum install -y yum-utils

# Add docker-ce repo
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
# Install
sudo dnf -y  install docker-ce --nobest
# Enable service
sudo systemctl enable --now docker
# Get Status
systemctl status  docker

# Logout....Lougin
exit
# Create user
sudo usermod -aG docker $USER
newgrp docker

# Validation
docker images
docker run hello-world
docker -v
```
[🏠 Back to Table of Contents](#Table-Of-content)

### 1.2 Install Docker Compose

Docker Compose is a tool for defining and running multi-container Docker applications. It uses YAML files to configure the application's services and performs the creation and start-up process of all the containers with a single command. The `docker-compose` CLI utility allows users to run commands on multiple containers at once, for example, building images, scaling containers, running containers that were stopped, and more. Please refer to [Reference Documentation](https://docs.docker.com/compose/install/) if you have for more detailed instructions.

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : Already **included** in the previous package *Docker for Windows*

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : Already **included** in the previous package *Docker for Mac*

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) : To install on linux (centOS) you can use the following commands

```bash
# Download deliverable and move to target location
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Allow execution
sudo chmod +x /usr/local/bin/docker-compose
```

### 1.3 Install curl

[Reference Article](https://help.ubidots.com/en/articles/2165289-learn-how-to-install-run-curl-on-windows-macosx-linux)

![Windows](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/windows32.png?raw=true) : Already **included** as stated [here](https://www.thewindowsclub.com/how-to-install-curl-on-windows-10)

![osx](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/mac32.png?raw=true) : Use brew

```
brew install curl
```

![linux](https://github.com/DataStax-Academy/kubernetes-workshop-online/blob/master/4-materials/images/linux32.png?raw=true) :Use your package installer like `yum` or apt-get

```
sudo apt-get install curl
```

[🏠 Back to Table of Contents](#table-of-content)


## 2. Start the demo

#### ✅ 2a. Clone the repository 

Download the repository as a zip [here](https://github.com/datastaxdevs/conference-2021-apachecon-stargate/archive/refs/heads/main.zip) or clone with the following git command

```bash
git clone https://github.com/datastaxdevs/conference-2021-apachecon-stargate.git
``` 

#### ✅ 2b. Start `backend-1`

We provide a `docker-compose.yaml` file ready to go with a `Cassandra 3.11.8` backend and stargate in version `1.0.34`

```bash
export CASSTAG=3.11.8
export SGTAG=v1.0.34
docker-compose up -d backend-1
```

#### ✅ 2c. When the bootstrap is completed, Start node2 with `docker-compose`

```bash
docker-compose up -d backend-2
```

#### ✅ 2d. When the bootstrap is completed, Start node3 with `docker-compose`

```bash
docker-compose up -d backend-3
````

#### ✅ 2e. Check status of your nodes with `nodetool`

```bash
docker exec -it `docker ps | grep backend-1 | cut -b 1-12` nodetool status
```

#### ✅ 2f. Extract IP of `backend-1` as variable `backend1ip`

- Extract variable
```bash
export backend1ip=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' `docker ps | grep backend-1 | cut -b 1-12`)
```

- Check that we are OK
```bash
echo $backend1ip
```

#### ✅ 2g. Connect as `CQLSH` using the new created var

```bash
docker exec -it `docker ps | grep backend-1 | cut -b 1-12` cqlsh $backend1ip -u cassandra -p cassandra
```

#### ✅ 2h. Create Keyspace `data_endpoint_auth`

The stargate node executes query with `LOCAL_QUORUM`, the table handling the token `data_endpoint_auth` must have a replication factory of 3.
The datacenter used everywhere is `datacenter1`
Also here we create an sample keyspace `ks1` for tests.

- Create keyspace

```sql
DROP KEYSPACE IF EXISTS data_endpoint_auth;
CREATE KEYSPACE data_endpoint_auth WITH replication = {'class': 'NetworkTopologyStrategy', 'datacenter1': '3'}  AND durable_writes = true;
```

- Exit CQLSH
```
exit
```

#### ✅ 2i. Start Stargate Node

- Start the node
```
docker-compose up -d stargate
```

- Wait for the node to be up
```
echo "Waiting for stargate to start up..."
while [[ "$(curl -s -o /dev/null -w ''%{http_code}'' http://localhost:8082/health)" != "200" ]]; do
    printf '.'
    sleep 5
done
```

#### ✅ 2j. Wait for the node to bootstrap and get IP

- Extract variable
```
export stargateip=$(docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' `docker ps | grep stargateio | cut -b 1-12`)
```

- check we are OK
```
echo $stargateip
```

#### ✅ 2k. Check we are all GOOD

```
echo "Check Status...."
echo "Backend-1 IP: $backend1ip"
echo "Stargate IP: $stargateip"
docker ps
docker exec -it `docker ps | grep backend-1 | cut -b 1-12` nodetool status
```

**👁️ Expected output**

```bash
Check Status....
Backend-1 IP: 172.19.0.2
Stargate IP: 172.19.0.5

PROMPT> docker ps
CONTAINER ID   IMAGE                              COMMAND                  CREATED       STATUS       PORTS                                                                                                                                                 NAMES
2200f5c86bfd   stargateio/stargate-3_11:v1.0.34   "./starctl"              3 hours ago   Up 3 hours   0.0.0.0:8080-8082->8080-8082/tcp, :::8080-8082->8080-8082/tcp, 0.0.0.0:8084->8084/tcp, :::8084->8084/tcp, 0.0.0.0:9045->9042/tcp, :::9045->9042/tcp   cassandra-3_stargate_1
16cb48746e72   cassandra:3.11.8                   "docker-entrypoint.s…"   3 hours ago   Up 3 hours   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                                                                                                           cassandra-3_backend-3_1
31bd64f67237   cassandra:3.11.8                   "docker-entrypoint.s…"   3 hours ago   Up 3 hours   7000-7001/tcp, 7199/tcp, 9042/tcp, 9160/tcp                                                                                                           cassandra-3_backend-2_1
835408a3d2d0   cassandra:3.11.8                   "docker-entrypoint.s…"   3 hours ago   Up 3 hours   7000-7001/tcp, 7199/tcp, 9160/tcp, 0.0.0.0:9044->9042/tcp, :::9044->9042/tcp                                                                          cassandra-3_backend-1_1

PROMPT>  docker exec -it `docker ps | grep backend-1 | cut -b 1-12` nodetool status

Datacenter: datacenter1
=======================
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  172.19.0.3  355.37 KiB  256          100.0%            75b42435-3197-42ed-bdd8-fdb00294865b  rack1
UN  172.19.0.2  331.99 KiB  256          100.0%            45a33f3a-9115-4878-a1c2-54d00b4c3ff0  rack1
UN  172.19.0.4  320.44 KiB  256          100.0%            9cb00b9d-3bc6-44bb-95a8-e0152d9db9f8  rack1

```

- You should be able to access the GRAPHQL Playground on [http://localhost:8080/playground](http://localhost:8080/playground)

**👁️ Expected output**
![image](pics/playground-home.png?raw=true)

- You should be able to access the Swagger UI on [http://localhost:8082/swagger-ui/#/](http://localhost:8082/swagger-ui/#/)

**👁️ Expected output**
![image](pics/swagger-home.png?raw=true)

- You should be able to access the Metrics UI [http://localhost:8084/metrics](http://localhost:8084/metrics)

**👁️ Expected output**
![image](pics/metrics.png?raw=true)

[🏠 Back to Table of Contents](#table-of-content)

## 3. Use CQL API

#### ✅ 3a. Start CQLSH

- Use this IP to connect with a cqlsh. *Note that the stargate image itself does not provide it we use the cqlsh from backend-1 as a sample client.*

```bash
docker exec -it `docker ps | grep backend-1 | cut -b 1-12` cqlsh $stargateip -u cassandra -p cassandra
```

#### ✅ 3b. Create your schema

```sql
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

describe ks1
```

#### ✅ 3c. Populate some data

- Insert value using plain CQL

```sql
INSERT INTO videos(videoid, email, title, upload, url, tags, frames, formats)
VALUES(uuid(), 'clu@sample.com', 'sample video', 
     toTimeStamp(now()), 'http://google.fr',
     { 'cassandra','accelerate','2020'},
     [ 1, 2, 3, 4], 
     { 'mp4':{width:1,height:1},'ogg':{width:1,height:1}});
```

- Insert Value using JSON

```sql
INSERT INTO videos JSON '{
   "videoid":"e466f561-4ea4-4eb7-8dcc-126e0fbfd573",
     "email":"clunven@sample.com",
     "title":"A Second videos",
     "upload":"2020-02-26 15:09:22 +00:00",
     "url": "http://google.fr",
     "frames": [1,2,3,4],
     "tags":   [ "cassandra","accelerate", "2020"],
     "formats": { 
        "mp4": {"width":1,"height":1},
        "ogg": {"width":1,"height":1}
     }
}';
```

- Read values

```sql
select * from videos;
```

- Read by id
```sql
select * from videos where videoid=e466f561-4ea4-4eb7-8dcc-126e0fbfd573;
```

- Quit `cqlsh`

```sql
exit
```

[🏠 Back to Table of Contents](#table-of-content)

## 4. Use REST API

This walkthrough has been realized using the [REST API Quick Start](https://stargate.io/docs/stargate/0.1/quickstart/quick_start-rest.html). Here we will the the [DATA](http://localhost:8082/swagger-ui/#/data) or SwaggerUI

![image](pics/swagger-general.png?raw=true)

#### ✅ 4a. Generate an auth token

```bash
curl -L -X POST 'http://localhost:8081/v1/auth' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "username": "cassandra",
    "password": "cassandra"
}'
```

Copy the token value (here `74be42ef-3431-4193-b1c1-cd8bd9f48132`) in your clipboard.

**👁️ Expected output**
```
{"authToken":"74be42ef-3431-4193-b1c1-cd8bd9f48132"}
```

#### ✅ 4b. List keyspaces

- [`GET: /v2/schemas/keyspaces`](http://localhost:8082/swagger-ui/#/schemas/getAllKeyspaces)
 
![image](pics/swagger-list-keyspace.png?raw=true)

- Click `Try it out`
- Provide your token in the field `X-Cassandra-Token`
- Click on `Execute`


#### ✅ 4c. List Tables

- [GET /v2/schemas/keyspaces/{keyspaceName}/tables](http://localhost:8082/swagger-ui/#/schemas/getAllTables)

![image](pics/swagger-list-tables.png?raw=true)

- Click `Try it out`
- Provide your token in the field `X-Cassandra-Token`
- keyspace: `ks1`
- Click on `Execute`


#### ✅ 4d. List Types

- [GET /v2/schemas/keyspaces/{keyspaceName}/types](http://localhost:8082/swagger-ui/#/schemas/findAll)

![image](pics/swagger-list-types.png?raw=true)

- Click `Try it out`
- X-Cassandra-Token: `<your_token>`
- keyspace: `ks1`
- Click on `Execute`

#### ✅ 4e Create a Table

- [POST /v2/schemas/keyspaces/{keyspaceName}/tables](http://localhost:8082/swagger-ui/#/schemas/createTable)

![image](pics/swagger-create-table.png?raw=true)

- Click `Try it out`
- X-Cassandra-Token: `<your_token>`
- keyspace: `ks1`
- Data
```json
{
  "name": "users",
  "columnDefinitions":
    [
        {
        "name": "firstname",
        "typeDefinition": "text"
      },
        {
        "name": "lastname",
        "typeDefinition": "text"
      },
      {
        "name": "email",
        "typeDefinition": "text"
      },
        {
        "name": "color",
        "typeDefinition": "text"
      }
    ],
  "primaryKey":
    {
      "partitionKey": ["firstname"],
      "clusteringKey": ["lastname"]
    },
  "tableOptions":
    {
      "defaultTimeToLive": 0,
      "clusteringExpression":
        [{ "column": "lastname", "order": "ASC" }]
    }
}
```

**👁️ Expected output**

```json
{
  "name": "users"
}
```

#### ✅ 4f. Insert Rows

*Notice than for the DML you move to `DATA`. Make sure you are using url with `V2`, `V1` would also work but this is NOT the same payload.* 

- [POST /v2/keyspaces/{keyspaceName}/{tableName}](http://localhost:8082/swagger-ui/#/data/createRow)

![image](pics/swagger-addrows.png?raw=true)

- X-Cassandra-Token: `<your_token>`
- keyspaceName: `ks1`
- tableName: `users`
- Data
```json
{   
    "firstname": "Cedrick",
    "lastname": "Lunven",
    "email": "c.lunven@gmail.com",
    "color": "blue"
}
```

You can note that the output code is `201` and return your primary key `{ "firstname": "Cedrick","lastname": "Lunven" }

- You can add a second record changing only the payload
```json
{
    "firstname": "David",
    "lastname": "Gilardi",
    "email": "d.gilardi@gmail.com",
    "color": "blue"
}
```

- Add a third
```json
{
    "firstname": "Kirsten",
    "lastname": "Hunter",
    "email": "k.hunter@gmail.com",
    "color": "pink"
}
```

#### ✅ 4g. Read multiple rows

- [GET /v2/keyspaces/{keyspaceName}/{tableName}/rows](http://localhost:8082/swagger-ui/#/data/getAllRows_1)
![image](pics/swagger-listrows.png?raw=true)

- X-Cassandra-Token: `<your_token>`
- keyspaceName: `ks1`
- tableName: `users`
- Click Execute

- Notice how now you can only limited return fields

- fields: `firstname, lastname`

#### ✅ 4h. Read a single partition

- [GET /v2/keyspaces/{keyspaceName}/{tableName}/{primaryKey}](http://localhost:8082/swagger-ui/#/data/getRows_1)

![image](pics/swagger-readrows.png?raw=true)

- X-Cassandra-Token: `<your_token>`
- keyspaceName: `ks1`
- tableName: `users`
- primaryKey; 'Cedrick`
- Click Execute

```diff
- Important: The Swagger user interface is limited as of now and you cannot test a composite key (here adding Lunven). This is a bug in the UI not the API.
```

#### ✅ 4i. Delete a row

- [DELETE /v2/keyspaces/{keyspaceName}/{tableName}/{primaryKey}](http://localhost:8082/swagger-ui/#/data/deleteRows)

![image](pics/swagger-deleterows.png?raw=true)

- X-Cassandra-Token: `<your_token>`
- keyspaceName: `ks1`
- tableName: `users`
- primaryKey; 'Cedrick`
- Click Execute

#### ✅ 4j. Searches

- [GET /v2/keyspaces/{keyspaceName}/{tableName}](http://localhost:8082/swagger-ui/#/data/getRowWithWhere)

![image](pics/swagger-searchrows.png?raw=true)


- X-Cassandra-Token: `<your_token>`
- keyspaceName: `ks1`
- tableName: `users`
- whereClause; '{"firstname": {"$eq":"David"}}`
- Click Execute

I let you try with `{"lastname": {"$eq":"Gilardi"}}`.. expected right ?


[🏠 Back to Table of Contents](#table-of-content)

## 5. Use Document API (swagger+curl)

This walkthrough has been realized using the [Quick Start](https://stargate.io/docs/stargate/0.1/quickstart/quick_start-document.html)



**✅ 5a. List Namespaces** :

![image](pics/swagger-doc-listnamespaces.png?raw=true)


**✅ 5b. Create a document** :

*Note: operations requiring providing `namespace` and `collections` on the swagger UI seems not functional. We are switching to CURL the API is working, this is a documentation bug that has been notified to the development team.*

![image](pics/swagger-doc-create.png?raw=true)


```json
{
   "videoid":"e466f561-4ea4-4eb7-8dcc-126e0fbfd573",
     "email":"clunven@sample.com",
     "title":"A Second videos",
     "upload":"2020-02-26 15:09:22 +00:00",
     "url": "http://google.fr",
     "frames": [1,2,3,4],
     "tags":   [ "cassandra","accelerate", "2020"],
     "formats": { 
        "mp4": {"width":1,"height":1},
        "ogg": {"width":1,"height":1}
     }
}
```

**✅ 5c. Retrieve documents** :

![image](pics/swagger-doc-search.png?raw=true)



**✅ 5d. Retrieve 1 document** :

![image](pics/swagger-doc-get.png?raw=true)


**✅ 5e. Search for document by properties** :


![image](pics/swagger-doc-search.png?raw=true)

- WhereClause

```json
{"email": {"$eq": "clunven@sample.com"}}
```

[🏠 Back to Table of Contents](#table-of-content)

## 6. Use GraphQL API (portal)

This walkthrough has been realized using the [GraphQL Quick Start](https://stargate.io/docs/stargate/0.1/quickstart/quick_start-graphql.html)

Same as Rest API generate a `auth token` 

**✅ 6a Generate Auth token** :
```bash
curl -L -X POST 'http://localhost:8081/v1/auth' \
  -H 'Content-Type: application/json' \
  --data-raw '{
    "username": "cassandra",
    "password": "cassandra"
}'
```

Save output as an environment variable
```
export AUTH_TOKEN=7c37bda5-7360-4d39-96bc-9765db5773bc
```

**✅ 6b. Open GraphQL Playground** :

- You should be able to access the GRAPH QL PORTAL on [http://localhost:8080/playground](http://localhost:8080/playground)

You can check on the right of the playground that you have access to documentation and schema which is the neat part about graphQL

**👁️ Expected output**
![image](pics/playground-home.png?raw=true)

**✅ 6c. Creating a Table** :

- Use this query
```
mutation {
  books: createTable(
    keyspaceName:"library",
    tableName:"books",
    partitionKeys: [ # The keys required to access your data
      { name: "title", type: {basic: TEXT} }
    ]
    values: [ # The values associated with the keys
      { name: "author", type: {basic: TEXT} }
    ]
  )
  authors: createTable(
    keyspaceName:"library",
    tableName:"authors",
    partitionKeys: [
      { name: "name", type: {basic: TEXT} }
    ]
    clusteringKeys: [ # Secondary key used to access values within the partition
      { name: "title", type: {basic: TEXT}, order: "ASC" }
    ]
  )
}
```

**👁️ Expected output**
![image](pics/graphql-createtables.png?raw=true)

**✅ 6d. Populating Table** :

Any of the created APIs can be used to interact with the GraphQL data, to write or read data.

First, let’s navigate to your new keyspace `library` inside the playground. Change tab to `graphql` and pick url `/graphql/library`.

- Use this query
```
mutation {
  moby: insertBooks(value: {title:"Moby Dick", author:"Herman Melville"}) {
    value {
      title
    }
  }
  catch22: insertBooks(value: {title:"Catch-22", author:"Joseph Heller"}) {
    value {
      title
    }
  }
}
```

- Don't forget to update the header again
```
{
  "x-cassandra-token":"7c37bda5-7360-4d39-96bc-9765db5773bc"
}
```
**👁️ Expected output**
![image](pics/graphql-insertdata.png?raw=true)


**✅ 6e. Read data** :

Stay on the same screen and sinmply update the query with 
```
query oneBook {
    books (value: {title:"Moby Dick"}) {
      values {
        title
        author
      }
    }
}
```

**👁️ Expected output**
![image](pics/graphql-readdata.png?raw=true)


[🏠 Back to Table of Contents](#table-of-content)

## 7. Create your ASTRA Instance

**✅ Create an free-forever Cassandra database with DataStax Astra**: [click here to get started](https://astra.datastax.com/register?utm_source=github&utm_medium=referral&utm_campaign=spring-petclinic-reactive) 🚀


![Astra Registration Screen](pics/db-auth.png?raw=true)


**✅ Use the form to create new database**

On the Astra home page locate the **Add Database** button

![Astra Database Creation Form](pics/db-creation-1.png?raw=true)

Select the **free tier** plan, this is a true free tier, free forever and no payment method asked 🎉 🎉

![Astra Database Creation Form](pics/db-creation-2.png?raw=true)

Select the proper region and click the `configure` button. The number of regions and cloud providers are limited in the free tier but please notice you can run the DB on any cloud with any VPC Peering.

![Astra Database Creation Form](pics/db-creation-3.png?raw=true)

Fill the `database name`, `keyspace name`, `username` and `password`. *Please remember your password as you will be asked to provide it when application start the first time.*

![Astra Database Creation Form](pics/db-creation-4.png?raw=true)

**✅ View your Database and connect**

View your database. It may take 2-3 minutes for your database to spin up. You will receive an email at that point.

**👁️ Expected output**

*Initializing*

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/dashboard-pending-1000.png?raw=true)

Once the database is ready, notice how the status changes from `pending` to `Active` and Astra enables the **connect** button.

![my-pic](https://github.com/datastaxdevs/shared-assets/blob/master/astra/dashboard-withdb-1000.png?raw=true)

