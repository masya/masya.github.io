---
layout: post
title: "PostgreSQL - часть 5" 
date: 2020-07-05 01:00:00 +0300
categories: postgresql
tags: [postgresql,install]
anons: "настройка репликации с помощью repmgr"
author: mAsYA
---

repmgr используется для автоматизации и управления потоковой репликацией PostgreSQL. repmgr - продукт 2ndQuadrant  с открытым исходным кодом.

<b>1) Архитектура кластера</b>

<table>
<tr><td>PG01</td><td>172.26.130.100</td><td>Primary</td></tr>
<tr><td>PG02</td><td>172.26.130.101</td><td>Standby</td></tr>
<tr><td>PG03</td><td>172.26.130.102</td><td>Standby</td></tr>
<tr><td>PG04</td><td>172.26.130.103</td><td>Witness</td></tr>
</table>

<b>2) Установка PostgreSQL 12 на всех узлах</b>

[Установка PostgreSQL из исходных кодов на всех узлах и инициализация кластера на Primary] [install-load]

<b>3) Установка repmgr на всех узлах</b>

{% highlight shell %}
sudo cd /usr/src/
sudo wget https://github.com/2ndQuadrant/repmgr/archive/v5.1.0.tar.gz
sudo su - postgres
cd /usr/src/
tar xzf v5.1.0.tar.gz
cd repmgr-5.1.0/
export PATH="/var/lib/pgsql/12.3/bin/:$PATH"
./configure
make
sudo make install
{% endhighlight %}

<b>4) Подготовка PostgreSQL на Primary</b>

{% highlight shell %}
sudo su - postgres
# создание пользователя repmgr для приложения
createuser -p 5434 --superuser repmgr
# создание БД для хранения метаданных
createdb -p 5434 --owner=repmgr repmgr
# путь поиска по умолчанию для пользователя repmgr
psql -p 5434 -c 'ALTER USER repmgr SET search_path TO repmgr, "$user", public;'
# добавляем параметры конфигурации в postgresql.conf
echo "listen_addresses = '*'" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "max_wal_senders = 10" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "max_replication_slots = 10" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "wal_level = 'replica'" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "hot_standby = on" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "archive_mode = on" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "archive_command = '/bin/true'" >> /var/lib/pgsql/12.3/data/postgresql.conf
echo "shared_preload_libraries = 'repmgr'" >> /var/lib/pgsql/12.3/data/postgresql.conf
# рестарт PostgreSQL
{% endhighlight %}

<b>5) Настройка repmgr на Primary и Standby узлах</b>

На Primary создаем файл конфигурации repmgr

{% highlight shell %}
sudo su - postgres
echo "node_id=1" > /etc/repmgr/12/repmgr.conf
echo "node_name='PG01'" >> /etc/repmgr/12/repmgr.conf
echo "conninfo='host=172.26.130.100 user=repmgr port=5434 dbname=repmgrconnect_timeout=2'" >> /etc/repmgr/12/repmgr.conf
echo "data_directory='/var/lib/pgsql/12.3/data'" >> /etc/repmgr/12/repmgr.conf
{% endhighlight %}

На Standby

{% highlight shell %}
sudo su - postgres
echo "node_id=2" > /etc/repmgr/12/repmgr.conf
echo "node_name='PG02'" >> /etc/repmgr/12/repmgr.conf
echo "conninfo='host=172.26.130.101 user=repmgr port=5434 dbname=repmgrconnect_timeout=2'" >> /etc/repmgr/12/repmgr.conf
echo "data_directory='/var/lib/pgsql/12.3/data'" >> /etc/repmgr/12/repmgr.conf
{% endhighlight %}

<b>6) Настройка pg_hba.conf на Primary узле</b>

Добавим строки в pg_hba.conf. Файл будет скопирован repmgr с Primary на Standby при настройке репликации.

{% highlight shell %}
local   replication     repmgr                                  trust
host    replication     repmgr          127.0.0.1/32            trust
host    replication     repmgr          172.26.130.0/24         trust

local   repmgr          repmgr                                  trust
host    repmgr          repmgr          127.0.0.1/32            trust
host    repmgr          repmgr          172.26.130.0/24         trust
{% endhighlight %}

{% highlight shell %}
sudo su - postgres
# Рестарт PostgreSQL
# проверка соединения standby > primary
psql 'host=172.26.130.100 user=repmgr port=5434 dbname=repmgr connect_timeout=2'
{% endhighlight %}

<b>7) Регистрация Primary в repmgr</b>

{% highlight shell %}
sudo su - postgres
/var/lib/pgsql/12.3/bin/repmgr -f /etc/repmgr/12/repmgr.conf primary register
# проверка состояния кластера
/var/lib/pgsql/12.3/bin/repmgr -f /etc/repmgr/12/repmgr.conf cluster show
{% endhighlight %}

<b>8) Клонирование Standby узлов</b>

{% highlight shell %}
sudo su - postgres
# проверка операции клонирования с ключем --dry-run
/var/lib/pgsql/12.3/bin/repmgr -h 172.26.130.100 -U repmgr -p 5434 -d repmgr -f /etc/repmgr/12/repmgr.conf standby clone --dry-run
# если узел завершает операцию со статусом
# INFO: all prerequisites for "standby clone" are met
# выполняем фактическое клонирование
/var/lib/pgsql/12.3/bin/repmgr -h 172.26.130.100 -U repmgr -p 5434 -d repmgr -f /etc/repmgr/12/repmgr.conf standby clone
{% endhighlight %}

<b>9) Регистрация Standby узлов в repmgr</b>

{% highlight shell %}
sudo su - postgres
# Запуск PostgreSQL на Standby
/var/lib/pgsql/12.3/bin/repmgr -f /etc/repmgr/12/repmgr.conf standby register
# должно быть получено сообщение об успешной регистрации
# NOTICE: standby node "PG02" (ID: 2) successfully registered
{% endhighlight %}

<b>10) Проверка статуса репликации</b>

{% highlight shell %}
sudo su - postgres
/var/lib/pgsql/12.3/bin/repmgr -f  /etc/repmgr/12/repmgr.conf cluster show --compact
{% endhighlight %}

[install-load]: https://masya.github.io/postgresql/2020/05/17/postgres-part-01.html
