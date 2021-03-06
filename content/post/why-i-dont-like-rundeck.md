---
title: Почему мне не нравится rundeck
author: ctrlok
layout: post
date: 2013-10-16
dsq_thread_id:
  - 1862775332
categories:
  - деплой
tags:
  - chef
  - rundeck

---
В новой компании в качестве менеджера конфигураций используют chef.  Но так как всем известно что деплоить любым декларативным инструментом сложновато &#8212; решили посмотреть на rundeck &#8212; инструмент предназанченный конкретно для деплоя. Он подключается к машинам по ssh и выполняет на них какие-то комманды. Я потратил какое-то время и постарался разобраться в том что это за инструмент.

Из плюсов:

  * **есть API**,  которое очень ничего так. Новые хосты можно добавлять генерируя YAML файлы и скармливая их через пост запросы серверу.  Можно удаленно запускать задачи, смотреть текущие задачи, смотреть список проектов. Экспортировать и импортировать задачи. Подробнее [тут.][1]
  * **Неплохой веб-интерфейс**. И он на самом деле не так уж и плох. Можно создать отдельные проекты, в проектах создать задачи.  На задачи можно подкрутить опции, которые можно выбрать перед выполнением задачи (например определение переменной) и использовать по время выполнения задачи.
  * **Сами задачи тоже неплохи**. Задачи деляться на шаги. Можно выполнять часть задачи локально, часть глобально, запустить скрипт. Также есть возможность работать как в режиме orchestration, когда переход к следующему шагу осуществляется только после завершения предыдущего на всех нодах, так и в режиме &#171;сначала нода 1, потом нода 2 и т.д.&#187;
  * **синхронизация нод и групп** из chef &#8212; собственно та причина по которой мне его предложили. Работает здорово, пусть и есть какие-то проблемы со скоростью обновления.

Теперь минусы:

  * chef-rundeck &#8212; тот демон, который синкает машины из шефа заказал **>750 метров** виртуальной памяти и использует  **120 оперативной**. И продолжает в том же духе. Это как-то многовато для синкалки.
  * Сам процесс, на тестовой машине, которая еще ничем не рулит и имеет в себе три джобы заказал **>1800 метров виртуальной памяти** и использует **630 оперативной**. Оба процесса нещадно свопят. И иногда подключается еще модуль авторизации, который кушает >250 метров оперативы.
  * При **выполнении форка шел**а локально **джава**, на которой написан rundeck, **падает**. Так что если вы хотите, как я, проверить md5 хэш файла в хранилище и сравнить его с локальным файлом, чтобы потом распространить &#8212; не пытайтесь.
  * **Невозможно переести задачу между проектами** кроме как экспортнуть ее в YAML файл локально, удалить и импортировать в другой проект. Вроде не сложно, но напрягает, особенно когда надо сделать почти аналогичный проект.
  * **Нет контроля версий**. Вообще. Все что вы сохранили остается сохраненным. Я накопал решение от фанатов, но оно очень смешное. 
  
{{< highlight bash "style=friendly" >}}
#!/bin/bash
rm -Rf /tmp/rundeckjobs/*
mkdir -p /tmp/rundeckjobs
for p in `ls /var/rundeck/projects` ;do rd-jobs list -f /tmp/rundeckjobs/$p-jobs-export$(date +%Y%m%d).xml -p $p ;done;

service rundeckd stop
tar cvpzf /tmp/rundeck-$(date +%Y%m%d).tar.gz /var/lib/rundeck/data /var/lib/rundeck/logs /etc/rundeck/ /tmp/rundeckjobs/*.xml
service rundeckd start
{{< /highlight >}}

    
Так что я даже не знаю что и сказать. Можно использовать гит, но сам процесс стартует пару минут &#8212; это не очень удобно и больше костыль.</li> 
    
* **Невнятные логи** демона. Так, например у меня он частенько падал с ошибкой Execution failed: 92: [Workflow , Node failures: {имяноды=[]}]
* И да, он **постоянно падает**. Например у меня он любит ложиться от повышения уровня логирования задачи. Конечно, может быть его можно приготовить и нормально, но согласитесь, что в нулевой системе, в которой есть только таск &#171;echo &#171;blablabl&#187; > /tmp/blablabla&#187; ничто не должно помешать серверу выполнить задачу локально на самом себе.  Также, как я писал выше, он нещадно кушает память и падает из-за этого. И так как я сейчас ничего серьезного на нем не кручу мне даже страшно представить что будет когда я начну крутить что-то серьезное.</ul> 
    
    Кроме прочего, если плюшки очевидны, то подводных камней может быть много. Пока этот инструмент мне не нравится, но надо еще его покрутить и посмотреть что он умеет.

 [1]: http://rundeck.org/docs/api/index.html
