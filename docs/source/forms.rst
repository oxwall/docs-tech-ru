.. _forms-label:

Формы
=====

Платформа **Oxwall** предоставляет достаточно богатый функционал для приема и обработки данных от клиента используя **HTML формы**.
Ниже мы рассмотрим основные моменты по созданию форм, приему и обработки, валидации и фильтрации данных формы.

Фильтры
-------

Фильтры в формах необходимы для первичной очистки вводимых данных перед их валидацией, к примеру если нам не нужны **html** теги или пробелы, мы с легкостью
можем от них избавится добавив нужный фильтр к полю формы. Ниже перечислены встроенные фильтры:

**ow_core/filter.php**

#. **StripTagsFilter** - вырезает все html теги
#. **TrimFilter** - обрезает пробелы из строки

У вас есть возможность создать собственный фильтр реализуя методы интерфейса - **OW_IFilter**

Валидаторы
----------

Основная задача валидаторов - это проверить корректность вводимых данных в поля формы. Для этого в платформе  **Oxwall** предусмотрено
огромное количество валидаторов:

**ow_core/validator.php**

#. **RequiredValidator** - проверяет были ли введены данные.
#. **WyswygRequiredValidator** - проверяет были ли введены данные в визуальный редактор (для того, чтобы проверить данные сперва удаляет все html теги, затем проверяет оставшийся текст).
#. **StringValidator** - проверяет строку или массив строк (опционально можно задать минимальную и максимальную длину строки).
#. **RegExpValidator** - проверяет строку или массив строк на соответствует регулярному выражению.
#. **EmailValidator** - проверяет email или массив email.
#. **UrlValidator** - проверяет url или массив url.
#. **AlphaNumericValidator** - проверяет строку или массив строк на наличие в ней только цифр и символов латинского алфавита.
#. **InArrayValidator** - проверяет строку на ее наличие в списке предопределенных значений.
#. **IntValidator** - проверяет число или массив чисел (опционально можно задать минимальное и максимальное число).
#. **FloatValidator** - проверяет действительное число или массив действительных чисел (опционально можно задать минимальное и максимальное число).
#. **DateValidator** - проверяет дату или массив дат (опционально можно задать формат даты, а также минимальный и максимальный год).
#. **CaptchaValidator** - проверяет значение `captcha <https://en.wikipedia.org/wiki/CAPTCHA>`_.
#. **RangeValidator** - проверяет число или массив чисел на нахождение их в некотором интервале.

Не смотря на большое количество встроенных валидатров, у вас всегда есть возможность создать собственный валидатор реализуя методы
абстрактного класса - **OW_Validator**

Создание формы
--------------

Пример создания формы :

**classes/user_form.php**

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_CLASS_UserForm extends Form
        {
            /**
             * Class constructor
             */
            public function __construct(MYSUPERPLUGIN_BOL_User $user = null)
            {
                // set the form name
                parent::__construct('user-form');

                // allow upload files
                $this->setEnctype(Form::ENCTYPE_MULTYPART_FORMDATA);

                // add user name field
                $userName = new TextField('userName');
                $userName->addFilter(new StripTagsFilter());
                $userName->setRequired(true);
                $userName->setValue(($user ? $user->userName : null));
                $this->addElement($userName);

                // add user email field
                $email = new TextField('email');
                $email->setRequired(true);
                $email->addValidator(new EmailValidator());
                $email->setValue(($user ? $user->email : null));
                $this->addElement($email);

                // add user age field
                $age = new TextField('age');
                $age->setRequired(true);
                $age->addValidator(new IntValidator());
                $age->setValue(($user ? $user->age : null));
                $this->addElement($age);

                // add user description field
                $description = new Textarea('description');
                $description->addFilter(new StripTagsFilter());
                $description->addFilter(new TrimFilter());
                $description->setValue(($user ? $user->description : null));
                $this->addElement($description);

                // add user image field
                $image = new FileField('image');
                $image->addValidator(new CustomImageValidator());
                $this->addElement($image);
            }
        }

        /**
         * Custom image validator
         */
        class CustomImageValidator extends OW_Validator
        {
            /**
             * Class constructor
             */
            public function __construct()
            {
                $this->setErrorMessage(OW::getLanguage()->text('mysuperplugin', 'image_validator_error_message'));
            }

            /**
             * Is image valid
             *
             * @param mixed $value
             * @return boolean
             */
            public function isValid( $value )
            {
                if ( !empty($value['name']) && isset($value['tmp_name'], $value['error']) )
                {
                    // validate image
                    return (int) $value['error'] === 0 &&
                            is_uploaded_file($value['tmp_name']) &&
                            UTIL_File::validateImage($value['name']) && getimagesize($value['tmp_name']);
                }

                return true;
            }
        }

Валидация формы
---------------

Для проверки введенных данных вы можете использовать ниже приведенный кусок кода в коде контроллера:

**controller/user.php**

.. code-block:: php

    <?php

        class MYSUPERPLUGIN_CTRL_User extends OW_ActionController
        {
            /**
             * Service
             *
             * @var MYSUPERPLUGIN_BOL_Service
             */
            protected $service;

            /**
             * Constructor
             */
            public function __construct()
            {
                parent::__construct();
                $this->service = MYSUPERPLUGIN_BOL_Service::getInstance();
            }

            /**
             * Add user
             */
            public function addUser()
            {
                // check permission
                $isAddAllowed = OW::getUser()->isAuthorized('superplugin', 'add_user');

                if ( !isViewAllowed )
                {
                    // get error message
                    $errorMessage = BOL_AuthorizationService::getInstance()->getActionStatus('superplugin', 'add_user');
                    throw new AuthorizationException($errorMessage['msg']);
                }

                // validate the form data
                $form = new MYSUPERPLUGIN_CLASS_UserForm();

                // make certain to merge the files info!
                $post = array_merge_recursive(
                    $_POST,
                    $_FILES
                );

                // validate the form
                if ( OW::getRequest()->isPost() && $form->isValid($post) )
                {
                    // get validated and filtered form values
                    $formValues = $form->getValues();

                    // add a new user
                    $userBol = new MYSUPERPLUGIN_BOL_User;
                    $userBol->userName = $formValues['userName'];
                    $userBol->email = $formValues['email'];
                    $userBol->age = $formValues['age'];
                    $userBol->description = !empty($formValues['description']) ? $formValues['description'] : null;
                    $userBol->createdStamp = time();
                    $userBol->ownerId = OW::getUser()->getId();

                    $this->service->addUser($userBol, $formValues['image']);

                    OW::getFeedback()->info(OW::getLanguage()->text('superplugin', 'user_successfully_added');
                    $this->redirect();
                }

                // init components
                $this->addForm($form);

                // init page settings
                OW::getDocument()->setHeading(OW::getLanguage()->text('superplugin', 'page_title_add_user'));
                OW::getDocument()->setHeadingIconClass('ow_ic_user');
                OW::getDocument()->setTitle(OW::getLanguage()->text('superplugin', 'page_title_add_user'));
            }
        }

Отображение формы
-----------------

**views/controllers/add_user.html**

.. code-block:: html

        {form name="user-form"}
            <table class="ow_table_1 ow_form">
                <tbody class="ow_paging">
                    <tr>
                        <td class="ow_td_label">* {text key="superplugin+user_name"}</td>
                        <td class="ow_value ow_valign_middle">
                            <div>{input name="userName"}</div>
                            <div>{error name="userName"}</div>
                        </td>
                    </tr>
                    <tr>
                        <td class="ow_td_label">* {text key="superplugin+email"}</td>
                        <td class="ow_value ow_valign_middle">
                            <div>{input name="email"}</div>
                            <div>{error name="email"}</div>
                        </td>
                    </tr>
                    <tr>
                        <td class="ow_td_label">* {text key="superplugin+age"}</td>
                        <td class="ow_value ow_valign_middle">
                            <div>{input name="age"}</div>
                            <div>{error name="age"}</div>
                        </td>
                    </tr>
                    <tr>
                        <td class="ow_td_label">{text key="superplugin+description"}</td>
                        <td class="ow_value ow_valign_middle">
                            <div>{input name="description"}</div>
                            <div>{error name="description"}</div>
                        </td>
                    </tr>
                    <tr>
                        <td class="ow_td_label">{text key="superplugin+image"}</td>
                        <td class="ow_value ow_valign_middle">
                            <div>{input name="image"}</div>
                            <div>{error name="image"}</div>
                        </td>
                    </tr>
                </tbody>
            </table>
            <div class="clearfix">
                {submit name="save" class="ow_ic_save ow_submit ow_right"}
            </div>
        {/form}