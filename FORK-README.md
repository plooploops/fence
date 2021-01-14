# README 

This fork is to add support for Azure Storage Blobs as data file URLs. 

# The Approach
* Follows the same pattern for S3 and GCP storage support.
  * Effectively a switch statement based on the URL
  * Use cloud platform specific code to generate a pre-signed URL on download

# Known Issues

* The current logic will not support cases where the a custom domain is mapped to the Azure Blob Storage endpoint. For instance
`https://mystorageaccount.blob.core.windows.net/mycontainer/myblob` becomes
`http://<subdomain.customdomain>/mycontainer/myblob`. Moreover custom domains may or may not use https.
* Does not support UPLOADs or multi-part UPLOADS
* Does not support "requester pays for signed urls"
* Fence seems very oriented to having schemes (e.g. `s3` and `gs`). Azure Blob storage doesn't use a custom scheme, just `https` 

## Building Docker Image and publishing to DockerHub

* Run `docker build -t andrebriggs/fence:latest .` then 
* Will create an Azure Pipeline for this soon.


### Prerequisites
* Local postgres DB running

We are going to simulate the commands from the [.travis.yaml](./.travis.yaml) file to create the DB:

```yaml
install:
  - ...
  - psql -c 'SELECT version();' -U postgres
  - psql -U postgres -c "create database fence_test_tmp"
```

Further, be sure that you setup a [local developmement environment](./docs/local_dev_environment.md)

## Running local tests
From within `./tests` run `poetry run pytest -vv --cov=fence --cov-report html`. This will generate a code coverage report. Currently all added code is covered.

### If running Docker Compose

If you are running compose services your default postgres port is going to be overtaken by the docker compose.
Connect to the docker compose instance and create the new database `fence_test_tmp`. First log into the postgres:
```
$ psql -Atx postgresql://postgres:postgres@localhost:5432
psql (13.1, server 9.6.20)
Type "help" for help.

postgres=# \conninfo
You are connected to database "postgres" as user "postgres" on host "localhost" (address "::1") at port "5432".
```
Next create the `fence_test_tmp` DB in the postgress shell context:
```
postgres=# CREATE DATABASE fence_test_tmp;
```

Verify the DB is creates in the postgres shell using 
```
postgres=# \l
<!-- (See all databases including) -->
```


<!-- Start postgres locally
`$ pg_ctl -D /usr/local/var/postgres start`

$ pg_ctl -D /usr/local/var/postgres stop

If you unstalled psql via homebrew you need to create the user `postgres`. Run:
`/usr/local/opt/postgres/bin/createuser -s postgres` -->

<!-- Then type  

psql -Atx postgresql:///fence_test_tmp?host=/tmp where /tmp is the socket
```
$ psql -Atx postgresql:///fence_test_tmp?host=/tmp
psql (13.1)
Type "help" for help.

fence_test_tmp=# \conninfo
You are connected to database "fence_test_tmp" as user "andrebriggs" via socket in "/tmp" at port "5432".
``` -->

<!-- If you have multiple instances of postgres running locally you can find the data directory by using teh command 
`SHOW data_directory;`

```
$ psql -Atx postgresql://postgres:postgres@localhost:5432
psql (13.1, server 9.6.20)
Type "help" for help.

postgres=# SHOW data_directory;
data_directory|/var/lib/postgresql/data
```

See teh difference:

```
$ psql -Atx postgresql:///fence_test_tmp?host=/tmp
psql (13.1)
Type "help" for help.

fence_test_tmp=# SHOW data_directory;
data_directory|/usr/local/var/postgres
``` -->