---
layout: post
title: "Создание Github Pages site c Jekyll - часть 2"
date: 2020-08-21 00:17:57 +0300
categories: other
tags: [github, jekyll, win10]
author: mAsYA
anons: "Разворачивание Jekyll с помощью Ansible"
---
<b>1) Разворачивание [Jekyll c помощью Bundler][jekyll-with-bundler]</b> 

Bundler является отличным инструментом для работы с Jekyll. Поскольку он отслеживает зависимости для каждого проекта, и особенно полезен, если требуется запускать разные версии Jekyll в разных проектах.

Кроме того, поскольку Bundler может устанавливать зависимости в папку проекта, это помогает избежать проблем с разрешениями. Обычный способ использования Jekyll - установить Jekyll в системный каталог установки по умолчанию gems, а затем запустить jekyll new. Здесь создается новый проект Jekyll с помощью Bundler без установки gems вне каталога проекта.

<b>2) Подготовка машины Vagrant</b>

{% highlight shell_session %}
cd c:\_VirtualBox\ansible-jekyll
vagrant.exe init ubuntu/bionic64
# при первом запуске загрузится образ виртуальной машины
vagrant.exe up
{% endhighlight %}

Содержимое [Vagrantfile][vagrantfile-github] - настроим Vagrant так, чтобы запросы к портам 8080 и 4001 на локальной машине перенаправлялись портам 80 и 4000 на машине Vagrant. 

{% highlight shell %}
# создание SSH-сеанса для проверки
ssh vagrant@192.168.33.10 -i ./.vagrant/machines/default/virtualbox/private_key
{% endhighlight %}

<b>3) Выполнение сценария [jekyll.yml][jekyll-yml-github]</b>

<b>4) Запуск Jekyll сервера и проверка</b>

{% highlight shell %}
cd jekyll/jekyll_example
bundle config set path 'vendor/bundle'
bundle exec jekyll serve --port=4000 --host=0.0.0.0
{% endhighlight %}

![jekyll site in browser](https://drive.google.com/uc?export=view&id=1mcFl4cs2jxbtX1v-Zc07J2uCGfLEYE57){:height="150px"}

[jekyll-yml-github]: https://github.com/masya/ansible-simple-projects/blob/master/jekyll-with-bundle/jekyll.yml
[vagrantfile-github]: https://github.com/masya/ansible-simple-projects/blob/master/jekyll-with-bundle/Vagrantfile
[jekyll-with-bundler]: https://jekyllrb.com/tutorials/using-jekyll-with-bundler
