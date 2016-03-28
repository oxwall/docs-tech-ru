.. _coding_standards-label:

Стандарты кодирования
=====================

Если вы решили писать код для платформы **Oxwall**, то вам необходимо стараться соблюдать все стандарты кодирования описанные ниже.

Общие
-----

Всегда используйте полную форму **PHP** тега:

.. code-block:: php

    <?php
    ?>

Для файлов, содержащих только **PHP** код, закрывающий тег - **?>** должен быть опущен.

Используя операторы  **include_once** и **require_once** не нужно использовать скобки.


.. code-block:: php

    <?php
        //RIGHT
        require_once 'header.php';

        //WRONG
        require_once('header.php');
    ?>

Отступы
-------

Всегда используйте **4 пробела** для отступов вместо символов таба.


Соглашения об именовании
------------------------

Названия файлов
+++++++++++++++


.. code-block:: php

    view.php
    base_dao.php
    my_super_class.php

Классы
++++++

.. code-block:: php

    class MySuperClass
    {
        //code here
    }

    class PREFIX_MySuperClass
    {
        //code here
    }

Функции
+++++++

.. code-block:: php

    function connect()
    function camelCaseFunction()
    function fooBar()
    my_global_function()

Переменные
++++++++++

.. code-block:: php

    public    $myVar;
    private   $hisVar;
    protected $x;

Константы
+++++++++

.. code-block:: php

    <?php

        define("MY_SUPER_CONSTANT", "Hello world");

Управляющие структуры
---------------------

Нужно использовать пробелы внутри скобок во всех управляющих структурах (foreach, for, while, if, switch, try, catch, итд).

Используйте **фигурные скобки**, даже в случае, если они не являются обязательными, поскольку это сделает код более удобным
для чтения и поможет избежать логических ошибок, возникающих при добавлении новых строк кода.

switch
++++++

.. code-block:: php

    <?php

        switch ( condition )
        {
            case 1:
                action1();
                break;

            case 2:
                action2();
                break;

            default:
                defaultAction();
                break;
        }


if, else
++++++++

Используйте оператор  **else if** вместо  **elseif**

.. code-block:: php

    <?php

        if ( $a !== $b )
        {
            return false;
        }
        else if ( false )
        {
            doSomething1();
        }
        else
        {
            doSomething2();
        }

Разделяете длинное условие на несколько строк

.. code-block:: php

    <?php

        if ( condition1
            || condition2
            && condition3 )
        {
            //code here
        }

foreach, for, while
+++++++++++++++++++

.. code-block:: php

    <?php

        foreach ( $a as $v )
        {
            echo $v;
        }

try, catch
++++++++++

.. code-block:: php

    <?php

        try
        {
            //code here
        }
        catch ( Exception $e )
        {
            //code here
        }

Объявление функций
------------------

Все имена функций должны быть в стиле - **camelCase**. Глобальные функции являются исключением,
они должны состоять из слов в нижнем регистре и символов подчеркивания (**my_global_function**).
В функциях не должно быть пробелов между именем функции и открывающей скобкой, а также перед возвратом значения должен быть символ новой строки.

.. code-block:: php

    <?php

        function fooBar( $param1, $param2 )
        {
            if ( $param1 !=== $param2 )
            {
                //code here
            }

            return true;
        }

Вы всегда должны объявлять типы входных параметров в функциях, когда это возможно:

.. code-block:: php

    <?php

        function doSomethingGood( MyClass $obj )
        {
            //code here
        }

Пример глобальной функции:

.. code-block:: php

    <?php

        function print_var( $var, $echo = false )
        {
            //code here
        }

Вызов функций
-------------

.. code-block:: php

    <?php

        myCoolFunction(1, 2, 3);
        $this->myCoolMethod(1, 2, 3);

Массивы
-------

Индексированные массивы
+++++++++++++++++++++++

.. code-block:: php

    <?php

        $arr = array ( 1, 2, 'no', 'pain', 'no', 'gain' );
        $longArr = array ( 1, 2, 3,
                           4, 5, 6,
                           7, 8, 9 );

Ассоциативные массивы
+++++++++++++++++++++

.. code-block:: php

    <?php

        $assoc = array ( 'key1' => 'value1',
                         'key2' => 'value2',
                         'key3' => 'value3' );