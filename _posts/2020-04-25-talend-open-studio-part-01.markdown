---
layout: post
title: "Talend’s Open Studio for ESB - часть 1"
date: 2020-04-25 00:17:57 +0300
categories: talend
tags: [talend, wsdl, service, soap]
anons: "Проектирование и тестирование простого веб-сервиса SOAP в TOS ESB"
---
`Talend’s Open Studio for ESB` является свободно распространяемым ПО, которое включает в себя eclipse-based студию для проектирования и тестирования веб-сервисов, а также runtime среду на основе `Apache Karaf`, которую можно использовать для развертывания веб-сервисов, построенных в студии. В этой части рассмотрим, как спроектировать и протестировать простой веб-сервис `SOAP` в Studio.

<b>1) Загрузите и запустите Studio и создайте простой веб-сервис SOAP</b>

Сначала загрузите и распакуйте Talend's Open Studio для ESB. Каталог «Runtime_ESBSE» содержит runtime контейнер, а каталог «Studio» - Open Studio for ESB. Запустите студию и создайте новый проект.

Мы создадим SOAP веб-сервис «double-it» с использованием Studio. Щелкните правой кнопкой мыши «Services» в левом меню и выберите «Create service». В поле Name - «DoubleIt» и нажмите «Далее», чтобы создать новый WSDL. WSDL можно увидеть, если дважды нажать на созданном сервисе. Внесем несколько изменений в сервис по умолчанию.

Нажмите «DoubleItPortType» и измените имя операции с «DoubleItOperation» на «DoubleIt». Затем нажмите на стрелку вправо рядом с «DoubleItRequest» и измените тип запроса с «string» на «int». Сделайте то же самое для типа ответа. Теперь сохраните сервис, и мы готовы перейти к следующему шагу.

![SOAP сервис в Talend Studio](https://drive.google.com/uc?export=view&id=1KZJ01_l--YInJXbLHavL3rQFPrdyNpTd){:height="150px"}

<b>2) Запустите сервис «DoubleIt», который мы создали</b>

После того как мы разработали сервис, нам нужно назначить сервису задание (job). Щелкните правой кнопкой мыши созданный нами сервис «DoubleIt» («Services / DoubleIt/ DoubleItPortType / DoubleIt» в меню слева), выберите «Assign job» и нажмите «Далее», чтобы создать new job. Теперь в разделе «Job Designs» в левом меню мы видим новое задание, а главном окне добавлены компоненты tESBProviderRequest tESBProviderResponse.

Перетащите компонент tESBProviderResponse в правую часть окна. Теперь нам нужно продумать сервисную логику. В нашем сервисе мы хотим взять входное число, умножить его на два и вернуть. «tXMLMap» компонент, доступный в Studio, который позволяет нам отображать XML.

Найдите компонент «tXMLMap» в палитре справа (в разделе «XML») и перетащите его в главное окно между двумя существующими компонентами. Теперь щелкните правой кнопкой мыши на tESBProviderRequest и выберите «Row» и «Main» и соедините с компонентом tXMLMap. Сделайте то же самое из tXMLMap в tESBProviderResponse, задав имя «Response» для вывода.

![Job Design в Talend Studio](https://drive.google.com/uc?export=view&id=1c6LobuOwqUAbu0YbiWxM9Eb1lG5x7rCk){:height="150px"}

Далее нужно настроить компонент «tXMLMap» для реализации логики сопоставления. Дважды щелкните «tXMLMap». Входной запрос доступен с левой стороны payload/root. Мы хотим отобразить payload запроса на payload ответа. Удерживая левую кнопку мыши слева payload/root запроса, переместите мышь вправо и отпустите на payload/root ответа, выбрав «Add linker to target node».

Прежде чем мы попытаемся реализовать логику умножить на 2, нам нужно изменить тип payload с «String» на «int». Перейдите на вкладку «Tree schema editor» в нижней части экрана и измените типы payload запроса и ответа на «int». Вернитесь на вкладку Response, нажмите «[row1.payload:/root]» и измените его на «2 * [row1.payload:/root]».

Теперь нам нужно изменить элементы запроса/ответа, чтобы они соответствовали WSDL. Щелкните правой кнопкой мыши «root» в левом столбце запроса и переименуйте его в «ns2: DoubleItRequest». Когда это будет сделано, снова щелкните правой кнопкой мыши по этому элементу и выберите «Установить пространство имен» с пространством имен «http://www.talend.org/service/» и префиксом «ns2». Аналогично, с правой стороны переименуйте «root» в «ns2: DoubleItResponse» и установите пространство имен таким же образом, как и для запроса. Теперь нажмите «Apply» и «Ok».

![Mapping tXMLMap в Talend Studio](https://drive.google.com/uc?export=view&id=11bLKrSy0WiaQxXa3x41AFFzMd72YKGaD){:height="300px"}

Теперь нажмите на вкладку «Run» в нижней части экрана и запустите сервис. Служба должна быть доступна по адресу «http://localhost:8090/services/DoubleIt».

<b>3) Создать клиента для сервиса "DoubleIt"</b>

Помимо разработки, создания и тестирования веб-сервисов, Talend Open Studio for ESB также может использоваться для создания клиентов для этих веб-сервисов. Для этого нам нужна new job. Щелкните правой кнопкой мыши «Job Designs» в меню слева и создайте новое задание. Перетащите компонент «tESBConsumer» на главный экран, а также два компонента «tLogRow». Щелкните правой кнопкой мыши на «tESBConsumer» и выберите «Row/Response» и перетащите стрелку на первый компонент tLogRow. Кроме того, выберите «Row/Fault» и перетащите стрелку на второй компонент tLogRow.
 
Щелкните левой кнопкой мыши на «tESBConsumer» и укажите «http://localhost:8090/services/DoubleIt?wsdl» в качестве местоположения WSDL (WSDL нашего сервиса доступен по этому адресу). Затем нажмите кнопку «Reload» с правой стороны и нажмите «Finish». Далее нам нужно реализовать клиентскую логику, а именно - указать число. Перетащите компоненты «tFixedFlowInput» и «tXMLMap» на экран. Сопоставьте «Row/Main» из «tFixedFlowInput» с «tXMLMap», а «Row» из «tXMLMap» с «tESBConsumer» с новым выходным именем «Request».

![Job Design в Talend Studio](https://drive.google.com/uc?export=view&id=1qfvFqx47MDTboZQZctDo0zxf33eS9UoX){:height="150px"}

Теперь нажмите на «tFixedFlowInput» и «Edit Schema». Добавьте новый столбец типа «int» с именем «numberToDouble». Вернувшись на экран компонента для «tFixedFlowInput», выберите «Use inline table» и введите число (например, 200). Теперь нажмите «tXMLMap», чтобы настроить mapping. Перетащите «numberToDouble» с левой стороны к «root» с правой стороны, выбрав «Add linker to target node». Щелкните правой кнопкой мыши «root» с правой стороны и переименуйте его в «ns2:DoubleItRequest», и «Set a Namespace» с пространством имен «http://www.talend.org/service/» и префиксом «ns2».

Нажмите кнопку ОК, сохраните задание и запустите его на вкладке «Run». В окне консоли мы должны увидеть ответ от сервиса, сообщающий нам, что 200 удвоилось - это «400». Job также обновляется, чтобы вы могли видеть поток вместе с пропускной способностью


![Детальная статистика](https://drive.google.com/uc?export=view&id=1C3KpOUrCb4RR8kGjnRJLSRYMfXp_NedL){:height="150px"}


<b>Источник:</b>

[Securing web services using Talend's Open Studio for ESB - part I][source-article]

[source-article]: https://coheigea.blogspot.com/2018/05/securing-web-services-using-talends.html
