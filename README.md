# airflow-postgres-deployment
This guide contains instructions on how to setup airflow with postgres backend.
It assumes you have basic knowledge about airflow, postgres. Please consult other tutorials and documentations about how to get started with these tools.
This guide also assumes you are setting up airflow as airflow user, in its own home folder.

# Install
By default, airflow uses sqlite3 database backend, which only support "SequentialExecutor". Postgres backend support "LocalExecutor", which parallelly execute tasks on local CPU resurces.
I choose conda to install python3, airflow, postgres database, and other python libraries to keep the airflow setup isoloated.
```
conda install postgres
pip install apache-airflow
pip install apache-airflow[postgres]
```

# Config (Before running postgres or airflow)
## Add to .bashrc
Let's setup environment variables required by airflow and PostgresSQL.
```
# postgre setup by zly
export PGDATA='/home/airflow/postgres'
# apache airflow
export AIRFLOW_HOME='/home/airflow/airflow'
```

## PostgreSQL database setup
Under psql prompt:
```
psql> CREATE DATABASE airflow
```

## PostgreSQL password setup
Enter postgresSQL prompt
```
psql
```

Set password for your airflow user, follow the instruction to set new password:
```
psql> \password
```

## Modify postgres/pg_hba.conf
The postgres database is only intended for airflow user to access. Hence the following
```
# "local" is for Unix domain socket connections only
local   all             airflow                                  peer
## IPv4 local connections:
host    all             airflow         127.0.0.1/32             md5
## IPv6 local connections:
host    all             airflow         ::1/128                  md5
```

## Modify airflow.conf
Change our airflow executor and database backend:
```
# executor = SequentialExecutor
executor = LocalExecutor
...
# sql_alchemy_conn = sqlite3://SOMETHING_I_DONT_REMEMBER
sql_alchemy_conn = postgresql+psycopg2://airflow:YOUR_PASSWORD@localhost:5432/airflow
```

## Airflow webserver password setup
Modify airflow.conf:
```
authenticate = True
auth_backend = airflow.contrib.auth.backends.password_auth
```
Setting password via python
```
py> import airflow
py> from airflow import models, settings
py> from airflow.contrib.auth.backends.password_auth import PasswordUser
py> user = PasswordUser(models.User())
py> user.username = 'YOUR_USER_NAME'
py> user.password = 'YOUR_PASSWORD'
py> session = settings.Session()
py> session.add(user)
py> session.commit()
py> session.close()
py> exit()
```

# Usage
Start your applications in the following order:
```
postgres # This will start postgres using $PGDATA as the default database folder
airflow scheduler
airflow webserver
```

# Systemd service files
Three systemd service files are included so postgres and airflow services can start during system reboot automatically, in the right order.
Put them in, typically, /etc/systemd/system folder and use `systemctl` to further enable them.
