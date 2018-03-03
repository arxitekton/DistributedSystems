### Створіть keyspace з найпростішої стратегією реплікації
```
create keyspace lab5 with replication={'class':'SimpleStrategy', 'replication_factor':1};
```

```
use lab5;
```

### таблиця items
```
CREATE TABLE item(
    category text,
    model text,
    producer text,
    price float,
    properties map<text, text>,
    PRIMARY KEY (category, price, model)
);

insert into item (category, model, producer, price, properties) values ('Phone', 'iPhone 6', 'Apple', 600.0, {'size': '5', 'rate': '5'});
insert into item (category, model, producer, price, properties) values ('Monitors', 'HP Pavilion 21.5-Inch IPS LED HDMI VGA', 'HP', 90.0, {'size': '21.5', 'rate': '4'});
insert into item (category, model, producer, price, properties) values ('Monitors', 'HP 23.8-inch FHD IPS', 'HP', 130.0, {'size': '23.8', 'rate': '4.5'});
insert into item (category, model, producer, price, properties) values ('Monitors', 'Samsung CHG90 Series Curved 49', 'Samsung', 1039.0, {'size': '49.0', 'rate': '5'});
insert into item (category, model, producer, price, properties) values ('Phone', 'Redmi Note 4', 'Xiaomi', 300.0, {'size': '5', 'version': 'China', 'rate': '4'});
insert into item (category, model, producer, price, properties) values ('Phone', 'Redmi Note 4x', 'Xiaomi', 300.0, {'size': '5', 'version': 'Global', 'rate': '4'});
```
### Напишіть запит, який показує структуру створеної таблиці (команда DESCRIBE)
```
describe table lab5.item
```

#### responce:
```
CREATE TABLE lab5.item (
    category text,
    price float,
    model text,
    producer text,
    properties map<text, text>,
    PRIMARY KEY (category, price, model)
) WITH CLUSTERING ORDER BY (price ASC, model ASC)
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '64', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND crc_check_chance = 1.0
    AND dclocal_read_repair_chance = 0.1
    AND default_time_to_live = 0
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair_chance = 0.0
    AND speculative_retry = '99PERCENTILE';
```
### Напишіть запит, який виведіть усі товари в певній категорії відсортовані за ціною
```
SELECT * FROM lab5.item WHERE category = 'Phone' ORDER BY price;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+-------------------------------------------------
    Phone |   300 |  Redmi Note 4 |   Xiaomi |  {'rate': '4', 'size': '5', 'version': 'China'}
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'rate': '4', 'size': '5', 'version': 'Global'}
    Phone |   600 |      iPhone 6 |    Apple |                      {'rate': '5', 'size': '5'}

(3 rows)
```
### Напишіть запити, які вибирають товари за різними критеріями в межах певної категорії:
 - назва, 
 - ціна (в проміжку), 
 - ціна та виробник 
```
SELECT * FROM lab5.item WHERE category = 'Phone' AND model = 'Redmi Note 4x' ALLOW FILTERING;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+-------------------------------------------------
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'rate': '4', 'size': '5', 'version': 'Global'}

(1 rows)
```
```
SELECT * FROM lab5.item WHERE category = 'Phone' AND price >= 200 AND price <= 400;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+-------------------------------------------------
    Phone |   300 |  Redmi Note 4 |   Xiaomi |  {'rate': '4', 'size': '5', 'version': 'China'}
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'rate': '4', 'size': '5', 'version': 'Global'}

(2 rows)
```
```
SELECT * FROM lab5.item WHERE category = 'Phone' AND price <= 400 AND producer = 'Xiaomi' ALLOW FILTERING;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+-------------------------------------------------
    Phone |   300 |  Redmi Note 4 |   Xiaomi |  {'rate': '4', 'size': '5', 'version': 'China'}
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'rate': '4', 'size': '5', 'version': 'Global'}

(2 rows)
```
### Напишіть запити, які вибирають товари за:
 - наявність певних характеристик
 - певна характеристика та її значення
```
SELECT * FROM lab5.item WHERE properties CONTAINS KEY 'version' ALLOW FILTERING;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+-------------------------------------------------
    Phone |   300 |  Redmi Note 4 |   Xiaomi |  {'rate': '4', 'size': '5', 'version': 'China'}
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'rate': '4', 'size': '5', 'version': 'Global'}

(2 rows)
```
```
SELECT * FROM lab5.item WHERE properties CONTAINS KEY 'version' AND properties['version'] = 'Global' ALLOW FILTERING;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+-------------------------------------------------
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'rate': '4', 'size': '5', 'version': 'Global'}

(1 rows)
```
### Оновить опис товару:
 - змінить існуючі значення певної характеристики 
 - додайте нові властивості (характеристики) товару
 - видалить характеристику товару
```
UPDATE lab5.item SET properties['version'] = 'International' WHERE category = 'Phone' AND price = 300.0 AND model = 'Redmi Note 4x' IF EXISTS;
```
#### responce:
```
 [applied]
-----------
      True
```
```
UPDATE lab5.item SET properties = properties + {'color' : 'red'} WHERE category = 'Phone' AND price = 300.0 AND model = 'Redmi Note 4x' IF EXISTS;
```
#### responce:
```
 [applied]
-----------
      True
```
#### let's check:
```
SELECT * FROM lab5.item WHERE properties CONTAINS KEY 'version' AND properties['version'] = 'International' ALLOW FILTERING;
```
#### responce:
```
 category | price | model         | producer | properties
----------+-------+---------------+----------+------------------------------------------------------------------------
    Phone |   300 | Redmi Note 4x |   Xiaomi | {'color': 'red', 'rate': '4', 'size': '5', 'version': 'International'}

(1 rows)
```
```
UPDATE lab5.item SET properties = properties - {'color'} WHERE category = 'Phone' AND price = 300.0 AND model = 'Redmi Note 4x' IF EXISTS;
```
#### responce:
```
 [applied]
-----------
      True
```
### Створіть таблицю orders 
```
CREATE TABLE orders(
    email text,
    name text,
    date timestamp,
    total float,
    map_items map<text, int>,
    PRIMARY KEY (email, date)
);

INSERT INTO lab5.orders (email, name, date, total, map_items) VALUES ('jsmith.gmail.com', 'John Smith', toTimestamp(now()), 2239.0, {'iPhone 6': 2, 'Samsung CHG90 Series Curved 49': 1});
INSERT INTO lab5.orders (email, name, date, total, map_items) VALUES ('jsmith.gmail.com', 'John Smith', toTimestamp(now()), 300.0, {'Redmi Note 4x': 1});
INSERT INTO lab5.orders (email, name, date, total, map_items) VALUES ('!.gmail.com', 'Chuck Norris', toTimestamp(now()), 10390.0, {'Samsung CHG90 Series Curved 49': 10});
INSERT INTO lab5.orders (email, name, date, total, map_items) VALUES ('!.gmail.com', 'Chuck Norris', toTimestamp(now()), 6000.0, {'iPhone 6': 10});
```
```
describe table lab5.orders
```
#### responce:
```
CREATE TABLE lab5.orders (
    email text,
    date timestamp,
    map_items map<text, int>,
    name text,
    total float,
    PRIMARY KEY (email, date)
) WITH CLUSTERING ORDER BY (date ASC)
    AND bloom_filter_fp_chance = 0.01
    AND caching = {'keys': 'ALL', 'rows_per_partition': 'NONE'}
    AND comment = ''
    AND compaction = {'class': 'org.apache.cassandra.db.compaction.SizeTieredCompactionStrategy', 'max_threshold': '32', 'min_threshold': '4'}
    AND compression = {'chunk_length_in_kb': '64', 'class': 'org.apache.cassandra.io.compress.LZ4Compressor'}
    AND crc_check_chance = 1.0
    AND dclocal_read_repair_chance = 0.1
    AND default_time_to_live = 0
    AND gc_grace_seconds = 864000
    AND max_index_interval = 2048
    AND memtable_flush_period_in_ms = 0
    AND min_index_interval = 128
    AND read_repair_chance = 0.0
    AND speculative_retry = '99PERCENTILE';
```
### Для замовника виведіть всі його замовлення відсортовані за часом коли вони були зроблені
```
SELECT * FROM lab5.orders WHERE email = '!.gmail.com' ORDER BY date DESC;
```
#### responce:
```
 email       | date                            | map_items                              | name         | total
-------------+---------------------------------+----------------------------------------+--------------+-------
 !.gmail.com | 2018-03-03 21:04:06.416000+0000 |                       {'iPhone 6': 10} | Chuck Norris |  6000
 !.gmail.com | 2018-03-03 21:03:59.027000+0000 | {'Samsung CHG90 Series Curved 49': 10} | Chuck Norris | 10390

(2 rows)
```
### Для замовника знайдіть замовлення з певним товаром

```
SELECT * FROM lab5.orders WHERE email = '!.gmail.com' AND map_items CONTAINS KEY 'iPhone 6' ALLOW FILTERING;
```
#### responce:
```
 email       | date                            | map_items        | name         | total
-------------+---------------------------------+------------------+--------------+-------
 !.gmail.com | 2018-03-03 21:04:06.416000+0000 | {'iPhone 6': 10} | Chuck Norris |  6000

(1 rows)
```
### Для замовника знайдіть замовлення за певний період і їх кількість
```
SELECT * FROM lab5.orders WHERE email = '!.gmail.com' AND date >= '2018-03-03 00:00:00' AND date <= '2018-03-03 23:59:59';
```
#### responce:
```
 email       | date                            | map_items                              | name         | total
-------------+---------------------------------+----------------------------------------+--------------+-------
 !.gmail.com | 2018-03-03 21:03:59.027000+0000 | {'Samsung CHG90 Series Curved 49': 10} | Chuck Norris | 10390
 !.gmail.com | 2018-03-03 21:04:06.416000+0000 |                       {'iPhone 6': 10} | Chuck Norris |  6000

(2 rows)
```
```
SELECT count(*) FROM lab5.orders WHERE email = '!.gmail.com' AND date >= '2018-03-03 00:00:00' AND date <= '2018-03-03 23:59:59';
```
#### responce:
```
 count
-------
     2
```
### Для кожного замовників визначте середню вартість замовлення
```
SELECT email, name, avg(total) FROM lab5.orders GROUP BY email;
```
```
show VERSION
```
#### responce:
```
[cqlsh 5.0.1 | Cassandra 3.9.0 | CQL spec 3.4.2 | Native protocol v4]
```
```
SELECT email, name, avg(total) FROM lab5.orders WHERE email = '!.gmail.com';
```
#### responce:
```
 email       | name         | system.avg(total)
-------------+--------------+-------------------
 !.gmail.com | Chuck Norris |              8195

(1 rows)
```
```
SELECT email, name, avg(total) FROM lab5.orders WHERE email = 'jsmith.gmail.com';
```
#### responce:
```
 email            | name       | system.avg(total)
------------------+------------+-------------------
 jsmith.gmail.com | John Smith |            1269.5

(1 rows)
```
### Для кожного замовників визначте суму на яку були зроблені усі його замовлення
```
SELECT email, name, sum(total) FROM lab5.orders GROUP BY email;
```
```
SELECT email, name, sum(total) FROM lab5.orders WHERE email = 'jsmith.gmail.com';
```
#### responce:
```
 email            | name       | system.sum(total)
------------------+------------+-------------------
 jsmith.gmail.com | John Smith |              2539

(1 rows)
```
```
SELECT email, name, sum(total) FROM lab5.orders WHERE email = '!.gmail.com';
```
#### responce:
```
 email       | name         | system.sum(total)
-------------+--------------+-------------------
 !.gmail.com | Chuck Norris |             16390

(1 rows)
```
### Для кожного замовників визначте замовлення з максимальною вартістю
```
SELECT email, name, max(total) FROM lab5.orders GROUP BY email;
```
```
SELECT email, name, max(total) FROM lab5.orders WHERE email = '!.gmail.com';
```
#### responce:
```
 email       | name         | system.max(total)
-------------+--------------+-------------------
 !.gmail.com | Chuck Norris |             10390
```
### Модифікуйте певне замовлення додавши / видаливши один або кілька товарів при цьому також змінюючи вартість замовлення
```
UPDATE lab5.orders SET map_items = map_items + {'Redmi Note 4x': 1}, total = 6300 WHERE email = '!.gmail.com' AND date ='2018-03-03 23:04:06.416'
```

### Для кожного замовлення виведіть час коли його ціна били занесена в базу (SELECT WRITETIME)
```
SELECT email, date, WRITETIME(total) FROM orders;
```
#### responce:
```
 email            | date                            | writetime(total)
------------------+---------------------------------+------------------
 jsmith.gmail.com | 2018-03-03 20:59:57.555000+0000 | 1520110797547000
 jsmith.gmail.com | 2018-03-03 21:00:52.578000+0000 | 1520110852577000
      !.gmail.com | 2018-03-03 21:03:59.027000+0000 | 1520111039024000
      !.gmail.com | 2018-03-03 21:04:06.416000+0000 | 1520113917607000

(4 rows)
```
### Створіть замовлення з певним часом життя (TTL), після якого воно видалиться 
```
INSERT INTO lab5.orders (email, name, date, total, map_items) VALUES ('jsmith.gmail.com', 'John Smith', toTimestamp(now()), 999.0, {'Redmi Note 4x': 1}) USING TTL 120;
```
#### check
```
 email            | date                            | map_items                                            | name       | total
------------------+---------------------------------+------------------------------------------------------+------------+-------
 jsmith.gmail.com | 2018-03-03 20:59:57.555000+0000 | {'Samsung CHG90 Series Curved 49': 1, 'iPhone 6': 2} | John Smith |  2239
 jsmith.gmail.com | 2018-03-03 21:00:52.578000+0000 |                                 {'Redmi Note 4x': 1} | John Smith |   300
 jsmith.gmail.com | 2018-03-03 21:11:11.137000+0000 |                                 {'Redmi Note 4x': 1} | John Smith |   999

(3 rows)
```
#### check later
```
 email            | date                            | map_items                                            | name       | total
------------------+---------------------------------+------------------------------------------------------+------------+-------
 jsmith.gmail.com | 2018-03-03 20:59:57.555000+0000 | {'Samsung CHG90 Series Curved 49': 1, 'iPhone 6': 2} | John Smith |  2239
 jsmith.gmail.com | 2018-03-03 21:00:52.578000+0000 |                                 {'Redmi Note 4x': 1} | John Smith |   300

(2 rows)
```
### Поверніть замовлення у форматі JSON
```
SELECT json * FROM lab5.orders WHERE email = '!.gmail.com' AND date = '2018-03-03 23:04:06.416';
```
#### responce:
```
 [json]
----------------------------------------------------------------------------------------------------------------------------------------------------------
 {"email": "!.gmail.com", "date": "2018-03-03 21:04:06.416Z", "map_items": {"Redmi Note 4x": 1, "iPhone 6": 10}, "name": "Chuck Norris", "total": 6300.0}

(1 rows)
```
### Додайте замовлення у форматі JSON
```
INSERT INTO lab5.orders JSON '{
  "email" : "jsmith.gmail.com", 
  "name" : "John Smith", 
  "date" : "2018-03-03 21:20:37",
  "total" : 300.0,
  "map_items" : {"Redmi Note 4x": 1}
 }';
```
#### check:
```
SELECT json * FROM lab5.orders WHERE email = 'jsmith.gmail.com';
```
#### responce:
```
 [json]
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
                                  {"email": "jsmith.gmail.com", "date": "2018-03-03 19:20:37.000Z", "map_items": {"Redmi Note 4x": 1}, "name": "John Smith", "total": 300.0}
 {"email": "jsmith.gmail.com", "date": "2018-03-03 20:59:57.555Z", "map_items": {"Samsung CHG90 Series Curved 49": 1, "iPhone 6": 2}, "name": "John Smith", "total": 2239.0}
                                  {"email": "jsmith.gmail.com", "date": "2018-03-03 21:00:52.578Z", "map_items": {"Redmi Note 4x": 1}, "name": "John Smith", "total": 300.0}

(3 rows)
```