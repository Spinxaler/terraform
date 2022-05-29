> Задача 1

- текст Dockerfile манифеста
```dockerfile
FROM centos:7
COPY ./src /var/src/
RUN cd /var/src/ && \
tar -xzf /var/src/elasticsearch-8.1.1-linux-x86_64.tar.gz && \
rm /var/src/elasticsearch-8.1.1-linux-x86_64.tar.gz && \
cp elasticsearch.yml /var/src/elasticsearch-8.1.1/config/ && \
adduser elastic && \
gpasswd -a elastic wheel && \
chown -R elastic:elastic /var/src/elasticsearch-8.1.1 && \
chmod 755 /var/src/elasticsearch-8.1.1 && \
chmod 777 /var/lib
USER elastic:wheel
EXPOSE 9200 9300
CMD ["var/src/elasticsearch-8.1.1/bin/elasticsearch"]
```
```bash
$ docker build .
$ docker login -u "spinxaler" -p "****" docker.io
$ docker tag 2f99f8d83166 spinxaler/devops-elasticsearch:8.1.1
$ docker push spinxaler/devops-elasticsearch:8.1.1
```
- ссылку на образ в репозитории dockerhub
https://hub.docker.com/repository/docker/spinxaler/devops-elasticsearch
- ответ `elasticsearch` на запрос пути `/` в json виде
```bash
$ docker run --rm -d --name es -p 9200:9200 -p 9300:9300 spinxaler/devops-elasticsearch:8.1.1
$ docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                                                                                  NAMES
5d0d0f7f91d4   spinxaler/devops-elasticsearch:8.1.1   "sh -c ${ES_HOME}/bi…"   43 seconds ago   Up 42 seconds   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp, 0.0.0.0:9300->9300/tcp, :::9300->9300/tcp   es

$ curl http://localhost:9200/
```
```json
{
  "name" : "netology_test",
  "cluster_name" : "netology_test_cluster",
  "cluster_uuid" : "CmnlpVLTRkey47ycIsplpA",
  "version" : {
    "number" : "8.1.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "d0925dd6f22e07b935750420a3155db6e5c58381",
    "build_date" : "2022-03-17T22:01:32.658689558Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

>Задача 2

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `elasticsearch` 3 индекса, в соответствии со таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

```bash
$ curl -X PUT "localhost:9200/ind-1?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
'
```
```bash
$ curl -X PUT "localhost:9200/ind-2?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  }
}
'
```
```bash
$ curl -X PUT "localhost:9200/ind-3?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 4,
    "number_of_replicas": 2
  }
}
'
```
Получите список индексов и их статусов, используя API и **приведите в ответе** на задание.

```bash
$ curl 'localhost:9200/_cat/indices?v'
health status index           uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   ind-1           LaPC2n6wRYu9o-11PsI0UQ   1   0          0            0       226b           226b
yellow open   ind-3           YrxEBcypQxKJmaJtBMjEQg   4   2          0            0       604b           604b
yellow open   my-index-000001 BykGdfA7RfSrdR5FpQcc3w   1   1          0            0       226b           226b
yellow open   ind-2           2uKFogwcQ76mup_L7cociA   2   1          0            0       452b           452b
```

Получите состояние кластера `elasticsearch`, используя API.

```bash
$ curl -X GET "localhost:9200/_cluster/health?pretty"
{
  "cluster_name" : "netology_test_cluster",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 10,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 50.0
}
```
Как вы думаете, почему часть индексов и кластер находится в состоянии yellow?

```
Первичный шард и реплика не могут находиться на одном узле, если копия не назначена. Таким образом, один узел не может размещать копии
```
Удалите все индексы.

```bash
$ curl -X DELETE 'http://localhost:9200/_all'
```

>Задача 3

Создайте директорию `{путь до корневой директории с elasticsearch в образе}/snapshots`.
```bash
$ docker exec -u root -it es bash
[root@5d0d0f7f91d4  elasticsearch]# mkdir $ES_HOME/snapshots
```

Используя API [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
данную директорию как `snapshot repository` c именем `netology_backup`.

```bash
bash
# echo path.repo: [ "/var/lib/elasticsearch/snapshots" ] >> "$ES_HOME/config/elasticsearch.yml"
# chown elasticsearch:elasticsearch /var/lib/elasticsearch/snapshots
$ docker restart es
$ curl -X PUT "localhost:9200/_snapshot/netology_backup?pretty" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/var/lib/elasticsearch/snapshots",
    "compress": true
  }
}'
{"acknowledged":true
```

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

```bash
$ curl -X PUT "localhost:9200/test?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
'
$ curl 'localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   my-index-000001  BykGdfA7RfSrdR5FpQcc3w   1   1          0            0       226b           226b
green  open   test             JuKFogwcQeZiwirdSjt3   1   0          0            0       226b           226b
```


[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `elasticsearch`.

```bash
bash
$ curl -X PUT "localhost:9200/_snapshot/netology_backup/snapshot_1?wait_for_completion=true&pretty"
```

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

```bash
ash
$ curl -X DELETE "localhost:9200/test?pretty"
$ curl -X PUT "localhost:9200/test-2?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  }
}
'
$ curl 'localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   my-index-000001  BykGdfA7RfSrdR5FpQcc3w   1   1          0            0       226b           226b
green  open   test-2           Xw_SdQXLQZuWJ8xPsI0UQ5   1   0          0            0       226b           226b
```

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.
```bash
curl -X POST "localhost:9200/_snapshot/netology_backup/snapshot_1/_restore?pretty" -H 'Content-Type: application/json' -d'
{
  "indices": "*",
  "include_global_state": true
}
'
```
```bash
$ curl 'localhost:9200/_cat/indices?v'
health status index            uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   my-index-000001  BykGdfA7RfSrdR5FpQcc3w   1   1          0            0       226b           226b
green  open   test             MGdf0vHRoudR5AcAdfA7yr   1   0          0            0       226b           226b
```
