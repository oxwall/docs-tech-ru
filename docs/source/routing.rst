.. _routing-label:

Маршрутизация и контроллеры
===========================

Маршрутизация помогает определять и доставлять все входящие запросы от клиентов в контроллеры, которые в свою очередь обрабатывают и возвращают
результат назад  клиенту.

Регистрация маршрутов
---------------------

В платформе **Oxwall** есть два способа использования маршрутизации для того, чтобы запросить или отправить данные на сайт.


Именованные маршруты
++++++++++++++++++++

При использовании именованных маршрутов, мы сами создаем шаблон того как будет выглядеть адрес запроса и какие параметры он будет принимать.
Основным преимуществом использования именованных маршрутов это - создания красивых и понятных адресов, которые хорошо воспринимаются человеком.
Также именованные маршруты хороши для `SEO оптимизации <https://en.wikipedia.org/wiki/Search_engine_optimization>`_. Регистрация маршрутов происходит в файле - **init.php** (подробнее в разделе - :ref:`plugin-structure-label`), ниже пример регистрации нескольких маршрутов:

.. code-block:: php

    <?php

        OW::getRouter()->addRoute(new OW_Route('superplugin_list_index', 'superplugin/', 'SUPERPLUGIN_CTRL_Index', 'viewList'));
        OW::getRouter()->addRoute(new OW_Route('superplugin_item_edit', 'superplugin/edit/:id/', 'SUPERPLUGIN_CTRL_Index', 'edit'));

При регистрации маршрута нам нужно указать:

#. Название.
#. Адрес с параметрами если нужно.
#. Название контроллера.
#. Название метода действия куда придет запрос на обработку.

После того как именованные маршруты зарегистрированы, мы можем их использовать в PHP коде (в контроллере к примеру) или напрямую в коде шаблонов (подробнее в разделе - :ref:`view-components-label`):

**Внутри кода шаблона страницы:**

.. code-block:: html

    <html>
        <body>
            <a href="{url_for_route for='superplugin_list_index'}">View list</a>
            <br />
            <a href="{url_for_route for="superplugin_item_edit:[id=>`$item->id`]"}">Edit item</a>
        </body>
    </html>

**Внутри PHP кода:**

.. code-block:: php

    <?php

        $listUrl = OW::getRouter()->urlForRoute('superplugin_list_index');
        $editUrl = OW::getRouter()->urlForRoute('superplugin_item_edit',  array('id' => $item->id));

Маршруты по умолчанию
+++++++++++++++++++++

Маршруты по умолчанию не нуждаются в инициализации и для того, чтобы их использовать вам нужно лишь указать название контроллера и название действия внутри контроллера.

**Внутри кода шаблона страницы:**

.. code-block:: html

    <html>
        <body>
            <a href="{url_for for="MYSUPERPLUGIN_CTRL_Index:ajaxSaveItem"}">Save</a>
        </body>
    </html>

**Внутри PHP кода:**

.. code-block:: php

    <?php

        $saveUrl = OW::getRouter()->urlFor('MYSUPERPLUGIN_CTRL_Index', 'ajaxSaveItem');

Так как маршруты по умолчанию возвращают не красивые адреса с точки зрения **SEO** их стоит использовать только для внутренних целей (к примеру отправка ajax запросов), для всех остальных случаев
рекомендуется использовать именованные адреса.

Контроллеры
-----------

После того как маршрут установлен то управление передается коду контроллера которому нужно обработать входящий запрос и вернуть данные в подходящем формате. Ниже
приведен пример кода контроллера с описанием методов которые были использованы выше при описании маршрутизации.

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_CTRL_Index extends OW_ActionController
        {
            /**
             * View list
             */
            public function viewList()
            {
                // do some logic
                //...

                // init view variables
                $this->assign('foo', 'bar');
                // ...
            }

            /**
             * Edit item
             *
             * @param array $params
             */
            public function edit($params = array())
            {
                $itemId = isset($params['id']) ? (int) $params['id'] : 0;

                // do some logic
                //...

                // init view variables
                $this->assign('foo', 'bar');
                // ...
            }

            /**
             * Ajax save item
             */
            public function ajaxSaveItem()
            {
               if ( OW::getRequest()->isAjax() )
               {
                    // do some logic
                    //...

                    // show result
                    echo json_encode(
                        'foo' => 'bar'
                    );
               }

               exit;
            }
        }