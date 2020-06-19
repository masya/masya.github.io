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

<b> Журнал</b>

<table>
<tr><td>Размер журнала в байтах между двумя LSN</td><td><h6>SELECT pg_current_wal_lsn() AS pos1 \gset<br>SELECT pg_current_wal_lsn() AS pos2 \gset<br>SELECT :'pos2'::pg_lsn-:'pos1'::pg_lsn</h6></td></tr>
<tr><td>журналы $PGDATA/pg_wal</td><td><h6>SELECT * FROM pg_ls_waldir() ORDER BY name</h6></td></tr>
</table>
