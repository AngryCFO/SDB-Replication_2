# Домашнее задание к занятию "Репликация и масштабирование. Часть 2" - `Александра Бужор`

---

### Задание 1

Задание 1
Опишите основные преимущества использования масштабирования методами:
- активный master-сервер и пассивный репликационный slave-сервер;
- master-сервер и несколько slave-серверов;
- активный сервер со специальным механизмом репликации — distributed replicated block device (DRBD);
- SAN-кластер.

Дайте ответ в свободной форме.

### Решение:

1. **Активный master-сервер и пассивный репликационный slave-сервер**: Это классический вариант для обеспечения высокой доступности. Основное преимущество здесь в том, что если основной сервер упадет, пассивный сервер может быстро взять на себя его функции. Это обеспечивает непрерывность работы сервиса без значительных потерь данных.

2. **Master-сервер и несколько slave-серверов**: Такая схема позволяет распределять нагрузку чтения между несколькими серверами, что существенно увеличивает производительность системы при чтении. Это особенно актуально для приложений, где операций чтения намного больше, чем записи.

3. **Активный сервер со специальным механизмом репликации — DRBD (Distributed Replicated Block Device)**: DRBD позволяет реплицировать данные на уровне блочных устройств в реальном времени между серверами. Это обеспечивает высокий уровень отказоустойчивости и доступности данных. Главное преимущество здесь в том, что DRBD может быть использован для любых типов данных и файловых систем, делая его универсальным решением для синхронной репликации.

4. **SAN-кластер (Storage Area Network)**: SAN обеспечивает высокоскоростной доступ к блочному хранилищу данных через сеть. Главные преимущества здесь в масштабируемости и гибкости управления данными. SAN позволяет множеству серверов доступ к общему пулу хранилища, что упрощает управление данными и резервное копирование, а также повышает общую производительность за счет распределения нагрузки.

Каждый из этих методов масштабирования имеет свои особенности и лучше всего подходит для определенных сценариев использования. Выбор зависит от конкретных требований к доступности, производительности и отказоустойчивости системы.

---

### Задание 2

Разработайте план для выполнения горизонтального и вертикального шаринга базы данных. База данных состоит из трёх таблиц:
- пользователи,
- книги,
- магазины (столбцы произвольно).
Опишите принципы построения системы и их разграничение или разбивку между базами данных.

Пришлите блоксхему, где и что будет располагаться. Опишите, в каких режимах будут работать сервера.

### Решение:

- **Горизонтальный шардинг (шардирование)** подразумевает разделение одной и той же таблицы на разные базы данных (или сервера), где каждая часть содержит уникальный набор данных. Это помогает распределить нагрузку и увеличить производительность при обработке больших объемов данных.

- **Вертикальный шардинг** подразумевает разделение таблицы или базы данных на части по столбцам. Каждая часть переносится в отдельную базу данных, что позволяет оптимизировать хранение и доступ к данным.

Для базы данных с таблицами **пользователи**, **книги**, и **магазины**, вот как может выглядеть план для горизонтального и вертикального шаринга:

### Горизонтальный шардинг:

```diff
+------------------+    +------------------+    +------------------+
|    User Shard 1  |    |    User Shard 2  |    |    User Shard N  |
+------------------+    +------------------+    +------------------+
| user_id (1-1000) |    | user_id (1001-2000)|  | user_id (N-N+1000)|
| name             |    | name                 |  | name             |
| rules            |    | rules                |  | rules            |
| password         |    | password             |  | password         |
| id_book          |    | id_book              |  | id_book          |
+------------------+    +------------------+    +------------------+

+------------------+    +------------------+    +------------------+
|   Books Shard 1  |    |   Books Shard 2  |    |   Books Shard N  |
+------------------+    +------------------+    +------------------+
| id_book (1-1000) |    | id_book (1001-2000)|  | id_book (N-N+1000)|
| name_books       |    | name_books           |  | name_books       |
| author           |    | author               |  | author           |
| year             |    | year                 |  | year             |
+------------------+    +------------------+    +------------------+

+-------------------+    +-------------------+    +-------------------+
|    Store Shard 1  |    |    Store Shard 2  |    |    Store Shard N  |
+-------------------+    +-------------------+    +-------------------+
| id_store (1-100)  |    | id_store (101-200)|    | id_store (N-N+100)|
| name_store        |    | name_store        |    | name_store        |
| location          |    | location          |    | location          |
| manager_id        |    | manager_id        |    | manager_id        |
+-------------------+    +-------------------+    +-------------------+
```


### Вертикальный шардинг:

```diff
+--------------+    +-----------------+    +-----------------+
| User Info    |    | User Security   |    | User Books      |
+--------------+    +-----------------+    +-----------------+
| user_id      |    | user_id         |    | user_id         |
| name         |    | password        |    | id_book         |
| rules        |    |                 |    |                 |
+--------------+    +-----------------+    +-----------------+

+--------------+    +-----------------+    +-----------------+
| Books Info   |    | Books Author    |    | Books Year      |
+--------------+    +-----------------+    +-----------------+
| id_book      |    | id_book         |    | id_book         |
| name_books   |    | author          |    | year            |
+--------------+    +-----------------+    +-----------------+

+-----------------+    +---------------------+    +-------------------+
| Store Info      |    | Store Location      |    | Store Management  |
+-----------------+    +---------------------+    +-------------------+
| id_store        |    | id_store            |    | id_store          |
| name_store      |    | location            |    | manager_id        |
+-----------------+    +---------------------+    +-------------------+
```

В блок-схемах выше:

- **Для горизонтального шардинга**, таблицы разделяются на шарды по определенным критериям. В нашем примере id, но возможно и по географическому признаку (если применим) или размеру магазина, что позволяет распределять данные и запросы между несколькими серверами для повышения производительности и масштабируемости.

- **Для вертикального шардинга**, каждая таблица разделяется на отдельные столбцы или группы столбцов, которые могут быть вынесены в отдельные базы данных. Это позволяет оптимизировать хранение данных и ускорить доступ к ним, так как запросы могут быть направлены непосредственно к нужной части данных.

Такая структура позволяет более гибко управлять нагрузкой и масштабировать систему по мере необходимости.

---

### Задание 3 *

Выполните настройку выбранных методов шардинга из задания 2.

Пришлите конфиг Docker и SQL скрипт с командами для базы данных.

### Решение:

### 1. Docker-compose конфигурация

```yaml
version: '3'

services:
  # Конфигурация для шарда пользователей 1
  users-shard1:
    image: mysql:8.0
    container_name: users-shard1
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: users_db
    volumes:
      - ./init-users-shard1.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3307:3306"
    networks:
      - shard-network

  # Конфигурация для шарда пользователей 2
  users-shard2:
    image: mysql:8.0
    container_name: users-shard2
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: users_db
    volumes:
      - ./init-users-shard2.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3308:3306"
    networks:
      - shard-network

  # Конфигурация для шарда книг
  books-shard:
    image: mysql:8.0
    container_name: books-shard
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: books_db
    volumes:
      - ./init-books.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3309:3306"
    networks:
      - shard-network

  # Конфигурация для шарда магазинов
  stores-shard:
    image: mysql:8.0
    container_name: stores-shard
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: stores_db
    volumes:
      - ./init-stores.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3310:3306"
    networks:
      - shard-network

  # Прокси для маршрутизации запросов
  proxy:
    image: haproxy:2.3
    container_name: mysql-proxy
    volumes:
      - ./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:ro
    ports:
      - "3306:3306"
    networks:
      - shard-network
    depends_on:
      - users-shard1
      - users-shard2
      - books-shard
      - stores-shard

networks:
  shard-network:
    driver: bridge
```

### 2. SQL скрипты инициализации

#### init-users-shard1.sql
```sql
CREATE DATABASE IF NOT EXISTS users_db;
USE users_db;

CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    registration_date DATE,
    CONSTRAINT user_id_range CHECK (user_id BETWEEN 1 AND 500000)
);

CREATE TABLE user_security (
    user_id INT PRIMARY KEY,
    password_hash VARCHAR(256),
    last_login DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Процедура для проверки диапазона id при вставке
DELIMITER //
CREATE PROCEDURE insert_user(
    IN p_user_id INT,
    IN p_name VARCHAR(100),
    IN p_email VARCHAR(100),
    IN p_registration_date DATE,
    IN p_password_hash VARCHAR(256)
)
BEGIN
    IF p_user_id BETWEEN 1 AND 500000 THEN
        START TRANSACTION;
        INSERT INTO users VALUES (p_user_id, p_name, p_email, p_registration_date);
        INSERT INTO user_security VALUES (p_user_id, p_password_hash, NOW());
        COMMIT;
    ELSE
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'User ID out of range for this shard';
    END IF;
END //
DELIMITER ;
```

#### init-users-shard2.sql
```sql
CREATE DATABASE IF NOT EXISTS users_db;
USE users_db;

CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    registration_date DATE,
    CONSTRAINT user_id_range CHECK (user_id > 500000)
);

CREATE TABLE user_security (
    user_id INT PRIMARY KEY,
    password_hash VARCHAR(256),
    last_login DATETIME,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Процедура для проверки диапазона id при вставке
DELIMITER //
CREATE PROCEDURE insert_user(
    IN p_user_id INT,
    IN p_name VARCHAR(100),
    IN p_email VARCHAR(100),
    IN p_registration_date DATE,
    IN p_password_hash VARCHAR(256)
)
BEGIN
    IF p_user_id > 500000 THEN
        START TRANSACTION;
        INSERT INTO users VALUES (p_user_id, p_name, p_email, p_registration_date);
        INSERT INTO user_security VALUES (p_user_id, p_password_hash, NOW());
        COMMIT;
    ELSE
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'User ID out of range for this shard';
    END IF;
END //
DELIMITER ;
```

#### init-books.sql
```sql
CREATE DATABASE IF NOT EXISTS books_db;
USE books_db;

CREATE TABLE books (
    book_id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(200),
    author VARCHAR(100),
    isbn VARCHAR(13),
    publication_year INT
);

CREATE TABLE book_details (
    book_id INT PRIMARY KEY,
    description TEXT,
    category VARCHAR(50),
    price DECIMAL(10,2),
    FOREIGN KEY (book_id) REFERENCES books(book_id)
);

-- Индексы для оптимизации поиска
CREATE INDEX idx_isbn ON books(isbn);
CREATE INDEX idx_author ON books(author);
CREATE INDEX idx_category ON book_details(category);
```

#### init-stores.sql
```sql
CREATE DATABASE IF NOT EXISTS stores_db;
USE stores_db;

CREATE TABLE stores (
    store_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    location VARCHAR(200),
    phone VARCHAR(20)
);

CREATE TABLE store_inventory (
    store_id INT,
    book_id INT,
    quantity INT,
    last_updated TIMESTAMP,
    PRIMARY KEY (store_id, book_id),
    FOREIGN KEY (store_id) REFERENCES stores(store_id)
);

-- Индексы для оптимизации
CREATE INDEX idx_location ON stores(location);
CREATE INDEX idx_inventory_book ON store_inventory(book_id);
```

### 3. Конфигурация HAProxy

```conf
global
    daemon
    maxconn 256

defaults
    mode tcp
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend mysql-cluster
    bind *:3306
    default_backend mysql-backends

backend mysql-backends
    server users-shard1 users-shard1:3306 check
    server users-shard2 users-shard2:3306 check
    server books-shard books-shard:3306 check
    server stores-shard stores-shard:3306 check
```

### 4. Инструкция по запуску

1. Создайте директорию проекта и поместите все файлы в нее:
```bash
mkdir mysql-sharding
cd mysql-sharding
```

2. Создайте все необходимые конфигурационные файлы:
- docker-compose.yml
- haproxy.cfg
- init-users-shard1.sql
- init-users-shard2.sql
- init-books.sql
- init-stores.sql

3. Запустите контейнеры:
```bash
docker-compose up -d
```

4. Проверьте статус контейнеров:
```bash
docker-compose ps
```

5. Проверьте подключение к любому из шардов:
```bash
mysql -h 127.0.0.1 -P 3307 -u root -p
```

### 5. Примеры использования

#### Вставка данных в шард пользователей:
```sql
-- Для первого шарда (user_id <= 500000)
CALL insert_user(1, 'John Doe', 'john@example.com', '2024-01-01', 'hash123');

-- Для второго шарда (user_id > 500000)
CALL insert_user(500001, 'Jane Doe', 'jane@example.com', '2024-01-02', 'hash456');
```

#### Добавление книги:
```sql
INSERT INTO books (title, author, isbn, publication_year)
VALUES ('Sample Book', 'Sample Author', '1234567890123', 2024);
```

#### Добавление магазина:
```sql
INSERT INTO stores (name, location, phone)
VALUES ('Book Store 1', 'New York', '+1234567890');
```
---
