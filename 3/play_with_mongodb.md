# Домашняя работа по теме № 3

## Цель: 
+ В результате выполнения ДЗ вы научитесь разворачивать MongoDB, заполнять данными и делать запросы.

## Задание: 
+ установить MongoDB одним из способов: ВМ, докер;
+ заполнить данными;
+ написать несколько запросов на выборку и обновление данных

### Установка и запуск mongodb (docker-compose) на локальной машине
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
4. Что бы проверить запущен ли контейнер, можно выполнить команду 
`docker ps | grep mongodb`

### Заливаем тестовые данные в mongodb
1. Выполним одну супер команду, которая скачает проект с тестовыми данными, распакует и зальет в  mongodb `wget -P mongo-shared/ https://github.com/neelabalan/mongodb-sample-dataset/archive/refs/heads/main.zip && unzip mongo-shared/main.zip -d mongo-shared/ && docker exec mongodb sh -c 'cd /data/files/mongodb-sample-dataset-main && ./script.sh 0.0.0.0 27017 root PaSSw0rd123!'`

### Подключаемся к mongodb через утилиту mongosh
1. Подключаемся к mongodb, выполнив команду `docker exec -ti mongodb mongosh mongodb://0.0.0.0:27017/ -u root -p PaSSw0rd123!`

### Делаем различные запросы в mongodb
#### Общие команды
1. Просмотреть список всех баз данных `show databases`
2. Переключиться на базу данных test `use test`
3. Просмотреть список коллекций `show collections`
4. Посмотреть все индексы коллекции `db.users.getIndexes()`
#### Делаем запросы
1. Перейдем в пространство sample_analytics `use sample_analytics`
2. Посмотрим на список коллекций `show collections`
3. Выведем все документы из коллекции customers `db.customers.find()`, структура документа будет следующего вида:
```
  {
    _id: ObjectId('5ca4bbcea2dd94ee58162a7c'),
    username: 'michael58',
    name: 'Christine Douglas',
    address: 'USNV Chavez\nFPO AP 78727',
    birthdate: ISODate('1989-12-25T23:58:01.000Z'),
    email: 'aaron99@yahoo.com',
    accounts: [ 177069, 233104, 671035, 575454, 285919, 947160 ],
    tier_and_details: {
      '14540f8bc96947fdb620f52768acce16': {
        tier: 'Gold',
        benefits: [ 'sports tickets' ],
        active: true,
        id: '14540f8bc96947fdb620f52768acce16'
      },
      '6f4836805a744592a2193a45e9801038': {
        tier: 'Silver',
        benefits: [ 'car rental insurance', 'concert tickets' ],
        active: true,
        id: '6f4836805a744592a2193a45e9801038'
      }
    }
  }
```
4. Найдем все документы, где в `accounts` записаны только 3 значения, для этого выполним команду `db.customers.find({ accounts: { $size: 3 } })`, что бы посчитать сколько таких докуменов `db.customers.find({ accounts: { $size: 3 } }).count()`
5. Найти количество документы, где birthdate <= 01.01.1990 `db.customers.find({ birthdate: { $lte: ISODate("1990-01-01T00:00:00Z") } }).count()`
6. Поиск с ограниченным отображением полей username, name, email `db.customers.find({ birthdate: { $lte: ISODate("1990-01-01T00:00:00Z") } }, { username: 1, name: 1, email: 1, _id: 0 })`
7. Вывести все документы, где в поле accounts встречается одно из значений [575454, 86702, 771641, 327942] `db.customers.find({accounts : {$in : [575454, 86702, 771641, 327942]}})`
##### А вот с рекурсивными запросами у меня не получилось, очень жаль
6. Вывести все документы с отображением вложенных полей tier `db.customers.find({}, { "$**.tier": 1 })`
7. Вывести все документы, где поле benefits существует `db.customers.find({ "tier_and_details.$**.benefits": { $exists: true } })`

### Создаем новую коллекцию и документы в mongodb
#### Общие команды
1. Просмотреть список всех баз данных `show databases`
2. Переключиться на базу данных test `use test`
3. Просмотреть список коллекций `show collections`
4. Посмотреть все индексы коллекции `db.users.getIndexes()`
#### Создаем новую коллекцию с индексом
5. Создадим новую коллекцию users `db.createCollection("users")`
6. Для быстрого find, update, delete создадим уникальный индекс по полю `email`, для этого выполним команду `db.users.createIndex({ email: 1 }, { unique: true })`
#### Добовляем документы в коллекцию
7. Добавим один документ в коллекцию `db.users.insertOne({ name: "Alice", email: "alice@example.com", age: 25 })`
8. Добавим несколько документов в коллекцию `db.users.insertMany([{ name: "Bob", email: "bob@example.com", age: 30 }, { name: "Charlie", email: "charlie@example.com", age: 35 }])`
#### Обновлям документы в коллекции
9. Обновить первый документ, где name = 'Alice' `db.users.updateOne({ name: "Alice" }, { $set: { age: 26 } })`
10. Обновить все документы, где age > 30 `db.users.updateMany({ age: { $gt: 30 } }, { $set: { status: "active" } })`
#### Удаляем документы из коллекции
11. Удалить первый документ, где name = 'Bob' `db.users.deleteOne({ name: "Bob" })`
12. Удалить все документы, где age > 30 `db.users.deleteMany({ age: { $gt: 30 } })`
#### Поиск документов по коллекции
13. Найти документы через регулярное выражение, к примеру найти все записи, где в поле email встречаются слово 'bob' `db.users.find({ email: /bob/ })`
#### Проверка ограничений индекса
14. Попытка добавить документы с существующим email `db.users.insertOne({ name: "Alice", email: "alice@example.com", age: 25 })`, будет ошибка дубликации по полю email - E11000 duplicate key error collection: test.users index: email_1 dup key: { email: "alice@example.com" }


