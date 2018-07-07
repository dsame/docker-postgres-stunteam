#Notes about docker
There's no `curl` installed because alpine base image contains `wget` by default

#Configuration options:

There are 3 phases to be configured: 1. building the docker, 2. running the postgres for 
the very first time and init db, 3. up the postgres daemon.

The first phase implemented in Dockerfile and the 2dn and 3rd in `docker-entrypoint.sh`

## Phase of building the docker
To choose PG version use PG_VERSION build arg
```
docker build --build-arg PG_VERSION=11beta2 ...
```
If PG_VERSION is not set the recent version will be fetched from https://www.postgresql.org/ftp/source/

To pass additional arguments to `./configure` use build arg CONFIGURE_OPTS:
```
docker build --build-arg CONFIGURE_OPTS=--some-configure-opt ...
```

To change default data dir from its default value '/var/lib/postgresql/data':
```
docker build --build-arg PGDATA=/path_to_db ...

```

## Running the docker container for the first time
The "first time run" is detected by the looking for `$PGDATA/PG_VERSION` file. If it does not
exist the `initdb` will be run.

There is mandatory options to init postgres db: `POSTGRES_PASSWORD` 
```
docker run -e POSTGRES_PASSWORD=secret ...

```

Also PGDATA may be changed from its default value set in the build time:
```
docker run -e POSTGRES_PASSWORD=secret -ePGDATA=/path_to_db -v/tmp/path_to_db_on_host:/path_to_db ...

```

To change the initdb behaviour it is possible to pass the additional options with
```
docker run -e POSTGRES_INITDB_ARGS=args...

```
see https://postgrespro.com/docs/postgresql/9.6/app-initdb.html

Passing env. variables `POSTGRES_USER` and `POSTGRES_DB` will create the database owned by
the user with super permissions.

And finally as soon as the database is created the scripts from `/docker-entrypoint-initdb.d/` 
to be executed. Thus in order to customize postgress instansce mount this directory to host system with
the necessary *.sh, *.sql or *.sql.gz files preloaded.
```
docker run -e POSTGRES_PASSWORD=secret -v/docker_entry_point_on_host:/docker-entry-point-initdb.d ...
```


## Full life cycle sample command:
```
m -rf /tmp/pgtest;docker build --build-arg PGDATA='/test' -t t . && docker stop pg;docker rm pg;docker run --name pg -e POSTGRES_PASSWORD=secret -ePGDATA='/test' -ePOSTGRES_USER=tuser -ePOSTGRES_DB=tdb -v/tmp/pgtest:/test  t
```
