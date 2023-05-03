# Домашнее задание к занятию "5. Elasticsearch"

## Задача 1

В этом задании вы потренируетесь в:
- установке elasticsearch
- первоначальном конфигурировании elastcisearch
- запуске elasticsearch в docker

Используя докер образ [centos:7](https://hub.docker.com/_/centos) как базовый и.
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для elasticsearch
- соберите docker-образ и сделайте `push` в ваш docker.io репозиторий
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины

Требования к `elasticsearch.yml`:
- данные `path` должны сохраняться в `/var/lib`
- имя ноды должно быть `netology_test`

В ответе приведите:
- текст Dockerfile манифеста
- ссылку на образ в репозитории dockerhub
- ответ `elasticsearch` на запрос пути `/` в json виде

Подсказки:
- возможно вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml
- при некоторых проблемах вам поможет docker директива ulimit
- elasticsearch в логах обычно описывает проблему и пути ее решения

Далее мы будем работать с данным экземпляром elasticsearch.

Ответы:

    FROM centos:7
    RUN yum install wget perl-Digest-SHA -y
    RUN groupadd -r elasticsearch && \
        useradd -r -g  elasticsearch elasticsearch
    USER elasticsearch
    WORKDIR /home/elasticsearch
    RUN wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz  && \
        wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.11.1-linux-x86_64.tar.gz.sha512 && \
        shasum -a 512 -c elasticsearch-7.11.1-linux-x86_64.tar.gz.sha512 && \
        tar -xzf elasticsearch-7.11.1-linux-x86_64.tar.gz
    COPY ./elasticsearch.yml ./elasticsearch-7.11.1/config
    EXPOSE 9200
    ENTRYPOINT ./elasticsearch-7.11.1/bin/elasticsearch


[Docker Hub](https://hub.docker.com/r/frolmr/6_5_elastic/tags)


    curl -X GET "localhost:9200/?pretty"
    {
      "name" : "netology_test",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "hddU2vG_Quy4fGrjqOpMPg",
      "version" : {
        "number" : "7.11.1",
        "build_flavor" : "default",
        "build_type" : "tar",
        "build_hash" : "ff17057114c2199c9c1bbecc727003a907c0db7a",
        "build_date" : "2021-02-15T13:44:09.394032Z",
        "build_snapshot" : false,
        "lucene_version" : "8.7.0",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }

## Задача 2

В этом задании вы научитесь:
- создавать и удалять индексы
- изучать состояние кластера
- обосновывать причину деградации доступности данных

Ознакомтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html).
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

Получите состояние кластера `elasticsearch`, используя API.

Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

Удалите все индексы.

**Важно**

При проектировании кластера elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

Ответы:
Dockerfile

    curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
    {
      "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
      }
    }
    '
    curl -X PUT "localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
    {
      "settings": {
        "number_of_shards": 2,
        "number_of_replicas": 1
      }
    }
    '
    curl -X PUT "localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
    {
      "settings": {
        "number_of_shards": 4,
        "number_of_replicas": 2
      }
    }
    '


Список индексов

    curl -X GET -u undefined:$ESPASS "localhost:9200/_cat/indices/ind-*?v=true&s=index&pretty"
    health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   ind-1 esoOQBzkSB62VBrWXJvE9w   1   0          0            0       208b           208b
    yellow open   ind-2 tdsm3LIKRo2BWfz11sYrLg   2   1          0            0       416b           416b
    yellow open   ind-3 HcKdUeeNRo-1oRchrmebGA   4   2          0            0       832b           832b


Состояние

    curl -X GET -u undefined:$ESPASS "localhost:9200/_cluster/health?pretty"
    {
      "cluster_name" : "elasticsearch",
      "status" : "yellow",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 7,
      "active_shards" : 7,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 10,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
     "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 41.17647058823529
    }
## Задача 3

В данном задании вы научитесь:
- создавать бэкапы данных
- восстанавливать индексы из бэкапов

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository).
данную директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html).
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html).
состояния кластера `elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`ами.

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее..

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

Подсказки:
- возможно вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `elasticsearch`

Ответы:
Используя API зарегистрируйте данную директорию как snapshot repository c именем netology_backup


    curl -X PUT -u undefined:$ESPASS "localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
    {
      "type": "fs",
      "settings": {
        "location": "snapshots"
      }
    }
    '
    {
      "acknowledged" : true
    }


Создайте индекс test с 0 реплик и 1 шардом


    curl -X GET -u undefined:$ESPASS "localhost:9200/_cat/indices/*?v=true&s=index&pretty"
    health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   test  wtfdytcJROyQPoiK1ir89w   1   0          0            0       208b           208b


Создайте snapshot состояния кластера elasticsearch

    ls -l elasticsearch-7.11.1/snapshots/
    total 48
    -rw-r--r-- 1 elasticsearch elasticsearch   434 Feb 18 20:31 index-0
    -rw-r--r-- 1 elasticsearch elasticsearch     8 Feb 18 20:31 index.latest
    drwxr-xr-x 3 elasticsearch elasticsearch  4096 Feb 18 20:31 indices
    -rw-r--r-- 1 elasticsearch elasticsearch 30964 Feb 18 20:31 meta-pPb7QEQtSCGhv9-FIKQfUw.dat
    -rw-r--r-- 1 elasticsearch elasticsearch   266 Feb 18 20:31 snap-pPb7QEQtSCGhv9-FIKQfUw.dat


Удалите индекс test и создайте индекс test-2


    curl -X GET -u undefined:$ESPASS "localhost:9200/_cat/indices/*?v=true&s=index&pretty"
    health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   test-2 8swc76KiSiCK_OfgOP0h1w   1   0          0            0       208b           208b


Восстановите состояние кластера elasticsearch из snapshot, созданного ранее


    curl -X POST -u undefined:$ESPASS "localhost:9200/_snapshot/netology_backup/snapshot_1/_restore?pretty"

    curl -X GET -u undefined:$ESPASS "localhost:9200/_cat/indices/*?v=true&s=index&pretty"
    health status index  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   test   kY6Fs7hOSL2mKoKiqOR3WA   1   0          0            0       208b           208b
    green  open   test-2 8swc76KiSiCK_OfgOP0h1w   1   0          0            0       208b           208b
