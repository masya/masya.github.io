---
layout: post
title: "PostgreSQL - часть 6" 
date: 2020-10-01 01:00:00 +0300
categories: postgresql
tags: [postgresql,install]
anons: "automatic failover с помощью демона repmgrd"
author: mAsYA
---

<b>1) Сборка из исходных кодов PostgreSQL 13.0 на всех узлах</b>

{% highlight shell %}
sudo wget https://ftp.postgresql.org/pub/source/v13.0/postgresql-13.0.tar.gz
sudo mkdir /usr/src/postgresql-13.0
sudo chmod -R 777 /usr/src/postgresql-13.0
sudo chown postgres:postgres /usr/src/postgresql-13.0
su - postgres
cd /usr/src/
tar xzf postgresql-13.0.tar.gz
cd postgresql-13.0
./configure --prefix=/var/lib/pgsql/13.0/
make -C /usr/src/postgresql-13.0/
sudo make -C /usr/src/postgresql-13.0/ install
{% endhighlight %}

<b>2) Сборка из исходных кодов repmgr 5.1 на всех узлах</b>

{% highlight shell %}
sudo wget https://repmgr.org/download/repmgr-5.1.0.tar.gz
sudo mkdir /usr/src/repmgr-5.1.0
sudo chmod -R 777 /usr/src/repmgr-5.1.0
sudo chown postgres:postgres /usr/src/repmgr-5.1.0
su - postgres
cd /usr/src/
tar xzf repmgr-5.1.0.tar.gz
cd repmgr-5.1.0/
export PATH="/var/lib/pgsql/13.0/bin/:$PATH"
./configure
make
sudo make -C /usr/src/repmgr-5.1.0/ install
{% endhighlight shell %}

<b>3) Выполняем шаги из [Части 5] [repmgr-load]</b>

- Подготовка PostgreSQL на Primary

- Настройка repmgr на Primary и Standby узлах

- Настройка pg_hba.conf на Primary узле

- Регистрация Primary в repmgr

- Клонирование Standby узлов

- Регистрация Standby узлов в repmgr

- Проверка статуса репликации

<b>4) Настройка repmgr на Witness-node

Настройка postgresql.conf на Witness-node

{% highlight shell %}
echo "listen_addresses = '*'" >> /var/lib/pgsql/13.0/data/postgresql.conf
echo "shared_preload_libraries = 'repmgr'" >> /var/lib/pgsql/13.0/data/postgresql.conf
{% endhighlight shell %}

Настройка pg_hba.conf на Witness-node

{% highlight shell %}
local   replication     repmgr                                  trust
host    replication     repmgr          127.0.0.1/32            trust
host    replication     repmgr          172.26.130.0/24         trust
local   repmgr          repmgr                                  trust
host    repmgr          repmgr          127.0.0.1/32            trust
host    repmgr          repmgr          172.26.130.0/24         trust
{% endhighlight shell %}

Создание и настройка пользователя, создание базы данных

{% highlight shell %}
export PATH="/var/lib/pgsql/13.0/bin/:$PATH"
createuser -p 5435 --superuser repmgr
createdb -p 5435 --owner=repmgr repmgr
psql -p 5435 -c 'ALTER USER repmgr SET search_path TO repmgr, "$user", public;'
{% endhighlight shell %}

Создание конф файла для repmgr

{% highlight shell %}
echo "node_id=4" >> /etc/repmgr/13/repmgr.conf
echo "node_name='PGwitness'" >> /etc/repmgr/13/repmgr.conf
echo "conninfo='host=172.26.130.103 user=repmgr port=5435 dbname=repmgr connect_timeout=2'" >> /etc/repmgr/13/repmgr.conf
echo "data_directory='/var/lib/pgsql/13.0/data'" >> /etc/repmgr/13/repmgr.conf
{% endhighlight shell %}

Проверяем доступность и регистрируем Witness-node

{% highlight shell %}
psql 'host=172.26.130.100 user=repmgr port=5435 dbname=repmgr connect_timeout=2'
/var/lib/pgsql/13.0/bin/repmgr -f /etc/repmgr/13/repmgr.conf witness register -h 172.26.130.100 -p 5435
/var/lib/pgsql/13.0/bin/repmgr -f /etc/repmgr/13/repmgr.conf cluster show
{% endhighlight shell %}

<b>5) Изменение файла sudoers</b>

{% highlight shell %}
echo "Defaults:postgres !requiretty" >> /etc/sudoers
echo "postgres ALL = NOPASSWD: /etc/init.d/postgresql13" >> /etc/sudoers
{% endhighlight shell %}

<b>6) Настройка параметров repmgrd</b>

{% highlight shell %}
echo "failover='automatic'" >> /etc/repmgr/13/repmgr.conf
echo "promote_command='/var/lib/pgsql/13.0/bin/repmgr standby promote -f /etc/repmgr/13/repmgr.conf --log-to-file'" >> /etc/repmgr/13/repmgr.conf
echo "follow_command='/var/lib/pgsql/13.0/bin/repmgr standby follow -f /etc/repmgr/13/repmgr.conf --log-to-file --upstream-node-id=%n'" >> /etc/repmgr/13/repmgr.conf
echo "monitor_interval_secs = 2" >> /etc/repmgr/13/repmgr.conf
echo "connection_check_type = 'ping'" >> /etc/repmgr/13/repmgr.conf
echo "reconnect_attempts = 4" >> /etc/repmgr/13/repmgr.conf
echo "reconnect_interval = 8" >> /etc/repmgr/13/repmgr.conf
# primary_visibility_consensus - на всех узлах
echo "primary_visibility_consensus = true" >> /etc/repmgr/13/repmgr.conf
echo "standby_disconnect_on_failover = true" >> /etc/repmgr/13/repmgr.conf
echo "repmgrd_service_start_command = '/var/lib/pgsql/13.0/bin/repmgrd -f /etc/repmgr/13/repmgr.conf -d'" >> /etc/repmgr/13/repmgr.conf
echo "repmgrd_service_stop_command = 'kill \`cat \$(/var/lib/pgsql/13.0/bin/repmgrd --show-pid-file)\`'" >> /etc/repmgr/13/repmgr.conf
echo "service_start_command='sudo /etc/init.d/postgresql13 start'" >> /etc/repmgr/13/repmgr.conf
echo "service_stop_command='sudo /etc/init.d/postgresql13 stop'" >> /etc/repmgr/13/repmgr.conf
echo "service_restart_command='sudo /etc/init.d/postgresql13 restart'" >> /etc/repmgr/13/repmgr.conf
echo "service_reload_command='sudo /etc/init.d/postgresql13 reload'" >> /etc/repmgr/13/repmgr.conf
echo "monitoring_history=yes" >> /etc/repmgr/13/repmgr.conf
echo "log_status_interval=60" >> /etc/repmgr/13/repmgr.conf
echo "log_file='/var/log/postgresql/repmgrd.log'" >> /etc/repmgr/13/repmgr.conf
{% endhighlight shell %}

<b>7) Запуск демона repmgrd</b>

на основном узле, на резервных узлах и узле-свидетеле

{% highlight shell %}
export PATH="/var/lib/pgsql/13.0/bin/:$PATH"
repmgr -f /etc/repmgr/13/repmgr.conf daemon start --dry-run
repmgr -f /etc/repmgr/13/repmgr.conf daemon start
repmgr -f /etc/repmgr/13/repmgr.conf cluster event --event=repmgrd_start
{% endhighlight shell %}

<b>8) Имитация сбоя на Primary</b>

Остановка кластера 

{% highlight shell %}
$ sudo /etc/init.d/postgresql13 stop
{% endhighlight shell %}

События при переключении

 Node ID : Event                      | Details                                                           
 2       : child_node_new_connect     | new witness "PGwitness" (ID: 4) has connected                     
 4       : repmgrd_upstream_reconnect | witness monitoring connection to primary node "PG02" (ID: 2)      
 4       : repmgrd_failover_follow    | witness node 4 now following new primary node 2                   
 3       : repmgrd_failover_follow    | node 3 now following new upstream node 2                          
 3       : standby_follow             | standby attached to upstream node "PG02" (ID: 2)                  
 2       : child_node_new_connect     | new standby "PG03" (ID: 3) has connected                          
 2       : repmgrd_reload             | monitoring cluster primary "PG02" (ID: 2)                         
 2       : repmgrd_failover_promote   | node 2 promoted to primary; old primary 1 marked as failed        
 2       : standby_promote            | server "PG02" (ID: 2) was successfully promoted to primary               

[repmgr-load]: https://masya.github.io/postgresql/2020/07/04/postgres-part-05.html
