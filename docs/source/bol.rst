.. _bol-label:

Слой бизнес логики
==================

В платформе **oxwall** все классы для работы с бизнес логикой принято хранить в директории **bol** внутри плагина (подробнее в разделе - ":ref:`plugin-structure-label`").


База данных
-----------

Для работы с базой данных нужно описать классы **DTO** (`Data Transfer Object <https://en.wikipedia.org/wiki/Data_transfer_object>`_) и **DAO** (`Data Access Object <https://en.wikipedia.org/wiki/Data_access_object>`_) для для каждой таблицы базы данных используемых в плагине.

DTO - Data Transfer Object
++++++++++++++++++++++++++

Класс **DTO** описывает сущность таблицы и используется при добавлении или изменении данных в таблице.
Пример класса описывающий сущность - "**message**":

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_BOL_Message extends OW_Entity
        {
          /**
           *  Title
           *
           * @var string
           */
           public $title;

          /**
           *  Message
           *
           * @var string
           */
           public $message;

          /**
           *  Created timestamp
           *
           * @var integer
           */
           public $createdTimestamp;

           /**
            *  Approved flag
            *
            * @var integer
            */
            public $approved;
        }

Как видно на примере, класс **MYSUPERPLUGIN_BOL_Message** описывает структуру таблицы **message**,
перечисляя в этом классе все поля, а также значения по умолчанию этих полей если нужно.

DAO - Data Access Object
++++++++++++++++++++++++


В классе **DAO** описывается вся бизнес логика работы с базой данных: поиск, создание и удаление записей.
Все классы **DAO** всегда реализуют паттерн - `Singleton Object <https://en.wikipedia.org/wiki/Singleton_pattern>`_. Пример класса **DAO**:

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_BOL_MessageDao extends OW_BaseDao
        {
            /**
             * Approved status
             */
            const APPROVED_STATUS = 1;

            /**
             * Disapproved status
             */
            const DISAPPROVED_STATUS = -1;

            /**
             * Singleton instance.
             *
             * @var MYSUPERPLUGIN_BOL_MessageDao
             */
            private static $classInstance;

            /**
             * Returns an instance of class (singleton pattern implementation).
             *
             * @return MYSUPERPLUGIN_BOL_MessageDao
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
             * Constructor.
             */
            protected function __construct()
            {
                parent::__construct();
            }

            /**
             * Get DTO class name
             *
             * @return string
             */
            public function getDtoClassName()
            {
                return 'MYSUPERPLUGIN_BOL_Message';
            }

            /**
             * Get table name
             *
             * @return string
             */
            public function getTableName()
            {
                return OW_DB_PREFIX . 'mysuperplugin_message';
            }

            /**
             * Delete message
             *
             * @param integer $userId
             * @param integer $recipientId
             * @return void
             */
            public function deleteMessage($userId, $recipientId)
            {
                $example = new OW_Example();
                $example->andFieldEqual('userId', $userId);
                $example->andFieldEqual('recipientId', $recipientId);
                $this->deleteByExample($example);
            }

            /**
             * Find active messages
             *
             * @param integer $limit
             * @return array
             */
            public function findActiveMessages($limit)
            {
                $example = new OW_Example();
                $example->setOrder('`id` ASC');
                $example->andFieldEqual('status', self::APPROVED_STATUS);
                $example->setLimitClause(0, $limit);

                return $this->findListByExample($example);
            }

            /**
             * Find active messages using the raw sql
             *
             * @param integer $limit
             * @return array
             */
            public function findActiveMessagesRawSql($limit)
            {
                $query = "SELECT * FROM `" . $this->getTableName() . "` LIMIT ?";

                return $this->dbo->queryForList($query, array($limit));
            }
        }

В данном классе нужно указать название таблицы базы данных в методе **getTableName**, а также  название **DTO** класса в методе **getDtoClassName**.
Следует отметить тот факт, что  если вы работаете только с одной таблицей то необходимо использовать конструктор запросов - **OW_Example**,
а если в запросе необходимы сложные объединения тогда нужно писать сырые запросы к базе данных,
как это сделано в методе - **findActiveMessagesRawSql**.

Сервис
------

Класс **service.php** является центральным для плагина, так как именно его нужно использовать в качестве провайдера данных,
а также инкапсулировать в нем всю бизнес логику плагина (даже если вы не работает с БД). Стоит отметить,
что если вы используете работу с базой данных то класс **service.php** будет выступать как бы промежуточным слоем,
т.е нельзя из кода контроллеров или откуда-либо еще напрямую обращаться к классам для работы с базой данных для этого нужен сервис.
Класс **service.php** так же как и  классы **DAO** всегда реализует паттерн - **Singleton**. Пример сервиса и инкапсуляции в нем работы с базой данных :

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_BOL_Service
        {
            /**
             * Class instance
             *
             * @var MYSUPERPLUGIN_BOL_Service
             */
            private static $classInstance;

            /**
             * Message DAO
             *
             * @var MYSUPERPLUGIN_BOL_MessageDao
             */
            private $messageDao;

            /**
             * Class constructor
             */
            private function __construct()
            {
                $this->messageDao = MYSUPERPLUGIN_BOL_MessageDao::getInstance();
            }

            /**
             * Returns class instance
             *
             * @return MYSUPERPLUGIN_BOL_Service
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
             * Add message
             *
             * @param MYSUPERPLUGIN_BOL_Message $messageDto
             * @return void
             */
            public function addMessage(MYSUPERPLUGIN_BOL_Message $messageDto)
            {
                $messageDto->createdTimestamp = time();
                $messageDto->approved = MYSUPERPLUGIN_BOL_MessageDao::APPROVED_STATUS;
                $this->messageDao->save($messageDto);
            }

            /**
             * Remove message
             *
             * @param integer $messageId
             * @return void
             */
            public function deleteMessage($messageId)
            {
                $this->messageDao->deleteById($messageId);
            }

            /**
             * Find active messages
             *
             * @param integer $limit
             * @return array
             */
            public function findActiveMessages($limit)
            {
                return $this->messageDao->findActiveMessages($limit);
            }

            // … etc
        }

**PS: По возможности нужно максимально выносить логику из контроллеров и компонентов в сервисы и делать эти методы готовыми к повторному использованию.**