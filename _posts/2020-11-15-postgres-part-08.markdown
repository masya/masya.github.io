---
layout: post
title: "PostgreSQL - часть 8" 
date: 2020-11-09 01:00:00 +0300
categories: postgresql
tags: [postgresql,install]
anons: "postgres_exporter — экспортер метрик PostgreSQL для Prometheus"
author: mAsYA
---

[postgres_exporter] [postgres-exporter] — утилита сбора метрик с экземпляров кластера СУБД PostgreSQL в формате доступном Prometheus, написанная на языке Go, имеет открытый исходный код и распространяется бесплатно. 

<b>1) Установка postgres_exporter</b>

postgres_exporter - самодостаточное приложение, скомпилированное под необходимый дистрибутив и архитектуру, и не требующее установки дополнительных пакетов на сервер. В репозитории проекта имеется YAML-файл с пользовательским набором метрик: [queries.yaml] [queries-yaml]

{% highlight shell %}
sudo -s
export VERSION="0.8.0"
wget https://github.com/wrouesnel/postgres_exporter/releases/download/v${VERSION}/postgres_exporter_v${VERSION}_linux-386.tar.gz -O - | tar -xzv -C /tmp
cp /tmp/postgres_exporter_v0.8.0_linux-386/postgres_exporter /usr/local/bin/
rm -rf /tmp/postgres_exporter_v0.8.0_linux-386
chown -R postgres:postgres /usr/local/bin/postgres_exporter
{% endhighlight %}

Сценарий запуска systemd сервиса postgres_exporter:

{% highlight shell %}
sudo touch /etc/systemd/system/postgres_exporter.service
sudo chmod 755 /etc/systemd/system/postgres_exporter.service
cat /etc/systemd/system/postgres_exporter.service
[Unit]
Description=Prometheus PostgreSQL Exporter
After=network.target

[Service]
Type=simple
Restart=always
User=postgres
Group=postgres
Environment="DATA_SOURCE_NAME=postgresql://postgres@127.0.0.1:5432/postgres?sslmode=disable"
Environment="DATA_SOURCE_PASS_FILE=/home/postgres/.pgpass"
ExecStart=/usr/local/bin/postgres_exporter --auto-discover-databases --extend.query-path=/home/postgres/queries.yaml
[Install]
WantedBy=multi-user.target
{% endhighlight shell %}

Запускаем и проверям:

{% highlight shell %}
sudo systemctl start postgres_exporter.service
sudo systemctl status postgres_exporter.service
sudo journalctl -u postgres_exporter.service
curl http://localhost:9187/metrics
{% endhighlight shell %}

<b>2) Запуск Prometheus</b>

{% highlight shell %}
sudo useradd --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.22.1/prometheus-2.22.1.linux-386.tar.gz
tar -xvzf prometheus-2.22.1.linux-386.tar.gz
sudo cp prometheus-2.22.1.linux-386/prometheus /usr/local/bin/
sudo cp prometheus-2.22.1.linux-386/promtool /usr/local/bin/
sudo mkdir /etc/prometheus
sudo cp -r prometheus-2.22.1.linux-386/consoles/ /etc/prometheus/consoles
sudo cp -r prometheus-2.22.1.linux-386/console_libraries/ /etc/prometheus/console_libraries
sudo cp prometheus-2.22.1.linux-386/prometheus.yml /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
{% endhighlight shell %}

Добавляем адрес postgres_exporter:

{% highlight shell %}
vi /etc/prometheus/prometheus.yml
  - job_name: postgresql
    static_configs:
      - targets: ['172.26.130.100:9187']
        labels:
          alias: postgres
{% endhighlight shell %}

Создаем сервис:

{% highlight shell %}
sudo touch /etc/systemd/system/prometheus.service
sudo chmod 755 /etc/systemd/system/prometheus.service
cat /etc/systemd/system/prometheus.service
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=default.target
{% endhighlight shell %}

Проверка:

{% highlight shell %}
sudo systemctl daemon-reload
sudo systemctl start prometheus.service
sudo systemctl status prometheus.service
sudo journalctl --full --no-pager -u prometheus
{% endhighlight shell %}

Простой веб-интерфейс доступен по адресу http://localhost:9090/graph

![web-prometheus](https://drive.google.com/uc?export=view&id=1rO67dXbaSCS3tpmfLxtlAHGB866USR3E){:height="150px"}

<b>4) Установка Grafana</b>

{% highlight shell %}
sudo apt-get install -y adduser libfontconfig1
wget https://dl.grafana.com/oss/release/grafana_7.3.1_amd64.deb
sudo dpkg -i grafana_7.3.1_amd64.deb
sudo systemctl start grafana-server
sudo systemctl status grafana-server
{% endhighlight shell %}

Далее идем на порт 3000 с логином и паролем admin. Grafana сразу предложит сменить пароль. Добавляем Prometheus в качестве источника данных.

Для установки Dashboard идем в диалог Import Dashboard. Открывается по следующей ссылке http://grafana-host:3000/dashboard/import

[postgres-exporter]: https://github.com/wrouesnel/postgres_exporter
[queries-yaml]: https://raw.githubusercontent.com/wrouesnel/postgres_exporter/master/queries.yaml
