---
layout: post
title: "Talend’s Open Studio for ESB - часть 2"
date: 2020-04-30 00:00:57 +0300Vlkc3DtrhPcRd
categories: talend
tags: [talend, wsdl, service, soap]
anons: "Проектирование веб-сервиса SOAP в TOS ESB"
author: mAsYA
---

<b>1) Создание SOAP веб-сервиса</b>

В дереве Repository щелкните правой кнопкой на Services - определить веб-службу можно, или создав новый файл WSDL, или импортировав существующий файл WSDL.

Выберете создать новый сервис

![SOAP сервис в Talend Studio](https://drive.google.com/uc?export=view&id=1pjbX6JqPrmd-OLMzeMxIKi3tzJ-ZKg-Y){:height="150px"}

<table>
<tr><td>Services</td><td>Name</td><td>UserService</td></tr>
<tr><td>Services</td><td>Target namespace</td><td>http://services.talend.org/UserService</td></tr>
<tr><td>Services</td><td>Prefix</td><td>tns</td></tr>
<tr><td>service</td><td>Name</td><td>UserServiceProvider</td></tr>
<tr><td>port</td><td>Address</td><td>http://localhost:8090/services/UserService</td></tr>
<tr><td>portType</td><td>Name</td><td>UserService</td></tr>
<tr><td>operation</td><td>Name</td><td>getUserInfo</td></tr>
</table>

Для просмотра элемента UserSearch наведите на стрелку вправо рядом

![Элемент UserSearch](https://drive.google.com/uc?export=view&id=1y3_zEU7Y9wdFFu4H_4AVlkc3DtrhPcRd){:height="150px"}

Для просмотра элемента UserDetail наведите на стрелку вправо рядом

![Элемент UserDetail](https://drive.google.com/uc?export=view&id=1aMVwbT7LBMNJFYsMPIeaiVDyp9_2h5x5){:height="150px"}

Описание схемы для web-сервиса

![Описание схемы](https://drive.google.com/uc?export=view&id=1PwGAE-FSk1qdqN_rRevLY8-GobCum61l){:height="150px"}

Сохраните файл WSDL. Мы будем использовать его для создания веб-сервиса.

Созданный веб-сервис отображается с восклицательным знаком под узлом «Services» представления «Repository». Значок восклицательного знака означает, что этот определенный веб-сервис не используется.

Щелкните правой кнопкой мыши на созданной службе и выберите Импортировать схему WSDL. Это позволит импортировать метаданные WSDL из службы в репозиторий в раздел Metadata>File xml для последующего использования другими компонентами.

<b>2) Создание поставщика данных (Data Service Provider)</b>

Поставщик данных использует компоненты tESBProviderRequest и tESBProviderResponse для создания доступа к web-службе и компонент tXMLMap для объединения данных о пользователях, предоставленных компонентом tFixedFlowInput, в основной поток запрос-ответ для публикации. 

 - щелкните правой кнопкой мыши на операции getUserInfo веб-службы UserService, выберете в контекстном меню «Назначить задание». Откроется мастер назначения задания - выберите «Создать новое задание».

- в созданном задании уже добавлены и настроены tESBProviderRequest и tESBProviderResponse. tESBProviderRequest отправит запрос указанному веб-сервису, а tESBProviderResponse отправит ответ, соответствующий запросу. Добавьте компоненты tXmlMap tFixedFlowInput и связи: 

![Job Design Talend Studio](https://drive.google.com/uc?export=view&id=1hWytW3v0Lvj0DTOtBuRyybc8lMK3Q5FN){:height="150px"}

Конфигурация tXMLMap:

![tXmlMap component](https://drive.google.com/uc?export=view&id=1pj3axVkcEZv6hiXuY1ySzIm-BcKKcyJA){:height="250px"}

