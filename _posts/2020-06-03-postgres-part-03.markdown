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
