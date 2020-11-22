---
layout: post
title: "Telegram bot для отправки сообщений"
date: 2020-11-15 00:01:00 +0300
categories: other
tags: [github, telegram, bash]
author: mAsYA
anons: "отправка уведомлений в telegram чат для мониторинга сервера"
---
Боты - специальные аккаунты в Telegram, созданные для того, чтобы автоматически обрабатывать и отправлять сообщения.

<b>1) Создание чат-бота в Telegram</b>

- для создания чат-бота в Telegram используется бот @BotFather. Чтобы создать своего бота, нужно найти @BotFather у себя в Telegram и перейти в чат с ним;

- нажать START. Выбрать команду /newbot. Ввести name. Ввести username, оканчивающийся на bot;

- получить API токен бота, который потребуется при настройке;

- перейти в чат, нажать START, отправить любой символ;

- по адресу: https://api.telegram.org/bot&lt;token&gt;/getUpdates
 
	где &lt;token&gt; - это API-токен, полученный на предыдущем шаге

	получить chat_id - это ID чата с ботом.

<b>2) Отправка в чат Telegram уведомлений</b>

С помощью ранее созданного бота и полученных ID можно отсылать с сервера уведомления в Telegram чат, и таким образом, получать данные или алерты.

Для отправки сообщения в чат, нужно использовать следующий URL:

https://api.telegram.org/bot&lt;token&gt;/sendMessage?chat_id=&lt;chat_id&gt;&text=&lt;text&gt;

где &lt;token&gt; — это API который выдал @BotFather

&lt;chat_id&gt; — это ID вашего чата с ботом.

Например:

{% highlight shell_session %}
curl -s -X POST https://api.telegram.org/bot123456:ABC-DEF1234ghIkl-zyx57W2v1u123ew11/sendMessage -d chat_id=123456789 -d text="Hello world!"
{% endhighlight %}

<b>3) Скрипт отправки bash</b>

{% highlight shell_session %}
#!/bin/bash
echo "Press CTRL+C to proceed."
trap "pkill -f 'sleep 1h'" INT
trap "set +x ; sleep 1h ; set -x" DEBUG
export http_proxy="HTTP_PROXY"
export https_proxy="HTTP_PROXY"
CHATID="ИД через пробел"
APITOKEN="API токен бота"
TIMEOUT="10"
URL="https://api.telegram.org/bot$APITOKEN/sendMessage"
TEXT=$1,$2,$3,$4,$5
for IDTELEGRAM in $CHATID
do
curl -s --max-time $TIMEOUT -d "chat_id=$IDTELEGRAM&disable_web_page_preview=1&text=$TEXT" $URL > /dev/null
done
{% endhighlight %}
