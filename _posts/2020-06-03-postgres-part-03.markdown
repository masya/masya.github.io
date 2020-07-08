---
layout: post
title: "PostgreSQL - часть 3" 
date: 2020-06-03 01:00:00 +0300
categories: postgresql
tags: [postgresql,install]
anons: "полезные запросы"
author: mAsYA
---

<b> Параметры конфигурации</b>

<table>
<tr><td>Текущие настройки параметров</td><td><h6>SELECT * FROM pg_settings WHERE name='ПАРАМЕТР'</h6></td></tr>
<tr><td>Параметры из конф-файлов (реальные значения могут отличаться)</td><td><h6>SELECT * FROM pg_file_settings WHERE name='ПАРАМЕТР'</h6></td></tr>
<tr><td>Чтение значения параметра во время выполнения</td><td><h6>SHOW ПАРАМЕТР<br>SELECT current_setting('ПАРАМЕТР')</h6></td></tr>
<tr><td>Установка параметра во время выполнения (true для транзакции; false для сеанса)</td><td><h6>SELECT set_config('ПАРАМЕТР', 'ЗНАЧЕНИЕ', false)</h6></td></tr>
</table>

<b> Управление доступом</b>
<table>
<tr><td>Роль на которую переключились, роль под которой создали сессию</td><td><h6>SELECT current_user, session_user</h6></td></tr>
<tr><td>Список ролей<br>Список привилегий для схемы<br>Список привилегий на объект<br>Список привилегий по умолчанию на объект</td><td><h6>\du+<br>\dn+<br>\dp+<br>\ddp+</h6></td></tr>
<tr><td>Посмотреть файл pg_hba.conf</td><td><h6>SELECT line_number, type, database, user_name, address, auth_method FROM pg_hba_file_rules</h6></td></tr>
<tr><td>Проверка корректности добавленных записей pg_hba.conf</td><td><h6>SELECT line_number, error FROM pg_hba_file_rules WHERE error IS NOT NULL</h6></td></tr>
</table>

<b> СТАТИСТИКА</b>

<table>
<tr><td>Статистика обращений к таблице в строках</td><td><h6>SELECT * FROM pg_stat_all_tables WHERE relid = 'ТАБЛИЦА:wq'::regclass</h6></td></tr>
<tr><td>Статистика обращений к таблице в страницах</td><td><h6>SELECT * FROM pg_statio_all_tables WHERE relid = 'ТАБЛИЦА'::regclass</h6></td></tr>
<tr><td>Статистика по БД</td><td><h6>SELECT * FROM pg_stat_database WHERE datname = 'БД'</h6></td></tr>
<tr><td>Статистика по процессам фоновой записи и контрольной точки</td><td><h6>SELECT * FROM pg_stat_bgwriter</h6></td></tr>
</table>

<b> СИСТЕМНЫЙ КАТАЛОГ</b>

<table>
<tr><td>OID relations с именами начинающимися PREFIX</td><td><h6>SELECT relname, relkind, relnamespace, relfilenode, relowner, reltablespace FROM pg_class WHERE relname ~ '^(PREFIX1|PREFIX2).*'</h6></td></tr>
</table>

<b> БАЗЫ ДАННЫХ</b>

<table>
<tr><td>Размер БД</td><td><h6>SELECT pg_size_pretty(pg_database_size('БД'))</h6></td></tr>
<tr><td>Изменение размера БД</td><td><h6>SELECT pg_database_size('БД') as size1 \gset<br>SELECT pg_database_size('БД') as size2 \gset<br>SELECT pg_size_pretty(:size2::bigint-:size1::bigint)</h6></td></tr>
</table>

<b> ТАБЛИЧНЫЕ ПРОСТРАНСТВА</b>

<table>
<tr><td>Размер ТП</td><td><h6>SELECT pg_size_pretty(pg_tablespace_size('ТП'))</h6></td></tr>
<tr><td>Найти объекты в ТП:<br>- OID ТП<br>- список имен БД<br>- соединение с БД из списка<br>- поиск объектов</td><td><h6>SELECT OID AS tsoid FROM pg_tablespace WHERE spcname='ТП'\gset<br>SELECT datname FROM pg_database WHERE OID IN (SELECT pg_tablespace_databases(:tsoid))<br>\c БД<br>SELECT relnamespace::regnamespace, relname, relkind FROM pg_class WHERE reltablespace = :tsoid</h6></td></tr>
<tr><td>ТП БД по умолчанию</td><td><h6>SELECT spcname FROM pg_tablespace WHERE OID = (SELECT dattablespace FROM pg_database WHERE datname = 'БД')</h6></td></tr>
</table>

<b> СХЕМЫ</b>                                                                                                                              
<table>
<tr><td>Путь поиска</td><td><h6>SHOW search_path<br>SELECT current_schemas(true)</h6></td></tr>
<tr><td>Список всех схем включая системные</td><td><h6>\set ECHO_HIDDEN on<br>\dnS+</h6></td></tr>
</table>

<b> Файлы объектов БД и слои</b>
<table>
<tr><td>Базовое имя файла объекта БД относительно PGDATA<br>- OID БД<br>- имя файла данных</td><td><h6>SELECT pg_relation_filepath('ОБЪЕКТ')<br>SELECT OID from pg_database WHERE datname = 'БД'<br>SELECT relfilenode FROM pg_class where relname='ОБЪЕКТ'</h6></td></tr>
<tr><td>Размер каждого слоя</td><td><h6>SELECT pg_relation_size('ОБЪЕКТ','main') main, pg_relation_size('ОБЪЕКТ','fsm') fsm, pg_relation_size('ОБЪЕКТ','vm') vm</h6></td></tr>
</table>

<b> Таблицы</b>
<table>
<tr><td></td><td><h6></h6></td></tr>
<tr><td></td><td><h6></h6></td></tr>
</table>    

<b> TOAST-таблицы</b>
<table>
<tr><td>Названия и идентификаторы TOAST-таблиц</td><td><h6>SELECT relname, relfilenode FROM pg_class WHERE OID = (SELECT reltoastrelid FROM pg_class WHERE relname = 'TABLE')</h6></td></tr>
<tr><td></td><td><h6></h6></td></tr>                                                                                                             </table>

<b> Блокировки</b>
<table>
<tr><td>- pid заблокированного процесса<br>- информация по блокировкам<br>- прерываем блокирующую транзакцию</td><td><h6>SELECT pid as blocked_pid  FROM pg_stat_activity WHERE backend_type = 'client backend' AND cardinality(pg_blocking_pids(pid)) > 0 \gset<br>SELECT locktype, transactionid, pid, mode, granted FROM pg_locks WHERE transactionid IN (SELECT transactionid FROM pg_locks WHERE pid = :blocked_pid AND NOT granted)<br>SELECT pg_terminate_backend(b.pid) FROM unnest(pg_blocking_pids(:blocked_pid)) AS b(pid)</h6></td></tr>
<tr><td></td><td><h6></h6></td></tr>
</table>

<b> Журнал</b>

<table>
<tr><td>Размер журнала в байтах между двумя LSN</td><td><h6>SELECT pg_current_wal_lsn() AS pos1 \gset<br>SELECT pg_current_wal_lsn() AS pos2 \gset<br>SELECT :'pos2'::pg_lsn-:'pos1'::pg_lsn</h6></td></tr>
<tr><td>журналы $PGDATA/pg_wal</td><td><h6>SELECT * FROM pg_ls_waldir() ORDER BY name</h6></td></tr>
</table>
