# Полезности для ElasticSearch и PostgreSQL


## Перенос данных postgre и elastic между серверами

### На машине, где уже все есть в postgre и elastic:
1. Делаем бэкап
```
pg_dump -U taskuser taskdb > backup.sql
```
2. Устанавливаем elasticdump
```
npm install elasticdump -g
```
3. Бэкап схем
```
elasticdump --input=http://127.0.0.1:9200/content --output=es_mapping.json --type=mapping
```
4. Бэкап данных
```
elasticdump --input=http://127.0.0.1:9200/content --output=es_index.json --type=data
```

### На машине, на которую все нужно перенести:
1. Удаляем старые данные и настраиваем заново taskdb
```
psql -U postgres -d taskdb -c "DROP SCHEMA public CASCADE;"
psql -U postgres -d taskdb -c "CREATE SCHEMA public;"
psql -U postgres -d taskdb -c "GRANT ALL ON SCHEMA public TO postgres; GRANT ALL ON SCHEMA public TO public;"
```
2. Восстанавливаем из бэкапа
```
psql -U postgres -d taskdb -f backup.sql
```
3. Останавливаем elastic, удаляем каталог с индексами (elasticpath/data/node/), запускаем elastic
4. Устанавливаем elasticdump
```
npm install elasticdump -g
```
5. Восстанваливаем схемы из файла бэкапа
```
elasticdump --input=es_mapping.json --output=http://127.0.0.1:9200/content --type=mapping
```
6. Восстанваливаем данные из файла бэкапа
```
elasticdump --input=es_index.json --output=http://127.0.0.1:9200/content --type=data
```

---

## Различные команды
1. Поиск по значению поля
```
GET /index_name/_search
{
  "query": {
    "multi_match" : {
      "query":    "search_string",
      "fields": [ "field_name" ]
    }
  }
}
```
2. Обновление поля
```
POST /index/type/id/_update
{
    "doc" : {
        "field_name" : new_value
    }
}
```
Пример:
```
POST /content/_doc/39653d8d-2191-469a-8d4a-bd4f19b71232/_update
{
  "doc" : {
    "type" : "SbisDocumentMdl"
  }
}
```
3. Удаление по полю
```
POST /index_name/_delete_by_query
{
    "query": {
    "multi_match" : {
      "query":    "search_string",
      "fields": [ "field_name" ]
    }
  }
}
```
Пример:
```
POST /content/_delete_by_query
{
  "query": {
    "multi_match" : {
      "query":    "ShowDocsMdl",
      "fields": [ "type" ]
    }
  }
}
```
3. Поиск по пустому полу (data:{})
```
GET /index_name/_search
{
  "query":{
    "bool":{
      "must_not":[
        {
          "exists":{
            "field":"tags"
          }
        }
      ]
    }
  }
}
```
