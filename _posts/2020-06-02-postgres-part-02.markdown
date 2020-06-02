---
layout: post
title: "PostgreSQL - часть 2" 
date: 2020-06-02 00:10:57 +0300
categories: postgresql
tags: [postgresql,install]
anons: "mamonsu — агент мониторинга для сбора метрик ОС и PostgreSQL"
author: mAsYA
---

<b>1) Варианты установки агента</b>

DEB packages for Debian/Ubuntu:

{% highlight shell %}
echo "deb [arch=amd64] https://repo.postgrespro.ru/mamonsu/latest/deb/ $(lsb_release -cs) main-$(lsb_release -cs)" > /etc/apt/sources.list.d/mamonsu.list
wget -O - https://repo.postgrespro.ru/mamonsu/keys/GPG-KEY-MAMONSU | sudo apt-key add - && sudo apt-get update
sudo apt-get install mamonsu
{% endhighlight %}

From git: 

{% highlight shell %}
git clone https://github.com/postgrespro/mamonsu.git && cd mamonsu && python setup.py build && python setup.py install
{% endhighlight %}

Build deb:

{% highlight shell %}
sudo apt-get install make dpkg-dev debhelper python-dev python-setuptools
git clone https://github.com/postgrespro/mamonsu.git && cd mamonsu && make deb && dpkg -i mamonsu*.deb
{% endhighlight %}

<b>2) Подготовка PostgreSQL</b>

{% highlight sql %}
CREATE USER user_mamonsu WITH PASSWORD 'PASSWORD';
CREATE DATABASE db_mamonsu OWNER user_mamonsu;
{% endhighlight %}

{% highlight shell %}
mamonsu bootstrap -M user_mamonsu -d db_mamonsu [параметры_подключения]
{% endhighlight %}   

<b>3) Настройка агента</b>

Вместе с пакетом распространяется предгенерированный конфиг. 

- Штатный конфиг: /etc/mamonsu/agent.conf

- Если вы пишите новые плагины, то сгенерировать конф-файл:

{% highlight shell %}
mamonsu export config /etc/mamonsu/agent.conf [параметры_экспорта]
{% endhighlight %}

- Выполнить настройку соединений в конф-файле.

<b>4) Экспорт шаблона в Zabbix</b>

{% highlight shell %}
# переменные окружения, которые можно передать через атрибуты
export ZABBIX_USER=USER
export ZABBIX_PASSWD=PASSWORD
export ZABBIX_URL=URL
# выгрузка шаблона в файл
mamonsu export template /etc/mamonsu/template.xml --template-name='PostgreSQL-Ubuntu' --application='App-PostgreSQL-Ubuntu'
# загрузка шаблона в Zabbix
mamonsu zabbix template export /etc/mamonsu/template.xml
# просмотр списка шаблонов
mamonsu zabbix template list
{% endhighlight %}

<b>5) Добавление хоста в Zabbix</b>

{% highlight shell %}
# создание группы хостов в Zabbix
mamonsu zabbix hostgroup create 'postgreSQL'
# получение hostgroup id
hostgroup_id=$(mamonsu zabbix hostgroup id 'postgreSQL')
# получение template id
template_id=$(mamonsu zabbix template id 'PostgreSQL-Ubuntu')
# создание хоста
mamonsu zabbix host create ИМЯ_УЗЛА_В_ZABBIX $hostgroup_id $template_id IP_MAMONSU
# запуск 
systemctl start mamonsu
{% endhighlight %}

