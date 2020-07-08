---
layout: post
title: "PostgreSQL - часть 1" 
date: 2020-05-18 00:10:57 +0300
categories: postgresql
tags: [postgresql,install]
anons: "Сборка и установка из исходных кодов"
author: mAsYA
---

<b>1) Для сборки требуется ряд программ и утилит</b>

Обязательно: tar, gzip/bzip2, GNU make, компилятор Си (C89)

Используются, но можно отключить: библиотеки GNU Readline, zlib

Дополнительно: языки программирования Perl, Python, Tcl для использования PL/Perl, PL/Python, PL/Tcl; Kerberos, OpenSSL, OpenLDAP, PAM для аутентификации и шифрования

{% highlight shell %}
# OS - Ubuntu
sudo apt-get install gcc make flex bison libreadline-dev zlib1g-dev libxml2-utils libsystemd-dev libossp-uuid-dev
sudo apt-get install docbook docbook-dsssl docbook-xsl opensp xsltproc
{% endhighlight %}

<b>2) Скачиваем и распаковываем архив</b> 

{% highlight shell %}
# скачать архив с исходниками
sudo wget https://ftp.postgresql.org/pub/source/v12.3/postgresql-12.3.tar.gz
# создать папку для исходников
sudo mkdir /usr/src/postgresql-12.3
sudo chmod -R 777 /usr/src/postgresql-12.3
sudo chown postgres:postgres /usr/src/postgresql-12.3/
sudo su - postgres
cd /usr/src/
# распаковать архив
tar xzf /usr/src/postgresql-12.3.tar.gz
{% endhighlight %}

<b>3) Создание конфигурации</b>

В команде configure можно указать различные параметры конфигурации. Например:

--prefix - каталог установки, по умолчанию /usr/local/pgsql;

--bindir - каталог для исполняемых двоичных программ;

--with-systemd - поддержка служебных уведомлений для systemd;

{% highlight shell %}
cd /usr/src/postgresql-12.3/
# конфигурирование, сборка, установка 
./configure --with-systemd --bindir=/usr/bin --sbindir=/usr/sbin --prefix=/var/lib/pgsql/ --with-ossp-uuid
{% endhighlight %}

<b>4) Сборка PostgreSQL</b>

{% highlight shell %}
make -C /usr/src/postgresql-12.3/
{% endhighlight %}

<b>5) Установка PostgreSQL</b>

{% highlight shell %}
sudo make -C /usr/src/postgresql-12.3 install
{% endhighlight %}   

<b>6) Инициализация и запуск кластера</b>

{% highlight shell %}
# инициализация кластера
initdb -D /var/lib/pgsql/data/ -k
# старт СУБД
pg_ctl -w -D /var/lib/pgsql/data/ -l /var/log/postgres/logfile start
{% endhighlight %}

<b>7) Установка расширений</b>

{% highlight shell %}
cd /usr/src/postgresql-12.3/contrib/pg_stat_statements/
make
sudo make install
{% endhighlight %}

Подобным образом устанавливается большинство расширений. Но для некоторых требуются дополнительные действия.


