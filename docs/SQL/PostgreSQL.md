# PostgreSQL. Настройка

!!! Статьи
    - [ ]  [Простое обнаружение проблем производительности в PostgreSQL](https://infostart.ru/1c/articles/1198118/)

## pg_stat_statements
Включение pg_stat_statements

Чтобы включить pg_stat_statements на вашем сервере, измените следующую строку в postgresql.conf и перезапустите PostgreSQL:

``` conf
shared_preload_libraries = 'pg_stat_statements'
```

После загрузки этого модуля на сервер PostgreSQL автоматически начнет собирать информацию. Хорошо то, что накладные расходы модуля действительно очень низкие (накладные расходы в основном просто «шум»).

Затем выполните следующую команду для создания представления, необходимого для доступа к данным:

``` sql
CREATE EXTENSION pg_stat_statements;
```

Расширение создаст представление с именем pg_stat_statements и сделает данные легко доступными.

Самый простой способ найти наиболее интересные запросы — отсортировать вывод pg_stat_statements по total_time:

	
``` sql 
SELECT * FROM pg_stat_statements ORDER BY total_time DESC;
```

Прелесть здесь в том, что тип запроса, который наиболее трудоемкий, естественно будет отображаться в верхней части списка. Лучший способ — пройтись от первого до, скажем, 10-го запроса и посмотреть, что там происходит.

### Углубленный анализ производительности PostgreSQL

pg_stat_statements может предложить гораздо больше, чем просто запрос и время, которое он занял. Вот структура представления:

``` sql 	
test=# \d pg_stat_statements
View "public.pg_stat_statements"
Column               |     Type         | Collation | Nullable | Default
---------------------+------------------+-----------+----------+---------
 userid              | oid              |           |          |
 dbid                | oid              |           |          |
 queryid             | bigint           |           |          |
 query               | text             |           |          |
 calls               | bigint           |           |          |
 total_time          | double precision |           |          |
 min_time            | double precision |           |          |
 max_time            | double precision |           |          |
 mean_time           | double precision |           |          |
 stddev_time         | double precision |           |          |
 rows                | bigint           |           |          |
 shared_blks_hit     | bigint           |           |          |
 shared_blks_read    | bigint           |           |          |
 shared_blks_dirtied | bigint           |           |          |
 shared_blks_written | bigint           |           |          |
 local_blks_hit      | bigint           |           |          |
 local_blks_read     | bigint           |           |          |
 local_blks_dirtied  | bigint           |           |          |
 local_blks_written  | bigint           |           |          |
 temp_blks_read      | bigint           |           |          |
 temp_blks_written   | bigint           |           |          |
 blk_read_time       | double precision |           |          |
 blk_write_time      | double precision |           |          |

``` 

Вполне полезно посмотреть и на столбец **stddev_time**. Если стандартное отклонение велико, можно ожидать, что некоторые из этих запросов будут быстрыми, а некоторые — медленными, что может привести к ухудшению работы пользователей.

Столбец **«rows»** также может быть достаточно информативным. Предположим, что 1000 вызовов вернули 1.000.000.000 строк: фактически это означает, что каждый вызов в среднем возвращал 1 миллион строк. Легко понять, что возвращать столько данных все время не слишком хорошо.

Если требуется проверить, показывает ли определенный тип запроса плохую производительность при кэшировании, будет интересен shared_ *. 
Вкратце: PostgreSQL может сообщить вам частоту обращений к кешу для каждого отдельного типа запроса в случае, если включен **pg_stat_statements**.

Также имеет смысл взглянуть на поля **temp_blks_** *. 
Каждый раз, когда PostgreSQL должен обратиться к диску для сортировки или материализации, потребуются временные блоки.

Наконец-то есть **blk_read_time** и **blk_write_time**. 
Обычно эти поля пусты, если не включен track_io_timing. Идея здесь заключается в том, чтобы иметь возможность измерять количество времени, которое определенный тип запроса тратит на ввод-вывод. Это позволит вам ответить на вопрос, привязана ли ваша система к вводу/выводу или к ЦП. В большинстве случаев рекомендуется включить I/O timing, поскольку это даст вам важную информацию.


### Работа с Java и Hibernate

**pg_stat_statements** дает хорошую информацию. Однако в некоторых случаях запрос может быть прерван из-за переменной конфигурации:

``` sql 
test=# SHOW track_activity_query_size;
track_activity_query_size
---------------------------
1024
(1 row)
``` 

Для большинства приложений 1024 байта абсолютно достаточно. Однако обычно это не тот случай, если вы используете Hibernate или Java. Hibernate имеет тенденцию посылать безумно длинные запросы к базе данных, и поэтому код SQL может быть обрезан задолго до запуска соответствующих частей (например, предложение FROM и т. Д.). Поэтому имеет смысл увеличить **track_activity_query_size** до более высокого значения (возможно 32.786).


### Полезные запросы для выявления узких мест в PostgreSQL

Есть один запрос, который я нашел особенно полезным в этом контексте: Следующий запрос показывает 20 запросов, занимающих много времени:

``` sql 

SELECT substring(query, 1, 50) AS short_query,
              round(total_time::numeric, 2) AS total_time,
              calls,
              round(mean_time::numeric, 2) AS mean,
              round((100 * total_time / sum(total_time::numeric) OVER ())::numeric, 2) AS percentage_cpu
FROM  pg_stat_statements
ORDER BY total_time DESC
LIMIT 20;

```

``` sql 
             short_query                              | total_time | calls | mean | percentage_cpu
----------------------------------------------------+------------+-------+------+----------------
SELECT name FROM (SELECT pg_catalog.lower(name) A   | 11.85      | 7     | 1.69 | 38.63
DROP SCHEMA IF EXISTS performance_check CASCADE;    | 4.49       | 4     | 1.12 | 14.64
CREATE OR REPLACE FUNCTION performance_check.pg_st  | 2.23       | 4     | 0.56 | 7.27
SELECT pg_catalog.quote_ident(c.relname) FROM pg_c  | 1.78       | 2     | 0.89 | 5.81
SELECT a.attname, +                                 | 1.28       | 1     | 1.28 | 4.18
SELECT substring(query, ?, ?) AS short_query,roun   | 1.18       | 3     | 0.39 | 3.86
CREATE OR REPLACE FUNCTION performance_check.pg_st  | 1.17       | 4     | 0.29 | 3.81
SELECT query FROM pg_stat_activity LIMIT ?;         | 1.17       | 2     | 0.59 | 3.82
CREATE SCHEMA performance_check;                    | 1.01       | 4     | 0.25 | 3.30
SELECT pg_catalog.quote_ident(c.relname) FROM pg_c  | 0.92       | 2     | 0.46 | 3.00
SELECT query FROM performance_check.pg_stat_activi  | 0.74       | 1     | 0.74 | 2.43
SELECT * FROM pg_stat_statements ORDER BY total_ti  | 0.56       | 1     | 0.56 | 1.82
SELECT query FROM pg_stat_statements LIMIT ?;       | 0.45       | 4     | 0.11 | 1.45
GRANT EXECUTE ON FUNCTION performance_check.pg_sta  | 0.35       | 4     | 0.09 | 1.13
SELECT query FROM performance_check.pg_stat_statem  | 0.30       | 1     | 0.30 | 0.96
SELECT query FROM performance_check.pg_stat_activi  | 0.22       | 1     | 0.22 | 0.72
GRANT ALL ON SCHEMA performance_check TO schoenig_  | 0.20       | 3     | 0.07 | 0.66
SELECT query FROM performance_check.pg_stat_statem  | 0.20       | 1     | 0.20 | 0.67
GRANT EXECUTE ON FUNCTION performance_check.pg_sta  | 0.19       | 4     | 0.05 | 0.62
SELECT query FROM performance_check.pg_stat_statem  | 0.17       | 1     | 0.17 | 0.56
(20 rows)

``` 

Последний столбец особенно примечателен: он показывает процент общего времени, потраченного на один запрос. Это поможет вам выяснить, насколько влияет запрос на общую производительность или не влияет.



!!! info "Источники"
    - [x] [Настройка PostgreSQL 11.5 и 1C: Предприятие 8.3.16 на Windows Server 2008R2](https://infostart.ru/1c/articles/1180438/)
    - [x] [infostart. Немного о конфигурировании PostgreSQL](https://infostart.ru/1c/articles/325482/)
    - [x] [infostart. Держи данные в тепле, транзакции в холоде, а VACUUM в голоде](https://infostart.ru/1c/articles/1191667/)
    - [x] [ИТС. Особенности использования PostgreSQL](https://its.1c.ru/db/metod8dev#browse:13:-1:1981:1987)
    - [x] [ИТС. Настройки PostgreSQL для работы с 1С:Предприятием. Часть 2](https://its.1c.ru/db/metod8dev#content:5866:hdoc)
    - [x] [ИТС. Настройки PostgreSQL](https://its.1c.ru/db/metod8dev#browse:13:-1:1989:2599:2600:2604)
    - [x] [Держи данные в тепле, транзакции в холоде, а VACUUM в голоде](https://is1c.ru/career/blog/derzhi-dannye-v-teple-tranzaktsii-v-kholode-a-vacuum-v-golode/)
    - [ ] [Простое обнаружение проблем производительности в PostgreSQL](https://infostart.ru/1c/articles/1198118/)
    - [x] [Работа с PostgreSQL настройка и масштабирование](https://postgresql.leopard.in.ua/) - 
    Перед вами справочное пособие по настройке и масштабированию PostgreSQL. В книге исследуются вопросы по настройке производительности PostgreSQL, репликации и кластеризации. Изобилие реальных примеров позволит как начинающим, так и опытным разработчикам быстро разобраться с особенностями масштабирования PostgreSQL для своих приложений.
    - [x] [PGTune](https://pgtune.leopard.in.ua/) - PGTune calculate configuration for PostgreSQL based on the maximum performance for a given hardware configuration. It isn't a silver bullet for the optimization settings of PostgreSQL. Many settings depend not only on the hardware configuration, but also on the size of the database, the number of clients and the complexity of queries. An optimal configuration of the database can only be made given all these parameters are taken into account.
    - [x] [3 WAYS TO DETECT SLOW QUERIES IN POSTGRESQL](https://www.cybertec-postgresql.com/en/3-ways-to-detect-slow-queries-in-postgresql/)
    - [x] [How to tune PostgreSQL for memory](https://www.enterprisedb.com/postgres-tutorials/how-tune-postgresql-memory)



!!! info "Репозитории"
    - [x] [Скрипты бекапа, восстановления, удаления и обслуживания баз 1С PostgreSQL (Windows)](https://github.com/anklav24/PostgreSQL-Scripts)
    - [x] [pgBadger - a fast PostgreSQL log analysis report](https://github.com/darold/pgbadger)


<details>  
  <summary>Настройка postgresql.conf</summary>

``` 
autovacuum = on
autovacuum_max_workers = Количество равным половине всех ядер сервера СУБД.
autovacuum_vacuum_cost_limit = 1
autovacuum_vacuum_cost_delay = 20ms
autovacuum_vacuum_scale_factor = 0.1 -> 0.01
autovacuum_analyze_scale_factor = 0.2 -> 0.005

```
</details>

!!! info "Описание параметров"

    **shared_buffers** — Количество памяти, выделенной PgSQL для совместного кеша страниц. Эта память разделяется между всеми процессами PgSQL. Делим доступную ОЗУ на 4. В нашем случае это 4 Gb.

    **effective_cache_size** — Оценка размера кэша файловой системы. Считается так: ОЗУ — shared_buffers. В нашем случае это 16Gb — 4Gb = 12Gb. Но рекомендуется указать этот параметр ниже, где-то 8-10Gb.

    **max_connections** = 100  максимальное число клиентских подключений, которые могут подсоединяться к базе данных одновременно.random_page_cost = 1.5 — 2.0 для RAID, 1.1 — 1.3 для SSD. Стоимость чтения рандомной страницы. Чем меньше seek time дисковой системы тем меньше (но > 1.0) должен быть этот параметр. Излишне большое значение параметра увеличивает склонность PgSQL к выбору планов с сканированием всей таблицы.
    
    **temp_buffers** = 512Mb. Максимальное количество страниц для временных таблиц. То есть это верхний лимит размера временных таблиц в каждой сессии (рекомендуемый размер 1/20 RAM).

    **wal_sync_method** = open_datasync.  метод, который используется для принудительной записи данных на диск. open_datasync – запись данных методом open() с параметром O_DSYNC, fdatasync – вызов метода fdatasync() после каждого commit, fsync_writethrough – вызывать fsync() после каждого commit игнорирую паралельные процессы, fsync – вызов fsync() после каждого commit, open_sync – запись данных методом open() с параметром O_SYNC. Наилучший по тесту для Windows считается open_datasync(для Линукс = fdatasync)
    
    **work_mem** — Считается так: ОЗУ / 32..64 — в нашем случае получается 512mb. Лимит памяти для обработки одного запроса. Эта память индивидуальна для каждой сессии. Теоретически, максимально потребная память равна max_connections * work_mem.

    **bgwriter_delay** — 20ms. Время сна между циклами записи на диск фонового процесса записи. Данный процесс ответственен за синхронизацию страниц, расположенных в shared_buffers с диском. Слишком большое значение этого параметра приведет к возрастанию нагрузки на  checkpoint процесс и процессы, обслуживающие сессии (backend). Малое значение приведет к полной загрузке одного из ядер.

    **synchronous_commit** — off. ОПАСНО! Выключение синхронизации с диском в момент коммита. Создает риск потери последних нескольких транзакций (в течении 0.5-1 секунды), но гарантирует целостность базы данных, в цепочке коммитов гарантированно отсутствуют пропуски. Но значительно увеличивает производительность.