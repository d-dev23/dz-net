## Задача 1

Используя docker поднимите инстанс MySQL (версию 8). Данные БД сохраните в volume.

Изучите [бэкап БД](https://github.com/netology-code/virt-homeworks/tree/virt-11/06-db-03-mysql/test_data) и 
восстановитесь из него.

Перейдите в управляющую консоль `mysql` внутри контейнера.

Используя команду `\h` получите список управляющих команд.

Найдите команду для выдачи статуса БД и **приведите в ответе** из ее вывода версию сервера БД.

Подключитесь к восстановленной БД и получите список таблиц из этой БД.

**Приведите в ответе** количество записей с `price` > 300.

В следующих заданиях мы будем продолжать работу с данным контейнером.

Ответы:
1.

    mysql> status
    --------------
    mysql  Ver 8.0.23 for Linux on x86_64 (MySQL Community Server - GPL)
    
    Connection id:          19
    Current database:
    Current user:           root@localhost
    SSL:                    Not in use
    Current pager:          stdout
    Using outfile:          ''
    Using delimiter:        ;
    Server version:         8.0.23 MySQL Community Server - GPL
    Protocol version:       10
    Connection:             Localhost via UNIX socket
    Server characterset:    utf8mb4
    Db     characterset:    utf8mb4
    Client characterset:    latin1
    Conn.  characterset:    latin1
    UNIX socket:            /var/run/mysqld/mysqld.sock
    Binary data as:         Hexadecimal
    Uptime:                 13 min 51 sec
    
    Threads: 2  Questions: 106  Slow queries: 0  Opens: 137  Flush tables: 3  Open tables: 55  Queries per second avg: 0.127
    --------------
2.

    mysql> SELECT COUNT(*) FROM orders WHERE price > 300;
    +----------+
    | COUNT(*) |
    +----------+
    |        1 |
    +----------+
    1 row in set (0.01 sec)

## Задача 2

Создайте пользователя test в БД c паролем test-pass, используя:
- плагин авторизации mysql_native_password
- срок истечения пароля - 180 дней 
- количество попыток авторизации - 3 
- максимальное количество запросов в час - 100
- аттрибуты пользователя:
    - Фамилия "Pretty"
    - Имя "James"

Предоставьте привелегии пользователю `test` на операции SELECT базы `test_db`.
    
Используя таблицу INFORMATION_SCHEMA.USER_ATTRIBUTES получите данные по пользователю `test` и 
**приведите в ответе к задаче**.

Ответы:

    mysql> SELECT * FROM INFORMATION_SCHEMA.USER_ATTRIBUTES WHERE USER = 'test' AND HOST = 'localhost';
    +------+-----------+---------------------------------------+
    | USER | HOST      | ATTRIBUTE                             |
    +------+-----------+---------------------------------------+
    | test | localhost | {"fname": "James", "lname": "Pretty"} |
    +------+-----------+---------------------------------------+
    1 row in set (0.00 sec)


## Задача 3

Установите профилирование `SET profiling = 1`.
Изучите вывод профилирования команд `SHOW PROFILES;`.

Исследуйте, какой `engine` используется в таблице БД `test_db` и **приведите в ответе**.

Измените `engine` и **приведите время выполнения и запрос на изменения из профайлера в ответе**:
- на `MyISAM`
- на `InnoDB`

Ответы:
1.

    mysql> SHOW TABLE STATUS\G
    *************************** 1. row ***************************
               Name: orders
             Engine: InnoDB
            Version: 10
         Row_format: Dynamic
               Rows: 5
     Avg_row_length: 3276
        Data_length: 16384
    Max_data_length: 0
       Index_length: 0
          Data_free: 0
     Auto_increment: 6
        Create_time: 2021-02-06 12:19:35
        Update_time: 2021-02-06 12:19:35
         Check_time: NULL
          Collation: utf8mb4_0900_ai_ci
           Checksum: NULL
     Create_options:
            Comment:
    1 row in set (0.02 sec)

2.

    SET profiling = 1;
    ALTER TABLE test_db.orders ENGINE = MyISAM;
    ALTER TABLE test_db.orders ENGINE = InnoDB;
    
    mysql> SHOW PROFILES;
    +----------+------------+--------------------------------------------+
    | Query_ID | Duration   | Query                                      |
    +----------+------------+--------------------------------------------+
    |        1 | 0.08565350 | ALTER TABLE test_db.orders ENGINE = MyISAM |
    |        2 | 0.07302175 | ALTER TABLE test_db.orders ENGINE = InnoDB |
    +----------+------------+--------------------------------------------+
    2 rows in set, 1 warning (0.00 sec)

## Задача 4 

Изучите файл `my.cnf` в директории /etc/mysql.

Измените его согласно ТЗ (движок InnoDB):
- Скорость IO важнее сохранности данных
- Нужна компрессия таблиц для экономии места на диске
- Размер буффера с незакомиченными транзакциями 1 Мб
- Буффер кеширования 30% от ОЗУ
- Размер файла логов операций 100 Мб

Приведите в ответе измененный файл `my.cnf`.

Ответы:

    # Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
    #
    # This program is free software; you can redistribute it and/or modify
    # it under the terms of the GNU General Public License as published by
    # the Free Software Foundation; version 2 of the License.
    #
    # This program is distributed in the hope that it will be useful,
    # but WITHOUT ANY WARRANTY; without even the implied warranty of
    # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    # GNU General Public License for more details.
    #
    # You should have received a copy of the GNU General Public License
    # along with this program; if not, write to the Free Software
    # Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
    
    #
    # The MySQL  Server configuration file.
    #
    # For explanations see
    # http://dev.mysql.com/doc/mysql/en/server-system-variables.html
    
    [mysqld]
    pid-file        = /var/run/mysqld/mysqld.pid
    socket          = /var/run/mysqld/mysqld.sock
    datadir         = /var/lib/mysql
    secure-file-priv= NULL
    
    innodb_flush_log_at_trx_commit = 2
    innodb_file_per_table = ON
    innodb_log_buffer_size = 1M
    innodb_buffer_pool_size = 30
    innodb_log_file_size = 100M
    
    # Custom config should go here
    !includedir /etc/mysql/conf.d/
