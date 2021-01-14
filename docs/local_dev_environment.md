# Set up a local development environment

This guide will cover setting up a fence development environment.

The cloud settings are still to be determined.

## Set up Working Directory

Clone the repo locally.

```console
git clone https://github.com/uc-cdis/fence.git
```

Navigate to the cloned repository directory.

## Set up Python 3

The environment was tested with python 3.8 on WSL1.  You can use `bash` to install python 3 if it's not already available.

```console
sudo apt-get update
sudo apt-get install python3
```

### Set up a Virtual Environment

Set up a virtual environment for use with this project using `bash`:

```console
python3 -m venv py38-venv
. py38-venv/bin/activate
```

### Set up Poetry

Install [Poetry](https://python-poetry.org/docs/#installation) for the appropriate environment if it's not already installed.

For this guide, we can use the following:
```console
curl -sSL https://raw.githubusercontent.com/python-poetry/poetry/master/get-poetry.py | python -
```

Another package (in WSL) may be required before running poetry:

```console
sudo apt-get update
sudo apt-get install libpq-dev
sudo apt install python-is-python3
```

Make sure you're in the repository directory as the working directory, which should contain a [poetry.lock file](https://raw.githubusercontent.com/uc-cdis/fence/master/poetry.lock).

Use poetry to install the project requirements:

```console
poetry install -vv
```

## Create local configuration file

You can use [cfg_help](https://raw.githubusercontent.com/uc-cdis/fence/master/cfg_help.py) to create a configuration file for testing.

```console
python cfg_help.py create
```

Make note of the path for the created configuration file.  You can make edits to this file for testing purposes.

You can get the path for the newly created config file:

```console
python cfg_help.py get
```

## Set up local Postgresql DB for testing

You can use a local postgresql for testing purposes.

### Set up local Postgresql DB on WSL

You can use `bash` to install postgres:

```console
sudo apt install postgresql-client-common
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
sudo apt-get install postgresql-12
```

Make sure the cluster is started:

```console
sudo pg_ctlcluster 12 main start
```

### Set up local Postgresql DB on Mac

If you're on mac, you can install postgres using brew:

```console
brew install postgres
```

### Update configuration settings in fence-config.yaml 

Fill in the details in the fence-config.yaml:

```python
import base64
import os
key = base64.urlsafe_b64encode(os.urandom(32))
print(key)
```

Copy the key for the ENCRYPTIONKEY value for use in the yaml file:

```yaml
# //////////////////////////////////////////////////////////////////////////////////////
# GENERAL
#   - Fill out all variables!
# //////////////////////////////////////////////////////////////////////////////////////
APP_NAME: 'Gen3 Data Commons'
# Where fence microservice is deployed
BASE_URL: 'http://localhost/user'
# postgres db to connect to
# connection url format:
#     postgresql://[user[:password]@][netloc][:port][/dbname]
DB: 'postgresql://postgres:postgres@localhost:5432/fence_test_tmp'

ENCRYPTION_KEY: 'COPIED_VALUE_FOR_BASE64_ENCODED_VALUE'
```

Note, the **DB** value is also set to a connection string, in this case you can configure it to use `'postgresql://postgres:postgres@localhost:5432/fence_test_tmp'` which will reflect the postgres user and database settings that you can set up next.

### Set up DB and users for testing

You'll need to connect to the postgresql and add test users and databases.

#### Connect to Postgresql on WSL

Connect to the local postgresql server

```console
sudo -i -u postgres
psql
```

#### Connect to Postgresql on Mac

If you're on a mac, use the following to connect to postgres:

```console
brew services start postgres
psql postgres
```

#### Helpful psql commands
It may be helpful to understand some psql commands too:

```console
\conninfo # check connection info
\l # list databases
\d # list tables in database
\c # list short connection info
\c postgres # connect to a database named postgres
\q # quit
```

#### Set up users and databases in psql

Initialize the database and users within the psql console:

```console
CREATE DATABASE "fence_test_tmp";
\c fence_test_tmp
CREATE USER postgres WITH PASSWORD "test";
ALTER USER postgres WITH PASSWORD "postgres";
\du
```

> You may need to use single quotes instead of double quotes depending on your shell environment.

You can set up the other test database:

```console
\c postgres
CREATE DATABASE "fence_test";
\c fence_test
CREATE USER test WITH PASSWORD "test";
ALTER USER postgres WITH PASSWORD "postgres";
\du
```

> You may need to use single quotes instead of double quotes depending on your shell environment.

## Setup Unit Testing

Based on the earlier poetry set up, you should have the Python dependencies installed.  Be sure to check for `pytest` in the listed packages using:

```console
python3 -m pip freeze
```

Navigate to the [.\fence\tests](https://github.com/uc-cdis/fence/tree/master/tests) directory.

```console
python3 -m pytest 
```

If the tests are able to run, you should be able to check for the existence of user-defined functions in the local postgresql db:

```console
sudo -i -u postgres
psql
\c fence_test_tmp

SELECT p.proname 
FROM pg_proc p 
left join pg_namespace n on p.pronamespace = n.oid 
where n.nspname not in ('pg_catalog', 'information_schema');
```

### Configure Local Tests

You may also need to update the [test-fence-config.yaml](https://github.com/uc-cdis/fence/tree/master/tests/test-fence-config.yaml) with the appropriate database connection string:

```yaml
DB: 'postgresql://username:password@hostname:port/fence_test_tmp'
```

You may consider the following sample values; of course use the appropriate settings to connect to psql:

| Name       | Value     |
| :------------- | :----------: |
| username | postgres |
| password | postgres |
| hostname | localhost |
| port | 5432 |

Further you may also need to clear out pycached folders.

```console
find . \( -name '__pycache__' -or -name '*.pyc' \) -delete
```

### Set up a test certificate

You can also generate a test certificate by running the following from your repository root directory:

```console
mkdir -p tests/resources/keys; cd tests/resources/keys; openssl genrsa -out test_private_key.pem 2048; openssl rsa -in test_private_key.pem -pubout -out test_public_key.pem
openssl genrsa -out test_private_key_2.pem 2048; openssl rsa -in test_private_key_2.pem -pubout -out test_public_key_2.pem
cd -
```