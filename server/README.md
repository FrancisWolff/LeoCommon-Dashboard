# LeoCommon-Dashboard


## Initial Setup

#### Required software:
 - PostgreSQL
 - LeoCommon server (https://github.com/LeoCommon/server#)
 - iridium-toolkit (https://github.com/muccc/iridium-toolkit)

#### Setup PostgreSQL:

1. Install PostgreSQL

    $ `apt install postgresql`
2. Login as superuser or create own user account with sufficient privileges

    $ `sudo -u postgres psql postgres`
3. Change the password for the superuser to something secure ( ! important step, default password "postgres" not secure ! )

    $ `\password postgres`
3. Create database with name postgres

    $ `createdb postgres`
4. Enter database

    $ `psql postgres`
5. Create tables

```
    CREATE SCHEMA public AUTHORIZATION pg_database_owner;

    CREATE SEQUENCE sensor_job_id_seq
        INCREMENT BY 1
        MINVALUE 1
        MAXVALUE 2147483647
        START 1
        CACHE 1
        NO CYCLE;

    CREATE TABLE jobs (
        "name" text NOT NULL,
        command text NULL,
        start_time int4 NULL,
        end_time int4 NULL,
        CONSTRAINT jobs_pkey PRIMARY KEY (name)
    );

    CREATE TABLE sensor_job (
        id serial4 NOT NULL,
        job_name text NULL,
        sensor_name text NULL,
        lat float8 NULL,
        lon float8 NULL,
        sample_rate int4 NULL,
        center_freq int4 NULL,
        bandwidth int4 NULL,
        gain int4 NULL,
        if_gain int4 NULL,
        bb_gain int4 NULL,
        decimation int4 NULL,
        CONSTRAINT sensor_job_job_name_sensor_name_key UNIQUE (job_name, sensor_name),
        CONSTRAINT sensor_job_pkey PRIMARY KEY (id),
        CONSTRAINT sensor_job_job_name_fkey FOREIGN KEY (job_name) REFERENCES jobs("name")
    );

    CREATE TABLE signal (
        id int4 NOT NULL,
        "timestamp" float8 NOT NULL,
        signal_level float4 NULL,
        background_noise float4 NULL,
        snr float4 NULL,
        count int4 NULL,
        CONSTRAINT signal_pkey PRIMARY KEY (id, "timestamp"),
        CONSTRAINT signal_id_fkey FOREIGN KEY (id) REFERENCES sensor_job(id)
    );

    CREATE TABLE stderr (
        id int4 NOT NULL,
        "timestamp" float8 NOT NULL,
        i int4 NULL,
        o int4 NULL,
        ok_s int4 NULL,
        ok int4 NULL,
        CONSTRAINT stderr_pkey PRIMARY KEY (id, "timestamp"),
        CONSTRAINT stderr_id_fkey FOREIGN KEY (id) REFERENCES sensor_job(id)
    );

    CREATE TABLE packets (
        id int4 NOT NULL,
        "type" text NOT NULL,
        count int4 NULL,
        CONSTRAINT packets_pkey PRIMARY KEY (id, type),
        CONSTRAINT packets_id_fkey FOREIGN KEY (id) REFERENCES sensor_job(id)
    );
```

#### Setup LeoCommon:

1. Follow the setup instructions at https://github.com/LeoCommon/server#
2. Create a dedicated dashboard account with user privileges on the server, its name and password will be relevant later

#### Setup iridium-toolkit:
1. In the root directory of the server, create tools folder

    $ `mkdir tools`
2. Enter folder

    $ `cd tools`

2. Clone the repository

    $ `git clone https://github.com/muccc/iridium-toolkit.git`

#### Setup Dashboard:
1. In the directory where the root of the server is located, clone the dashboard repository

    $ `git clone https://github.com/FrancisWolff/LeoCommon-Dashboard.git`
2. Copy the dashboard server folder into the LeoCommon server folder and merge all folders/ replace all files

    $ `cp -R ./LeoCommon-Dashboard/server/* ./server`
3. Remove the LeoCommon-Dashboard directory

    $ `rm -r -f ./LeoCommon-Dashboard`
4. Install dashboad dependencies: in root folder of the server, activate the virtual environment

    $ `source env/bin/activate`
5. Install requirements

    $ `pip install -r requirements.txt`
6. Deactivate virtual environment

    $ `deactivate`
7. Locate the `.env` file in the `/env` directory, add the following lines and fill the empty quotation marks with own values

    `DASH_DB_USER=""` the name of the postgres user ("postgres" or own user account)

    `DASH_DB_PASSWORD=""` the password of the postgres user

    `DASH_USER=""` the name of the dashboard user from step 2 in Setup LeoCommon

    `DASH_PASSWORD=""` the password of the dashboard user from step 2 in Setup LeoCommon
8. Setup is now complete, the data_daemon will start every day at midnight to check for new data and add it to the dashboard database

