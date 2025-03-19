# Домашняя работа по теме № 3

## Цель: 
+ В результате выполнения ДЗ вы научитесь разворачивать MongoDB, заполнять данными и делать запросы.

## Задание: 
+ установить MongoDB одним из способов: ВМ, докер;
+ заполнить данными;
+ написать несколько запросов на выборку и обновление данных

### Установка, запуск и подключение к mongodb (docker-compose)
1. Создаем файл docker-compose.yml и заполняем его следующим содержимым:
        
        version: '3.8'

        services:
        mongo:
            image: mongo:latest
            container_name: mongodb
            restart: always
            environment:
            MONGO_INITDB_ROOT_USERNAME: root
            MONGO_INITDB_ROOT_PASSWORD: PaSSw0rd123!
            ports:
            - "27017:27017"
2. При необходимости меняем имя пользователя, пароль или порт подключения к базе данных, редактируем соответствующие строки:
    - MONGO_INITDB_ROOT_USERNAME: root
    - MONGO_INITDB_ROOT_PASSWORD: PaSSw0rd123!
    - "27017:27017", где "HOST_PORT:CONTAINER_PORT"
3. Для запуска контейрена с базой данных выполняем команду `mkdir mongo-shared && docker-compose up -d` из директории, где расположен файл docker-compose.yml. Вывод будет следующий:
4. Что бы проверить, запущен ли контейнер можно выполнить команду 
`docker ps | grep mongodb`, вывод будет следующий:
5. Для подключения к mongodb используем утилиту `название`

### Заливаем тестовые данные в mongodb
1. Выполним одну супер команду, которая скачает проект с тестовыми данными, распакует и зальет в  mongodb `wget -P mongo-shared/ https://github.com/neelabalan/mongodb-sample-dataset/archive/refs/heads/main.zip && unzip mongo-shared/main.zip -d mongo-shared/ && docker exec mongodb sh -c 'cd /data/files/mongodb-sample-dataset-main && ./script.sh 0.0.0.0 27017 root PaSSw0rd123!'`

### Подключаемся к mongodb через утилиту mongosh
1. Подключаемся к mongodb, выполнив команду `docker exec -ti mongodb mongosh mongodb://0.0.0.0:27017/ -u root -p PaSSw0rd123!`

### Делаем различные запросы в mongodb
1. Просмотреть список всех баз данных `show databases`
2. Переключиться на базу данных test `use test`
3. Просмотреть список коллекций `show collections`
4. Создадим новую коллекцию users `db.createCollection("users")`
5. Для быстрого find, update, delete создадим уникальный индекс по полю `email`, для этого выполним команду `db.users.createIndex({ email: 1 }, { unique: true })`
6. Посмотреть все индексы коллекции `db.users.getIndexes()`
7. Добавим один документ в коллекцию `db.users.insertOne({ name: "Alice", email: "alice@example.com", age: 25 })`
8. Добавим несколько документов в коллекцию `db.users.insertMany([{ name: "Bob", email: "bob@example.com", age: 30 }, { name: "Charlie", email: "charlie@example.com", age: 35 }])`
9. Обновить первый документ, где name = 'Alice' `db.users.updateOne({ name: "Alice" }, { $set: { age: 26 } })`
10. Обновить все документы, где age > 30 `db.users.updateMany({ age: { $gt: 30 } }, { $set: { status: "active" } })`
11. Удалить документ, где name = 'Bob' `db.users.deleteOne({ name: "Bob" })`
12. Найти документы через регулярное выражение, к примеру найти все записи, где в поле email встречаются слово 'bob' `db.users.find({ email: /bob/ })`
13. Попытка добавить документы с существующим email `db.users.insertOne({ name: "Alice", email: "alice@example.com", age: 25 })`, будет ошибка дубликации по полю email - E11000 duplicate key error collection: test.users index: email_1 dup key: { email: "alice@example.com" }


