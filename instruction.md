### Цель инструкции:
- дать возможность студентам заниматься интересными подзадачами - автозагрузка по http регулярная, создание ДАГа, и т.п. Благодаря тому, что время не мелочи не утекает.
### Статус инструкции:
- проверено и работает все, кроме последнего создания таблицы
- осталось поправить: помеченное 🚧, и привести к одному языку (англ или русский).


## Step 0: VM
connect to VM in terminal 
ssh <VM_user>:<VM_ip>

## Step 1: Docker

1.1 Install Docker
If Docker is not installed on your system, install it by following the official Docker documentation for your operating system.
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04

1.2: Create a Docker Network
 
```
docker network create my_network
```

1.3: Create a Volume for PostgreSQL Data
 
```
docker volume create postgres_1_vol
``` 

1.4: Run PostgreSQL Container
 
```
docker run -d \
--name postgres_1 \
-e POSTGRES_USER=postgres \
-e POSTGRES_PASSWORD=123 \
-e POSTGRES_DB=test_app \
-v postgres_1_vol:/var/lib/postgresql \
--network my_network \
postgres:15
```
🧨💪 "The /var/lib/postgresql/data directory is typically where PostgreSQL stores its data files." - это пишут, но если использовать такой путь, то вызываются ошибки. Если делать без data, то этот этап в порядке.
💪 строка --network my_network позволяет автоматом добавить контейнеры в сеть (это минус 2 строки кода ниже).
💡 user, password: юзер оставляем, пароль можно поменять

1.5: Run Superset Container
 
```
docker run --rm -d \
-p 8088:8088 \
-e "SUPERSET_SECRET_KEY=$(openssl rand -base64 42)" \
--name superset \
--network my_network \
apache/superset
```

(optional) 1.6 Connecting Containers to Network
The containers should already be on the same network (my_network) as specified during their creation. If not, you can manually connect them:
 
```
docker network connect my_network postgres_1
docker network connect my_network superset
```

## Step 2: Initialize Superset
2.1 Create an Admin User:
 
```
docker exec -it superset superset fab create-admin \
--username admin \
--firstname Superset \
--lastname Admin \
--email admin@superset.com \
--password admin
```
💡 логин-пароль admin/admin лучше оставить, тк для внешнего юзера удобно (🚧 или есть возражения?)

2.2 Upgrade Superset Database:
 
```
docker exec -it superset superset db upgrade
```

2.3 Initialize Superset Services:
 
```
docker exec -it superset superset init
```

## Step 3: Set Up SSH Tunneling (Optional for Remote Access)
❗️ in local terminal (in VSCode etc)
найти по своей виртуалке - юзернейм и IP
 
```
ssh -L 8080:localhost:8088 <user_name>@<ip_address>
```
### Настроить подключение БД и Superset 
Зайти на http://localhost:8080.

В Superset click on +, поля заполнить так:
- 💪в полe ip вставить либо ip (именно БД, а не ВМ - см чуть ниже), либо более простой способ 💪 - вставить имя докера для БД. Выше тут было --name postgres_1. Вставляем postgres_1
- порт 5432
- login-pw - from step with postgres (дописать 🚧): postgres, 123
- Database: test_app (имя, которое указали тут в начале мануала)



Для поля IP, Как найти IP postgres: 

```
docker inspect <имя_сети>
```
name - from step "Create a Docker Network"
see IP in postgres part (below)


## Step 4. Postgres

Зайти в контейнер, скачать, распаковать и добавить в Postgres базу прямо там. Заходим в контейнер и запускаем bash внутри:
```
docker exec -it postgres_1 bash

```
- postgres_1 это name контейнера, заменить если у вас другое.

Переход в папку ❗️
cd /var/lib/postgresql/

Обновляем и устанавливаем пакеты: 
❗️важно тк часто wget не установлен
```
apt-get update
apt-get install -y wget
```

Скачиваем базу:
wget https://9c579ca6-fee2-41d7-9396-601da1103a3b.selstorage.ru/credit_clients.csv

Переключаемся на пользователя postgres и создаем таблицу для базы: 
❗️
```
su postgres
```
```
psql 
```
❗️ последнее важно - then starts the PostgreSQL CLI, allowing you to execute SQL commands like creating tables and copying data from the CSV file.


```
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
    Exited INTEGER
);
```


Копировать данные: 
```
\copy customer_data FROM '/var/lib/postgresql/credit_clients.csv' DELIMITER ',' CSV HEADER;
```
🚧🚧 тут ошибка на customer_data

```

