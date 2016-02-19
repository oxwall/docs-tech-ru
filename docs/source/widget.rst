.. _widget-label:

Виджеты
=======

`Виджеты <https://en.wikipedia.org/wiki/Web_widget>`_ - это независимые элементы на страницах сайта, которые легко можно удалить, добавить из кода плагинов.
С помощью виджетов можно решать различные задачи по отображению контента, к примеру отобразить список пользователей на
странице или список последних фотографий на странице профайла, организовать поиск итд.

Создание виджета
----------------

Для начала нам нужно создать пустой файл виджета  внутри директории плагина, в под директории **components** (подробнее в разделе - ":ref:`plugin-structure-label`"). Затем добавляем в него PHP код:

.. code-block:: php

    <?php

    class MYSUPERPLUGIN_CMP_ExampleWidget extends BASE_CLASS_Widget
    {
        /**
         * Default repeat greeting count
         */
        const DEFAULT_REPEAT_GREETING_COUNT = 10;

        /**
         * Repeat greeting count
         *
         * @var integer
         */
        protected $repeatGreetingCount;

        /**
         * Repeat greeting
         *
         * @var boolean
         */
        protected $repeatGreeting;

        /**
         * Class constructor
         *
         * @param BASE_CLASS_WidgetParameter $paramObj
         */
        public function __construct( BASE_CLASS_WidgetParameter $paramObj )
        {
            parent::__construct();

            // process custom widget’s settings
            $this->repeatGreeting = !empty($paramObj->customParamList['repeatGreeting']);

            $this->repeatGreetingCount = isset($paramObj->customParamList['repeatGreetingCount'])
                ? (int) $paramObj->customParamList['repeatGreetingCount']
                : self::DEFAULT_REPEAT_GREETING_COUNT;

        }

        /**
         * Init view variables
         *
         * @return void
         */
        public function onBeforeRender()
        {
            parent::onBeforeRender();

            // assign view variables
            $this->assign('greetingMesasge', 'Hello World');
            $this->assign('date', date('Y-m-d'));
            $this->assign('repeatGreetingCount', $this->repeatGreetingCount);
            $this->assign('repeatGreeting', $this->repeatGreeting);
        }

        /**
         * Get widget auth access
         *
         * @return string
         */
        public static function getAccess()
        {
            return self::ACCESS_MEMBER;
        }

        /**
         * Get custom widget setting list
         *
         * @return array
         */
        public static function getSettingList()
        {
            $settingList = array();

            $settingList['repeatGreetingCount'] = array(
                'presentation' => self::PRESENTATION_NUMBER,
                'label' => OW::getLanguage()->text('mysuperplugin', 'widget_example_count_repeat_greeting'),
                'value' => 3
            );

            $settingList['repeatGreeting'] = array(
                'presentation' => self::PRESENTATION_CHECKBOX,
                'label' => OW::getLanguage()->text('mysuperplugin', 'widget_example_repeat_greeting'),
                'value' => true
            );

            return $settingList;
        }

        /**
         * Get standard widget settings
         *
         * @return array
         */
        public static function getStandardSettingValueList()
        {
            return array(
                self::SETTING_TITLE => OW::getLanguage()->text('mysuperplugin', 'example_widget_title'),
                self::SETTING_ICON => self::ICON_INFO,
                self::SETTING_SHOW_TITLE => true,
                self::SETTING_WRAP_IN_BOX => true
            );
        }
    }

Рассмотрим класс более подробно:

#. Все классы виджетов являются потомками класса - **BASE_CLASS_Widget**, который в свою очередь является потомком класса  **OW_Component**, который используется как вспомогательный класс для отображения страниц (:ref:`view-components-label`).
#. В конструктор класса передаются экземпляр класса **BASE_CLASS_WidgetParameter** который содержит в себе настройки виджета:
    a. **$customParamList** - ассоциативный массив пользовательских настроек добавленных администратором (их рассмотрим ниже).
    b. **$additionalParamList** - ассоциативный массив дополнительных параметров, к примеру если виджет расположен на странице профиля то в этот массив будет передан ID просматриваемого пользователя.
#. В методе **onBeforeRender** мы подготавливаем данные которые будут переданы в последствии в файл представления (view).
#. В методе **getAccess** мы выставляем уровень доступа к виджету, т.е кому он будет отображен. Следует отметить, что есть всего три  уровня доступа к виджету:  "гость", "пользователь" и "все" (эти константы можно найти в родительском классе - **BASE_CLASS_Widget**)
#. В методе **getSettingList**  мы декларативно описываем список пользовательских настроек виджета, к примеру в нашем примере мы даем возможность админу вводить какое количество раз вывести текст приветствия в виджете.
#. В методе **getStandardSettingValueList** мы определяем настройки внешнего вида виджета (заголовок, иконка итд.)

Для того, чтобы закончить виджет осталось только создать файл представления (view), для этого создаём пустой **html**
файл в директории - **"views/components/example_widget.html"**. Добавляем в него контент отображения:

.. code-block:: html

    <ul>
        {if $repeatGreeting}
            {for $foo=1 to $repeatGreetingCount}
                <li>{$greetingMesasge} ({$date})</li>
            {/for}
        {else}
            <li>{$greetingMesasge} ({$date})</li>
        {/if}
    </ul>

В файлах представления используется синтаксис шаблонизатора - `Smarty <http://www.smarty.net/>`_


Активация виджета
-----------------

Для того, чтобы виджет был зарегистрирован системой, а в последствии запущен нужно написать его инициализацию. Так как виджеты должны работать
только из активных плагинов, мы делаем инициализацию только в файле - **activate.php** (:ref:`plugin-structure-label`)

.. code-block:: php

    <?php

        $widgetService = BOL_ComponentAdminService::getInstance();

        // add example widget on profile page
        $widget = $widgetService->addWidget('MYSUPERPLUGIN_CMP_ExampleWidget', false);
        $widgetPlace = $widgetService->addWidgetToPlace($widget, BOL_ComponentService::PLACE_PROFILE);
        $widgetService->addWidgetToPosition($widgetPlace, BOL_ComponentService::SECTION_LEFT);

Данный пример кода добавялет виджет на страницу просмотра профайлов в левую часть страницы.

Деактивация виджета
-------------------

Как только виджет удаляется или становится не активным, то виджет нужно деактивировать:

.. code-block:: php

    <?php

        BOL_ComponentAdminService::getInstance()->deleteWidget('MYSUPERPLUGIN_CMP_ExampleWidget');
