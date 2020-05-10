---
layout: post
title: "Talend’s Open Studio for ESB - часть 3" 
date: 2020-05-07 00:17:57 +0300
categories: talend
tags: [talend, wsdl, service, soap]
anons: "Deploy сервиса и клиента в Talend ESB runtime container"
---

Ранее мы рассмотрели, как `Talend Open Studio ESB` можно использовать для разработки и тестирования веб-службы SOAP и для создания клиента этой службы. В этой статье мы покажем, как развернуть ранее созданный сервис и клиент в Talend ESB runtime container.

<b>1) Talend ESB runtime container</b>

В первой части мы запустили Talend Studio через каталог Studio, чтобы спроектировать и протестировать SOAP сервис DoubleIt. Однако, это подходит только для тестирования дизайна сервиса. Когда мы будем готовы развернуть службу или клиент, который мы создали в Studio, нам понадобится runtime контейнер, который доступен в каталоге Runtime_ESBSE в дистрибутиве Talend Open Studio for ESB. Рассматриваемый runtime контейнер - это мощный и готовый к работе контейнер, основанный на Apache Karaf. Мы можем запустить его в каталоге Runtime_ESBSE/container/bin/trun.

По умолчанию Talend ESB runtime стартует с дефолтным набором плагинов (которые можно просмотреть командой la). Все нужные нам библиотеки будут запущены автоматически, поэтому дальнейшая работа здесь не требуется.

<b>2) Экспорт сервиса и client job из Studio</b>

Чтобы развернуть SOAP-сервис DoubleIt и client job, нам нужно экспортировать их из Studio. Щелкните правой кнопкой мыши по сервису DoubleIt в левом меню и сначала выберите «ESB Runtime Options», отметив галочкой «Log Messages», чтобы видеть входные/выходные сообщения сервиса в журнале. Затем снова щелкните правой кнопкой мыши DoubleIt, выберите «Export Service» и сохраните полученный файл .kar.

Прежде чем экспортировать client job, нам нужно внести изменение. Порт по умолчанию, который Studio использовал для службы SOAP DoubleIt (8090), отличается от порта Karaf (8040). Нажмите «tESBConsumer» и измените номер порта в адресе на «8040». Затем, после сохранения, щелкните правой кнопкой мыши на задании DoubleItClient и выберите «Build job». В разделе «Build Type» выберите «OSGI bundle for ESB» и нажмите «Finish» для экспорта задания

![Build Job Type](https://drive.google.com/uc?export=view&id=169i8TqeALqLC7z5daI57s7u3iNGxewXb){:height="150px"}

<b>3) Deploy сервиса и client job в runtime container</b>

Скопируйте служебный файл .kar в Runtime_ESBSE/container/deploy. Это автоматически развернет службу в Karaf (что можно проверить, запустив la в консоли - вы должны увидеть сервис последним в списке). Затем также скопируйте клиентский jar в каталог deploy. Ответ будет выведен в окне консоли (благодаря компоненту tLogRow), а полное сообщение можно увидеть в журналах сервера (log/tesb.log)

![Container console](https://drive.google.com/uc?export=view&id=1JfEHqnLmjexoSTc_Mtzi28wegjdPxRu_){:height="150px"}

<b>Источник</b>

[Securing web services using Talend's Open Studio for ESB - part II][source-article]

[source-article]: https://coheigea.blogspot.com/2018/05/securing-web-services-using-talends_21.html
