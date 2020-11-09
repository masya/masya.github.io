---
layout: post
title: "PostgreSQL - часть 7" 
date: 2020-11-05 01:00:00 +0300
categories: postgresql
tags: [postgresql,install]
anons: "barman - инструмент для резервного копирования"
author: mAsYA
---

<b>1) Рабочее окружение</b>

PG01|172.26.130.100|Primary
PG02|172.26.130.101|Standby
PG03|172.26.130.102|Backup

<b>2) Установка на Ubuntu/Debian</b>

Выполняем шаги из [инструкции] [install-barman]

- на бэкап-сервере

{% highlight shell %}
curl https://dl.2ndquadrant.com/default/release/get/deb | sudo bash
apt-get install barman barman-cli
{% endhighlight %}

<b>2) Настройка соединений по ssh</b>

Настройка резервного копирования начинается с процесса взаимного обмена RSA ключами

- на бэкап-сервере

{% highlight shell %}
su - barman
mkdir -p ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
ssh-keygen -t rsa
scp .ssh/id_rsa.pub USER@pg01:
scp .ssh/id_rsa.pub USER@pg02:
{% endhighlight %}

- на мастере и стэндбае

{% highlight shell %}
su - postgres
scp .ssh/id_rsa.pub USER@pg03:
cat ~USER/id_rsa.pub >> .ssh/authorized_keys
ssh barman@pg03
{% endhighlight %}

- на бэкап-сервере

{% highlight shell %}
su - barman
cat ~USER/id_rsa.pub >> .ssh/authorized_keys
ssh postgres@pg01
ssh postgres@pg02
{% endhighlight %}

<b>3) Настройка global</b>

- на бэкап-сервере

{% highlight shell %}
cat /etc/barman.conf
[barman]
configuration_files_directory = /etc/barman.d
barman_home = /var/lib/barman
barman_user = barman
log_file = /var/log/barman/barman.log
log_level = INFO
compression = gzip
immediate_checkpoint = true
basebackup_retry_times = 3
basebackup_retry_sleep = 30
last_backup_maximum_age = 1 DAYS
retention_policy = RECOVERY WINDOW OF 4 WEEKS
post_backup_script = /var/lib/barman/post_barman.sh
{% endhighlight %}

<b>4) Настройка master</b>

- на бэкап-сервере

{% highlight shell %}
cat /etc/barman.d/pg01.conf
[pg01]
description =  "Primary DB Server (via SSH)"
ssh_command = ssh postgres@172.26.130.100
conninfo = host=172.26.130.100 user=postgres port=5437
backup_method = rsync
reuse_backup = link
backup_options = concurrent_backup
parallel_jobs = 8
archiver = on
wal_retention_policy = main
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
path_prefix = "/var/lib/postgres/13.0/bin"
{% endhighlight shell %}

<b>5) Настройка standby</b>

- на бэкап-сервере

{% highlight shell %}
cat /etc/barman.d/pg02.conf
[pg02]
description = "Standby DB Server (streaming-only)"
conninfo = host=172.26.130.101 user=postgres port=5437
streaming_conninfo = host=172.26.130.101 user=postgres port=5437
backup_method = postgres
streaming_archiver = on
streaming_archiver_name = barman_receive_wal
archiver = off
slot_name = barman
wal_retention_policy = main
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
path_prefix = "/var/lib/postgres/13.0/bin"
{% endhighlight shell %}

<b>6) Копирование с master</b>

Путь, по которому будет складываться архив:

- на бэкап-сервере

{% highlight shell %}
barman show-server pg01 | grep incoming_wals_directory
incoming_wals_directory: /var/lib/barman/pg01/incoming
{% endhighlight shell %}

Изменения в конфигурации кластера на pg01:

{% highlight shell %}
ALTER SYSTEM SET archive_mode = on;
ALTER SYSTEM SET archive_command = 'rsync -a %p barman@172.26.130.102:/var/lib/barman/pg01/incoming/%f';
{% endhighlight shell %}

Проверка подключения:

- на бэкап-сервере

{% highlight shell %}
barman check pg01
{% endhighlight shell %}

Создание бэкапа:

- на бэкап-сервере

{% highlight shell %}
barman backup pg01
{% endhighlight shell %}

Просмотр списка бэкапов:

- на бэкап-сервере

{% highlight shell %}
barman list-backup pg01
{% endhighlight shell %}

Конкретный архив:

- на бэкап-сервере

{% highlight shell %}
barman show-backup pg01 20201108T213920
{% endhighlight shell %}

Создание бэкапа по расписанию:

- на бэкап-сервере

{% highlight shell %}
crontab -l
00 22 * * * /usr/bin/barman backup pg01
* * * * * /usr/bin/barman cron
{% endhighlight shell %}

<b>7) Копирование с standby</b>

Изменения в конфигурации кластера на pg02:

{% highlight shell %}
ALTER SYSTEM SET synchronous_standby_names = 'barman_receive_wal';
{% endhighlight shell %}

Создание слота:

- на бэкап-сервере

{% highlight shell %}
barman receive-wal --create-slot pg02
{% endhighlight shell %}

Проверка соединения:

- на бэкап-сервере

{% highlight shell %}
barman check pg02
{% endhighlight shell %}

Проверка статуса потоковой репликации:

- на бэкап-сервере

{% highlight shell %}
$ barman replication-status pg02
{% endhighlight shell %}

Создание резервной копии:                                                                                                            

- на бэкап-сервере

{% highlight shell %}
$ barman backup pg02
{% endhighlight shell %}

<b>8) Восстановление из архивов standby</b>

Восстановление на время из архивов standby с использованием бэкапа-20201108T235034 в папку /var/lib/postgres/13.0/data/pg на сервер pg02 

- на бэкап-сервере

{% highlight shell %}
barman recover --target-time "2020-11-09 01:27:41+03:00" --remote-ssh-command "ssh postgres@172.26.130.101" pg02 20201108T235034 /var/lib/postgres/13.0/data/pg --get-wal
{% endhighlight shell %}

В случае ошибки в логе:

{% highlight shell %}
2020-11-09 02:27:21.313 MSK [32232] LOG:  invalid record length at 0/11000578: wanted 24, got 0
ERROR: Remote 'barman get-wal' command has failed!
{% endhighlight shell %}

На сервере, котором выполняется восстановление:

{% highlight shell %}
pg_waldump -p /var/lib/postgres/13.0/data/pg/pg_wal -s 0/11000400 -e 0/11000800
{% endhighlight shell %}

Требуется найти время последнего COMMIT и восстанавливать на этот момент времени:

{% highlight shell %}
613, lsn: 0/11000430, prev 0/11000148, desc: COMMIT 2020-11-09 02:12:48.093720 MSK
{% endhighlight shell %}

<b>9) Восстановление из архивов master</b> 

- на бэкап-сервере

{% highlight shell %}
barman recover --target-time "2020-11-09 01:29:00+03" --remote-ssh-command "ssh postgres@172.26.130.101" pg01 20201108T234905 /var/lib/postgres/13.0/data/pg
{% endhighlight shell %}

<b>10) [Hook scrips] [hook-barman] </b>

{% highlight shell %}
cat /var/lib/barman/post_barman.sh
#!/bin/bash

host=$(hostname -f)
export MAILTO=WWW@WWW.RU

file_name=/tmp/barman_backup.info
desc="backup status $BARMAN_STATUS cluster $BARMAN_SERVER on the host $host"

echo "Host                          : $host" > $file_name
echo "Name of the server            : $BARMAN_SERVER" >> $file_name
echo "ID of the backup              : $BARMAN_BACKUP_ID" >> $file_name
echo "Backup destination directory  : $BARMAN_BACKUP_DIR ">> $file_name
echo "ID of the previous backup     : $BARMAN_PREVIOUS_ID" >> $file_name
echo " " >> $file_name
echo "Status of the backup          : $BARMAN_STATUS" >> $file_name
echo "Error message                 : $BARMAN_ERROR " >> $file_name
echo "Version of Barman             : $BARMAN_VERSION" >> $file_name

echo " " >> $file_name
echo "--- description ---" >> $file_name
echo " " >> $file_name

barman show-backup $BARMAN_SERVER $BARMAN_BACKUP_ID >> $file_name

cat $file_name | mailx -s "$desc" $MAILTO

rm $file_name
{% endhighlight shell %}

[install-barman]: http://docs.pgbarman.org/release/2.12/#installation-on-debianubuntu-using-packages
[hook-barman]: http://docs.pgbarman.org/release/2.12/#hook-scripts
