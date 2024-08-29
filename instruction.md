## Step 1: Docker

1.1 Install Docker
If Docker is not installed on your system, install it by following the official Docker documentation for your operating system.
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

1.2: Create a Docker Network
bash
```
docker network create my_network
```

1.3: Create a Volume for PostgreSQL Data
bash
```
docker volume create postgres_1_vol
``` 

1.4: Run PostgreSQL Container
bash
```
docker run -d \
--name postgres_1 \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD='strongpassword' \
-e POSTGRES_DB=test_app \
-v postgres_1_vol:/var/lib/postgresql/data \
--network my_network \
postgres:15
```

1.5: Run Superset Container
bash
```
docker run --rm -d \
-p 8088:8088 \
-e "SUPERSET_SECRET_KEY=$(openssl rand -base64 42)" \
--name superset \
--network my_network \
apache/superset
```

## Step 2: Initialize Superset
2.1 Create an Admin User:
bash
```
docker exec -it superset superset fab create-admin \
--username admin \
--firstname Superset \
--lastname Admin \
--email admin@superset.com \
--password strongpassword
```

2.2 Upgrade Superset Database:

bash
```
docker exec -it superset superset db upgrade
```

2.3 Initialize Superset Services:

bash
```
docker exec -it superset superset init
```

## Step 3. Postgres

3.1: Add PostgreSQL Database to Superset

Access Superset at http://localhost:8088 using your browser.
Login with your Superset admin credentials.
Add a New Database Connection:
Navigate to Data -> Databases -> + Database.
Select PostgreSQL and enter the connection details:
Host: postgres_1
Port: 5432
Database: test_app
Username: postgres
Password: strongpassword
Test the connection and save.

3.2: Adding a CSV File to PostgreSQL
Copy the CSV File to the Container:

bash
```
docker cp /path/to/your/local/credit_clients.csv postgres_1:/var/lib/postgresql/data/
```

Access the PostgreSQL Container:
bash
```
docker exec -it postgres_1 bash
```

Create the Table and Import Data:
bash
```
psql -U postgres -d test_app
CREATE TABLE customer_data (
Date DATE,
CustomerId INTEGER,
Surname TEXT,
CreditScore INTEGER,
Geography TEXT,
Gender TEXT,
Age INTEGER,
Tenure INTEGER,
Balance FLOAT,
NumOfProducts INTEGER,
HasCrCard INTEGER,
IsActiveMember INTEGER,
EstimatedSalary FLOAT,
Exited INTEGER);
\copy customer_data FROM '/var/lib/postgresql/data/credit_clients.csv' DELIMITER ',' CSV HEADER;
```

## Step 4: Connecting Containers to Network
The containers should already be on the same network (my_network) as specified during their creation. If not, you can manually connect them:
bash
```
docker network connect my_network postgres_1
docker network connect my_network superset
```

## Step 5: Set Up SSH Tunneling (Optional for Remote Access)
bash
```
ssh -L 8080:localhost:8088 <user_name>@<ip_address>
```
Access Superset at http://localhost:8080.
