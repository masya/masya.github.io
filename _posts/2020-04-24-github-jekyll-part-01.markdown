---
layout: post
title: "Создание Github Pages site c Jekyll - часть 1"
date: 2020-04-24 00:17:57 +0300
categories: other
tags: [github, jekyll, win10]
author: mAsYA 
anons: "Подготовка десктопа на Windows 10"
---
1) Для работы требуется предварительно установить [Git][git-load] [Jekyll][jekyll-load] [Bundler][bundler-load]

2) Подготовка десктопа на `Windows 10`

[Установка подсистемы Windows для Linux][subsystem-win].

- Запустите PowerShell с правами администратора

{% highlight shell %}
PS > Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
{% endhighlight %}

- Скачайте, установите, инициализируйте дистрибутив Ubuntu

- Запустите Ubuntu Bash

{% highlight shell %}
# обновить репозиторий и пакеты
sudo apt-get update -y && sudo apt-get upgrade -y
# для установки Ruby будем использовать репозиторий BrightBox
sudo apt-add-repository ppa:brightbox/ruby-ng
sudo apt-get update
# установить пакеты
sudo apt-get install ruby2.6 ruby2.6-dev build-essential zlib1g-dev dh-autoreconf
sudo apt install ruby-bundler
# проверка RubyGems окружения
gem environment
# добавить переменные и перечитать
# задать папку для установки gem
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
{% endhighlight %}
[git-load]: https://git-scm.com/downloads
[jekyll-load]: https://jekyllrb.com/docs/installation
[bundler-load]: https://bundler.io
[subsystem-win]: https://msdn.microsoft.com/en-us/commandline/wsl/install_guide
