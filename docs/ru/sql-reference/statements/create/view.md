---
toc_priority: 3
toc_title: Представление
---

# CREATE VIEW {#create-view}

``` sql
CREATE [MATERIALIZED] VIEW [IF NOT EXISTS] [db.]table_name [TO[db.]name] [ENGINE = engine] [POPULATE] AS SELECT ...
```

Создаёт представление. Представления бывают двух видов - обычные и материализованные (MATERIALIZED).

Обычные представления не хранят никаких данных, а всего лишь производят чтение из другой таблицы. То есть, обычное представление - не более чем сохранённый запрос. При чтении из представления, этот сохранённый запрос, используется в качестве подзапроса в секции FROM.

Для примера, пусть вы создали представление:

``` sql
CREATE VIEW view AS SELECT ...
```

и написали запрос:

``` sql
SELECT a, b, c FROM view
```

Этот запрос полностью эквивалентен использованию подзапроса:

``` sql
SELECT a, b, c FROM (SELECT ...)
```

Материализованные (MATERIALIZED) представления хранят данные, преобразованные соответствующим запросом SELECT.

При создании материализованного представления без использования `TO [db].[table]`, нужно обязательно указать ENGINE - движок таблицы для хранения данных.

При создании материализованного представления с испольованием `TO [db].[table]`, нельзя указывать `POPULATE`

Материализованное представление устроено следующим образом: при вставке данных в таблицу, указанную в SELECT-е, кусок вставляемых данных преобразуется этим запросом SELECT, и полученный результат вставляется в представление.

Если указано POPULATE, то при создании представления, в него будут вставлены имеющиеся данные таблицы, как если бы был сделан запрос `CREATE TABLE ... AS SELECT ...` . Иначе, представление будет содержать только данные, вставляемые в таблицу после создания представления. Не рекомендуется использовать POPULATE, так как вставляемые в таблицу данные во время создания представления, не попадут в него.

Запрос `SELECT` может содержать `DISTINCT`, `GROUP BY`, `ORDER BY`, `LIMIT`… Следует иметь ввиду, что соответствующие преобразования будут выполняться независимо, на каждый блок вставляемых данных. Например, при наличии `GROUP BY`, данные будут агрегироваться при вставке, но только в рамках одной пачки вставляемых данных. Далее, данные не будут доагрегированы. Исключение - использование ENGINE, производящего агрегацию данных самостоятельно, например, `SummingMergeTree`.

Недоработано выполнение запросов `ALTER` над материализованными представлениями, поэтому они могут быть неудобными для использования. Если материализованное представление использует конструкцию `TO [db.]name`, то можно выполнить `DETACH` представления, `ALTER` для целевой таблицы и последующий `ATTACH` ранее отсоединенного (`DETACH`) представления.

Представления выглядят так же, как обычные таблицы. Например, они перечисляются в результате запроса `SHOW TABLES`.

Отсутствует отдельный запрос для удаления представлений. Чтобы удалить представление, следует использовать `DROP TABLE`.

[Оригинальная статья](https://clickhouse.tech/docs/ru/sql-reference/statements/create/view) 
<!--hide-->