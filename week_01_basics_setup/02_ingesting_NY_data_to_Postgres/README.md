# Step-by-step ingestion

## Ingesting data into Postgres

To ingest the data into Postgres we first need to run Postgres with Docker by running
```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
```

After run the code above, we can connect to Postgres using ```pgcli -h localhost -p 5432 -u root -d ny_taxi```

## Knowing pdAdmin

Now, lets start the pgAdmin container with
```
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  dpage/pgadmin4
```

## Connecting pdAdmin and Postgres

After this, we need to run Postgres and pgAdmin together. To do that, we need to create a network connection: 

- STEP 1: ```docker network create pg-network```

And then,

- STEP 2: run Postgres
```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v /home/coderleo/Desktop/data_engineering_zoomcamp/week_01_basics_setup/02_ingesting_NY_data_to_Postgres/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13
```

- STEP 3: run pdAdmin
```
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin-2 \
  dpage/pgadmin4
```

(Note that in this proccess we need to use two terminals)

## Dockerizing the ingestion script

To ingest the data into our database, the main structure used by the ingess_data.py file is as follow:

```
URL="https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2021-01.parquet"

python3 ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}
```

### Building the image and running it

With the structure above we can build our ingest image and run the respective container:

- STEP 1: ```docker build -t taxi_ingest:v001 .```

- STEP 2:
```
docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
```

### Running multiple containers with docker compose

After ingesting the data into our database, we can interact with Postgres via pdAdmin by running multiple containers.

To do this we use *docker compose*, which specify all the configurations needed to run our containers in the same network.
To do this, we need to specify a *docker-compose.yaml* file and after this run the command ```docker compose up``` to start the containers and ```docker compose down``` to stop then.