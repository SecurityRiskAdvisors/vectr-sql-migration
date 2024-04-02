# VECTR MongoDB to Postgres Migration Tool

The purpose of this tool is to convert a VECTR 8.x MongoDB to a VECTR 9.0 Postgres DB. 

**All of these instructions assume you're running a standard community edition install with docker-compose on Ubuntu. If you have deviated from this standard install you will need to adjust accordingly**

## Directory Configuration

1. Create a directory for the tool to run in, for this example we'll use /opt/vectr-sql-migration 
    ```shell
    mkdir /opt/vectr-sql-migration
    ```

2. Download and unzip the migration zip

    ```shell
   cd /opt/vectr-sql-migration
   wget https://github.com/SecurityRiskAdvisors/vectr-sql-migration/releases/download/ce-1.0.0/sqlmigration-1.0.0.zip
   unzip sqlmigration-1.0.0.zip
    ```
3. Create required directories

    ```shell
   mkdir -p /opt/vectr-sql-migration/user/mongo
   mkdir -p /opt/vectr-sql-migration/sqlmigrations
   mkdir -p /opt/vectr-sql-migration/mongodumps
    ```
   
## Run Mongodump and Extract

1. Run the perform_mongodump.sh script with parameters as follows. 
   1. Following -d insert the VECTR directory, in this example we will assume /opt/vectr 
   2. Following the -o insert the directory created previously, in our example /opt/vectr-sql-migration/mongodumps
       ```shell
      ./perform_mongodump.sh -d /opt/vectr -o /opt/vectr-sql-migration/mongodumps
       ```
2. Extract the contents to the user/mongo directory created previously, in our example /opt/vectr-sql-migration/user/mongo Note you will need the tar filename output from the perform_mongodump.sh script. This should be in a time format, for example 2024_02_05__22_39_53.tgz
   ```shell
   tar -zxvf /opt/vectr-sql-migration/mongodumps/2024_02_05__22_39_53.tgz -C /opt/vectr-sql-migration/user/mongo/
   ```
   
At this point you should have the dump extracted. To confirm the folder structure is correct look for the following GoldStandard folder, in our example /opt/vectr-sql-migration/user/mongo/GoldStandard

If so you are ready to proceed, if not look for any unnecessary nested folders until it is as above. 

## Run Migration

**Warning: There are new constraints on the size of some fields in the sql data model that previously did not exist.
The default behavior of the migration tool is to truncate large strings to conform to the new schema.
If you wish to change this, in the .env file set FORCE_MIGRATE=false This will cause the migration tool to exit with errors instead of modifying data.**

1. From your vectr-sql-migration directory run the docker-compose. Note we're running this command without the -d. We must observe the migration running in case of any errors. If you do see a violation it will print to the log. Open an issue under this project using the issue template. 

 ```shell
docker compose up
```
   
This may take some time to process depending on the speed of your VM. A successful migration will show this at the end:
```shell
vectr-migration_1  | DONE MIGRATION
vectrmigration1_vectr-migration_1 exited with code 0
```

## Clean up 

1. In the case you have a successful migration, you will want to Ctrl + C to stop the containers. Then to remove the migration docker envionment **Warning: This command destroys the volumes used for migration, which should no longer be needed. If you run this command in the incorrect directory you may delete data inadventently. Ensure you are in the directory with the sql-migration docker-compose.yml** 
```shell
docker compose down -v
```

## Load into VECTR 9.0

See our full Docs site to continue this guide,

https://docs.vectr.io/postgresmigration/
