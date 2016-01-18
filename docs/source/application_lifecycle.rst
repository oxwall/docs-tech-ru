.. _application_lifecycle-label:

Жизненый цикл приложения
========================

Жизненный цикл приложения начинается с файла -  **index.php**, который является единственной точкой входа, другими словами все запросы поступающие от клиентов
направляются прямо в этот файл (реализация паттерна - `Front controller <https://en.wikipedia.org/wiki/Front_controller>`_).
Такое поведение задано с помощью директив файла **.htaccess** который находится в корне приложения и имеет следующие настройки:

.. code-block:: .htaccess

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_URI} !^/index\.php
    RewriteCond %{REQUEST_URI} !/ow_updates/index\.php
    RewriteCond %{REQUEST_URI} !/ow_updates/
    RewriteCond %{REQUEST_URI} !/ow_cron/run\.php
    RewriteCond %{REQUEST_URI} !/e500\.php
    RewriteCond %{REQUEST_URI} !/captcha\.php
    #RewriteCond %{REQUEST_URI} (/|\.php|\.html|\.htm|\.xml|\.feed|robots\.txt|\.raw|/[^.]*)$  [NC]
    RewriteCond %{REQUEST_FILENAME} (/|\.php|\.html|\.htm|\.xml|\.feed|robots\.txt|\.raw|/[^.]*)$  [NC]
    RewriteRule (.*) index.php

Инициализация программного окружения
------------------------------------

После того как запрос был передан в файл **index.php** в нем начинается серия инициализаций программного окружения.

#. Инициализация констант.
#. Инициализация автолоадеров.
#. Инициализация профайлера.
#. Инициализация логирования.
#. Инициализация сессий.

Инициализация приложения
------------------------

На этапе инициализации приложения происходит ряд важных установок и вызовов на которые можно повлиять из плагинов ниже перечислены наиболее важные из них:


Определение контекста приложения.
+++++++++++++++++++++++++++++++++

В Oxwall  существует 3 контекста: мобильный, десктопный и api. После определения контекста, приложение просто регистрирует пекеджпойинтеры (неймспейсы),
т.е по сути подключает файлы инициализации из определенного контекста. На данный момент из кода плагинов не возможно программно поменять контекст.


Инициализация плагинов.
+++++++++++++++++++++++

На данном этапе у каждого установленного и активного плагина вызывается файл init.php (:ref:`plugin-structure-label`) который регистрирует ресурсы плагина
(маршрутизация контроллеров, системные события итд). После инициализации плагинов срабатывает системное событие которое оповещает приложение о том, что инициализация плагинов закончена:

.. code-block:: php

    $event = new OW_Event(OW_EventManager::ON_PLUGINS_INIT);
    OW::getEventManager()->trigger($event);

Это событие можно использовать в свои целях, к примеру после загрузки всех плагинов можно выполнить логику своего плагина.

.. code-block:: php

    class MYSUPERPLUGIN_CLASS_EventHandler
    {
        public function genericInit()
        {
          OW::getEventManager()->bind(OW_EventManager::ON_PLUGINS_INIT, array($this, 'afterPluginsInit'));
        }

        public function afterPluginsInit')
        {
            // do something...
        }
    }

Инициализация темы.
+++++++++++++++++++

Далее приложение пытается определить тему по умолчанию (тема по умолчанию задается в настройках админ панели) и активировать ее,
взяв название темы из системных настроек приложения. Однако плагины могут повлиять на этот процесс, для этого нужно подписаться
на системное событие и передать в него название другой темы. Пример системного события:

.. code-block:: php

    $activeThemeName = OW::getEventManager()->call('base.get_active_theme_name');
    $activeThemeName = $activeThemeName ? $activeThemeName : OW::getConfig()->getValue('base', 'selectedTheme');

Меняем дефолтную тему

.. code-block:: php

    class MYSUPERPLUGIN_CLASS_EventHandler
    {
        public function genericInit()
        {
          OW::getEventManager()->bind(‘base.get_active_theme_name’, array($this, 'changeTheme'));
        }

        public function changeTheme()
        {
            return ‘new_theme_name’;
        }
    }

Инициализация объекта ответа.
+++++++++++++++++++++++++++++

В самом конце инициализации приложения, создается объект ответа "Response", который и будет в последствии возвращен клиенту.
В этом объекте содержаться заголовки ответа, которые можно менять по ходу жизненного цикла приложения.
А также объект документ "Document", который содержит непосредственно сгенерированный html код.
Сторонние плагины имеют возможность изменять содержимое объекта документ, для этого существует ряд системных событий

1. Событие срабатывает, перед отправкой сгенерированного контента клиенту

.. code-block:: php

    $event = new OW_Event(OW_EventManager::ON_BEFORE_DOCUMENT_RENDER);
    OW::getEventManager()->trigger($event);

2. Событие срабатывает, после отправки сгенерированного контента клиенту

.. code-block:: php

    $event = new OW_Event(OW_EventManager::ON_AFTER_DOCUMENT_RENDER);
    OW::getEventManager()->trigger($event);

После инициализации приложения срабатывает событие:

.. code-block:: php

    $event = new OW_Event(OW_EventManager::ON_APPLICATION_INIT);
    OW::getEventManager()->trigger($event);


Инициализация контроллера
+++++++++++++++++++++++++

После инициализации приложения, **oxwall** пытается определить название контроллера, и действия которому нужно передать управление.
Для этого сравнивается URL клиента и список зарегистрированных маршрутов. Если маршрут не найден то генерируется исключительная ситуация и идет
перенаправление на контроллер отображающий 404 ошибку.