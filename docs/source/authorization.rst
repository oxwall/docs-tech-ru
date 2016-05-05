.. _authorization-label:

Авторизация
===========

В платформе **Oxwall** авторизация реализована на основе ролей пользователей.
Каждая роль в платформе разрешает или запрещает пользователям определенный набор действий.


Регистрация действий
--------------------

Регистрация списка действий плагина производится в файле **install.php** (подробнее в разделе - :ref:`plugin-structure-label`).
Ниже приведен пример регистрации нескольких действий:

**install.php**

.. code-block:: php

    <?php

        $authorization = OW::getAuthorization();

        $authorization->addGroup('superplugin');

        $authorization->addAction('superplugin', 'view_items');
        $authorization->addAction('superplugin', 'edit_items');
        $authorization->addAction('superplugin', 'delete_items');

В приведенном примере при установке плагина мы создаем новую группу авторизации - **superplugin**
(название группы авторизации должно совпадать с ключем плагина описанным в plugin.xml).
Далее к созданной группе добавляем список действий которые в дальнейшем будем использовать к примеру
для проверки прав пользователей просматривать, редактировать, удалять контент итд.

После регистрации списка действий нужно
добавить для них описание для того, чтобы пользователи и администратор сайта смогли понять за, что отвечают эти действия.
Для этого нам нужно подписаться на системное событие в файле - **classes/event_handler.php** (подробнее в разделе - :ref:`system-vents-label`) и вернуть описания этих действий:

**classes/event_handler.php**

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_CLASS_EventHandler
        {
            /**
             * Class instance
             *
             * @var MYSUPERPLUGIN_CLASS_EventHandler
             */
            private static $classInstance;

            /**
             * Class constructor
             */
            private function __construct()
            {}

            /**
             * Get instance
             *
             * @return MYSUPERPLUGIN_CLASS_EventHandler
             */
            public static function getInstance()
            {
                if ( self::$classInstance === null )
                {
                    self::$classInstance = new self();
                }

                return self::$classInstance;
            }

            /**
             * Generic init (should be started for all contexts such as: mobile, desktop, cli and api)
             *
             * @return void
             */
            public function genericInit()
            {
                $em = OW::getEventManager();

                // init auth labels
                $em->bind('admin.add_auth_labels', array($this, 'addAuthLabels'));
            }

             /**
              * Add auth labels
              *
              * @param BASE_CLASS_EventCollector $event
              * @return void
              */
             public function addAuthLabels(BASE_CLASS_EventCollector $event)
             {
                 $event->add(
                     array(
                         'superplugin' => array(
                             'label' => OW::getLanguage()->text('superplugin', 'auth_group_label'),
                             'actions' => array(
                                 'view_items' => OW::getLanguage()->text('superplugin', 'auth_action_label_view_items'),
                                 'edit_items' => OW::getLanguage()->text('superplugin', 'auth_action_label_edit_items'),
                                 'delete_items' => OW::getLanguage()->text('superplugin', 'auth_action_label_delete_items')
                             )
                         )
                     )
                 );
             }
        }

В файле **init.php**  который запускается при каждом запросе от клиента, запускаем метод который подписывается на системные события:

**init.php**

.. code-block:: php

    <?php

        MYSUPERPLUGIN_CLASS_EventHandler::getInstance()->genericInit();

Проверка прав пользователей
---------------------------

Для того, чтобы проверить разрешено ли текущему пользователю выполнить те или иные действия нужно выполнить код:

.. code-block:: php

    <?php

        // is view items allowed for current user ?
        $isViewAllowed = OW::getUser()->isAuthorized('superplugin', 'view_items');

        if ( !$isViewAllowed )
        {
            // get error message
            $errorMessage = BOL_AuthorizationService::getInstance()->getActionStatus('superplugin', 'view_items');
            throw new AuthorizationException($errorMessage['msg']);
        }

Модераторы
----------

Модераторы эта группа пользователей которых назначает администратор сайта в админ панели. Администратор сайта может
разрешить управлять одним и более плагином, что в свою очередь значит, что модератору
будут разрешены любые действия в плагине по управлению контентом пользователей.


Пример кода который проверяет является ли текущий пользователь модератором:

.. code-block:: php

    <?php

        // is current user a moderator of the "superplugin" ?
        $isModerator = OW::getUser()->isAuthorized('superplugin');

        // a person who allowed to do any actions in the "superplugin" auth group
        if ( $isContentOwner || $isModerator )
        {
            // do some logic
        }
