.. _authorization-label:

Авторизация
===========

В платформе **Oxwall** авторизация реализована на основе ролей пользователей.
Каждая роль в платформе разрешает или запрещает пользователям определенный набор действий.


Регистрация действий
--------------------

Регистрация списка действий плагина производится в файле **install.php** (подробнее в разделе - :ref:`plugin-structure-label`).
Ниже приведен пример регистрации нескольких действий:

.. code-block:: php

    <?php

        $authorization = OW::getAuthorization();
        $authorization->addGroup('superplugin');
        $authorization->addAction('superplugin', 'view_items');
        $authorization->addAction('superplugin', 'edit_items');
        $authorization->addAction('superplugin', 'delete_items');

В приведенном примере при установке плагина мы создаем новую группу авторизации - **superplugin**.
Далее к созданной группе добавляем список действий которые в дальнейшем будем использовать к примеру
для проверки прав пользователей просматривать, редактировать, удалять контент итд.

Далее нам нужно добавить описание добавленных действий для того, чтобы пользователи и администратор сайта смогли понять за, что отвечают эти действия.
Для этого нам нужно подписаться на системное событие в файле - **classes/event_handler.php** (подробнее в разделе - :ref:`system-vents-label`) и вернуть описания действий:

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_CLASS_EventHandler
        {
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

Проверка прав пользователей
---------------------------

Для того, чтобы проверить разрешено ли текущему пользователю выполнить те или иные действия нужно выполнить код:

.. code-block:: php

    <?php

        // is view items allowed for current user ?
        $isViewAllowed = OW::getUser()->isAuthorized('superplugin', 'view_items');

        if ( !isViewAllowed )
        {
            // get error message
            $errorMessage = BOL_AuthorizationService::getInstance()->getActionStatus('superplugin', 'view_items');
            throw new AuthorizationException($errorMessage['msg']);
        }