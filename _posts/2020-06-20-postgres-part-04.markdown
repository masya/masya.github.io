---
layout: post
title: "PostgreSQL - часть 4" 
date: 2020-06-19 01:00:00 +0300
categories: postgresql
tags: [postgresql,install]
anons: "pgbouncer - пул соединений к PostgreSQL"
author: mAsYA
---

1) Установка pgbouncer

{% highlight shell %}
cd /usr/src/
sudo wget http://www.pgbouncer.org/downloads/files/1.13.0/pgbouncer-1.13.0.tar.gz
tar xzf pgbouncer-1.13.0.tar.gz
sudo apt-get install libevent-dev libssl-dev pkg-config
cd /usr/src/pgbouncer-1.13.0/
./configure --prefix=/usr/local --with-systemd
make
sudo make install
{% endhighlight %}

2) Конф-файл pgbouncer.ini

{% highlight shell %}
[databases]
db = host=127.0.0.1 port=5432 dbname=postgres auth_user=www
[pgbouncer]
logfile = /var/log/pgbouncer/pgbouncer.log
auth_query = SELECT uname, phash FROM user_lookup($1)
listen_addr = *
listen_port = 6432
auth_type = md5
auth_file = userlist.txt
admin_users = adm
pool_mode = transaction
{% endhighlight %}

3) Создание пользователя и выдача прав

{% highlight sql %}
CREATE USER www WITH PASSWORD '12345';
GRANT CONNECT ON DATABASE postgres TO www;
CREATE OR REPLACE FUNCTION user_lookup(in i_username text, out uname text, out phash text)
RETURNS record AS $$
BEGIN
    SELECT usename, passwd FROM pg_catalog.pg_shadow
    WHERE usename = i_username INTO uname, phash;
    RETURN;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
REVOKE ALL ON FUNCTION user_lookup(text) FROM public, postgres;
GRANT EXECUTE ON FUNCTION user_lookup(text) TO www;
{% endhighlight %}

4) Файл userlist.txt со списком пользователей, которым разрешено подключение

{% highlight shell %}
echo -n '12345www' | md5sum
561d7a90e46d1d21679fd4b44d6bec05
echo -n '12345adm' | md5sum
ed85866ddd9b072e5a3cab008bf771c4
cat /etc/pgbouncer/userlist.txt
"www" "md5561d7a90e46d1d21679fd4b44d6bec05"
"adm" "md5ed85866ddd9b072e5a3cab008bf771c4"
{% endhighlight %}

5) Запуск pgbouncer

{% highlight shell %}
pgbouncer -d /etc/pgbouncer/pgbouncer.ini
psql -U postgres -h 127.0.0.1 -p 6432 -d db
psql -h 127.0.0.1 -p 6432 -U adm pgbouncer
{% endhighlight %}

6) Ротация лога pgbouncer

{% highlight shell %}
sudo cat /etc/logrotate.d/pgbouncer
/var/log/pgbouncer/pgbouncer.log {
  daily
  rotate 3
  missingok
  notifempty
  sharedscripts
  nocompress
  create 0640 postgres postgres
  postrotate
         /bin/kill -HUP `cat /var/run/postgres/pgbouncer.pid 2> /dev/null` 2>/dev/null || true
  endscript
}
{% endhighlight %}
