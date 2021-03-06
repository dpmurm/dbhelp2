# Урок 6 : Регистрация и авторизация. Часть 1

Сегодня с вами хотел бы поговорить про такую вещь как авторизация, регистрация и аутентификация. В дальнейшем я предполагаю что пользователи нашего блога смогут оставлять комментарии, сами создавать статьи и редактировать свой профиль. Чтобы это реализовать - надо во-первых разграничивать права пользователей, а во-вторых написать саму регистрацию (и обработку этих самых пользователей)...

## Вступление

Я думаю моя статья не тянет на премию оригинальности т.к. вопросы освещенные в ней уже много раз поднимались ("Аутентификация и авторизация" [рус] и "Building a Blog System using Yii" [англ]). Несмотря на то что уже много было сказано по этой теме, я не могу пропустить такой важный момент в своей серии уроков "Создаем блог вместе".

## Начнем

Наш небольшой план на данный урок выглядит следующим образом:

1. Создаем для пользователя отдельный контроллер

2. Создаем модель для работы с таблицей "users"
3. Создаем форму регистрации
4. Работа с моделью. Правила валидации
5. Создаем форму авторизации
6. Привязываем механизм авторизации
7. Работа с моделью. Правила валидации

Приступим...

## Создаем отдельный контроллер

Так как пользователи на нашем сайте это отдельные логические единицы которые должны в будущем обладать функциями:

1. Авторизации

2. Изменения профиля
3. Выхода (logout)
4. и тп.

то логичнее будет выделить всё это в отдельный контроллер.

Открываем protected/controllers и создаем UserController.php:

```php
<?php
/**

 * UserController
 * 
 * Контроллер для наших пользователей. Содержит в себе следующие функции:
 * - авторизация
 * - регистрация
 * - выход
 * - редактирование профиля [в будущем]
  * 
 * @version 1.0
    *
     */
    class UserController extends CController
    {       
    public function actions()
    {
        return array(
            // Создаем экшинс captcha.
            // Он понадобиться нам для формы регистрации (да и авторизации)
             'captcha'=>array(
                'class'=>'CCaptchaAction',
                'backColor'=> 0x003300,
                'maxLength'=> 3,
                'minLength'=> 3,
                 'foreColor'=> 0x66FF66,
            ),
        );
    }

    /**
     * Метод входа на сайт
     * 
     * Метод в котором мы выводим форму авторизации
     * и обрабатываем её на правильность.
         */
         public function actionLogin()
         {
         }    

    /**
     * Метод выхода с сайта
     * 
     * Данный метод описывает в себе выход пользователя с сайта
     * Т.е. кнопочка "выход"
         */
         public function actionLogout()
         {
         }

    /**
     * Метод регистрации
        *
     * Выводим форму для регистрации пользователя и проверяем
     * данные которые придут от неё.
         */
         public function actionRegistration()
         {
         }

} 
```
Я сразу добавил туда пустых методов которые нам понадобятся. Надеюсь прочитав комментарии вам сразу стало понятно что для чего :)

## Создаем модель

Как вы понимаете все наши пользователи будут находиться в таблице user. Сама таблица у нас уже была создана в прошлых уроках, а вот модель для её работы мы не создали. Я надеюсь вы помните по старым урокам что именно модель является связующим звеном нашего контроллера с базой данных, поэтому мы в ней очень сильно нуждаемся.

Заходим в protected/models и создаем файл User.php:
```php
<?php
class User extends CActiveRecord
{        
      // для капчи
      public $verifyCode;
      // для поля "повтор пароля"
      public $passwd2;

      public static function model($className=__CLASS__)
      {
          return parent::model($className);
      }
      public function tableName()
      {
          return 'user';
      }
}
```

Нам этого каркаса хватит для простой работы с таблицей user. Правила влаидации, лэйблы и сэйв атрибуты - мы напишем чуть-чуть позже (в этом уроке!).

## Создаем форму регистрации

Создание формы регистрации в принципе включает в себя два пункта: создание отображение, обработка формы в экшинсе.
Давайте будем двигаться по логичной схеме и первым делом создадим actionRegistration

И так если вы забыли, открыть надо UserController.php. У нас там есть:

```php
public function actionRegistration()
{
}
```

заменяем на нечто подобное:

```php
public function actionRegistration()
 {
    // тут думаю все понятно
    $form = new User();

    // Проверяем являеться ли пользователь гостем
    // ведь если он уже зарегистрирован - формы он не должен увидеть.
    if (!Yii::app()->user->isGuest) {
         throw new CException('Вы уже зарегистрированны!');
    } else {
        // Если $_POST['User'] не пустой массив - значит была отправлена форма
        // следовательно нам надо заполнить $form этими данными
         // и провести валидацию. Если валидация пройдет успешно - пользователь
        // будет зарегистрирован, не успешно - покажем ошибку на экран
        if (!empty($_POST['User'])) {
            
             // Заполняем $form данными которые пришли с формы
            $form->attributes = $_POST['User'];
            
            // Запоминаем данные которые пользователь ввёл в капче
             $form->verifyCode = $_POST['User']['verifyCode'];
            
                // В validate мы передаем название сценария. Оно нам может понадобиться
                // когда будем заниматься созданием правил валидации [читайте дальше]
                 if($form->validate('registration')) {
                    // Если валидация прошла успешно...
                    // Тогда проверяем свободен ли указанный логин..

                        if ($form->model()->count("login = :login", array(':login' => $form->login))) {
                             // Указанный логин уже занят. Создаем ошибку и передаем в форму
                            $form->addError('login', 'Логин уже занят');
                            $this->render("registration", array('form' => $form));
                         } else {
                            // Выводим страницу что "все окей"
                            $form->save();
                            $this->render("registration_ok");
                        }
                                         
                } else {
                    // Если введенные данные противоречат 
                    // правилам валидации (указаны в rules) тогда
                    // выводим форму и ошибки.
                     // [Внимание!] Нам ненадо передавать ошибку в отображение,
                    // Она автоматически после валидации цепляеться за 
                    // $form и будет [автоматически] показана на странице с 
                     // формой! Так что мы тут делаем простой рэндер.
                    
                    $this->render("registration", array('form' => $form));
                }
         } else {
            // Если $_POST['User'] пустой массив - значит форму некто не отправлял.
            // Это значит что пользователь просто вошел на страницу регистрации
            // и ему мы должны просто показать форму.
             
            $this->render("registration", array('form' => $form));
        }
    }
}
```

После такого кол-ва комментариев в коде - логику как это работает думаю можно не объяснять. Как вы видите после успешной регистрации я делаю render страницы registration_ok. Такой страницы у нас пока еще не создано (как и registration), но их создание будет описано дальше.

Проверку на то, занят логин или нет, я делаю в экшинсе специально. Этим я хотел показать вам
что вы можете использовать addError в контроллере. Правильнее было бы вынести это в качестве
отдельного правила (rules) в модель.

Для вывода ошибок я использую встроенный класс CException. Это весьма неудобно на рабочих проектах,
поэтому вы можете заменить эту строчку на render какой то страницы для ошибок.

Теперь перейдем к созданию непосредственно самой формы.

Заходим в views/user и создаем registration_ok.php:

```html
<h1>Спасибо</h1>
 <p>Регистрация прошла успешно. Теперь вы можете войти под своим логином</p>
```


Теперь там же создаем registration.php:
```html
<h1>Регистрация</h1>
<!-- Открываем форму !-->
<?=CHtml::form(); ?>
 <!-- То самое место где будут выводиться ошибки
     если они будут при валидации !-->
<?=CHtml::errorSummary($form); ?><br>


<table id="form2" border="0" width="400" cellpadding="10" cellspacing="10">
     <tr>
        <!-- Выводим поле для логина !-->
        <td width="150"><?=CHtml::activeLabel($form, 'login'); ?></td>
        <td><?=CHtml::activeTextField($form, 'login') ?></td>
     </tr>
    <tr>
        <!-- Выводим поле для пароля !-->
        <td><?=CHtml::activeLabel($form, 'passwd'); ?></td>
        <td><?=CHtml::activePasswordField($form, 'passwd') ?></td>
     </tr>
    <tr>
        <!-- Выводим капчу !-->
        <td><?php $this->widget('CCaptcha', array('buttonLabel' => '<br>[новый код]')); ?></td>
         <td><?=CHtml::activeTextField($form,'verifyCode'); ?></td>
    </tr>
    <tr>
        <td></td>
        <!-- Кнопка "регистрация" !-->
         <td><?=CHtml::submitButton('Регистрация', array('id' => "submit")); ?></td>
    </tr>
</table>
<!-- Закрываем форму !-->
 <?=CHtml::endForm(); ?>
```
Я добавил немного комментариев к отображению. Советую удалить их сразу после прочтения т.к.
они будут видны в html коде вашей страницы.

В принципе форма наша уже работает. Зайдите на страницу localhost/user/registration/ и попробуйте.
Если после регистрации вы увидите ошибку от базы данных - не удивляйтесь, ведь наша форма не проверяет входящие данные.

Вы наверняка подумали что я сошел сума, ведь мы используем $form->validate в нашем контроллере и валидация теоретически должна проходить. Теоретически - она проходит, но вы забыли что правил валидации мы негде не указывали :] Поэтому независимо от введенных данных - валидация по пустым правилам будет говорить что всё окей :] Поэтому идем в модель и исправляем это безобразие :]

## Работа с моделью. Правила валидации.

Открываем protected/models/User.php. Добавляем туда метод rules :

```php
/**
 * Правила валидации
 */
public function rules()
{
    return array(
        // логин, пароль не должны быть больше 128-и символов, и меньше трёх
        array('login, passwd', 'length', 'max'=>128, 'min' => 3),
        // логин, пароль не должны быть пустыми
        array('login, passwd', 'required'),
        // для сценария registration поле passwd должно совпадать с полем passwd2
        array('passwd', 'compare', 'compareAttribute'=>'passwd2', 'on'=>'registration'),
        // правило для проверки капчи что капча совпадает с тем что ввел пользователь
        array('verifyCode', 'captcha', 'allowEmpty'=>!extension_loaded('gd')),
        array('login', 'match', 'pattern' => '/^[A-Za-z0-9А-Яа-я\s,]+$/u','message' => 'Логин содержит недопустимые символы.'),
    );
}
```

Этих правил нам будет достаточно для формы регистрации. Теперь зададим safeAttributes, т.е. те поля которые могут быть массово присвоены. safeAttributes выступает в качестве некого защитного механизма. Объясняю в двух словах как это работает и от чего защищает.

У нас есть форма регистрации. Когда мы отправляем её в экшинс приходят данные в виде \$_POST массива, значения которого мы массово присваиваем в \$form при помощи строчки:

```php
$form->attributes = $_POST['User'];
```


Это аналогично если бы мы сделали:

 

```php
$form->login = $_POST['User']['login'];
 $form->passwd = $_POST['User']['passwd'];
 $form->passwd2 = $_POST['User']['passwd2'];
```


и т.п.

После этого проходит валидация и все эти данные сохраняются в базу данных через вызов метода $form->save(). Метод save() проверяет заполнено ли значение первичного ключа в $form (т.е. $form->id): если заполнено - делает update этой записи, если не заполнено - делает insert новой записи в базу данных.

Т.к. у нас id с формы не передается - метод save() всегда будет вставлять новую запись (делать insert), а не делать обновление (update) какой то записи. Все было бы хорошо, но злоумышленник может сохранить вашу форму к себе на компьютер, добавить туда поле "id". Теперь он делает отправку формы с заведомо заполненным полем "id" и у нас получается что метод save() не вставит новую запись, а проведет обновление логина, пароля для записи с указанным "id".

Так вот чтобы строчка

```php
$form->attributes = $_POST['User'];
```


не присваивала в $form переменные которые мы не предполагаем там видеть - надо использовать safeAttributes.В методе safeAttributes() мы перечисляем список полей которые могут быть массово присвоены. Мой метод выглядит следующим образом:

```php
/**
 * Список атрибутов которые могут быть массово присвоены
 * в любом из наших сценариев
 *
 * @return unknown
 */
public function safeAttributes()
{
    return array('login', 'passwd', 'passwd2', 'verifyCode');
}
```


После этого нам нестрашно что кто то будет изменять нашу форму и добавлять туда поля "id". Даже если в \$_POST['id'] будет значение - метод safeAttributes не позволит ему попасть в \$form->id.

Теперь давайте зададим лэйблы. Вы наверняка заметили что на форуме у нас в качестве названия поля выводить "login", "passwd" и тп. Хотелось бы, конечно, это изменить на более понятный язык (русский). Для этого добавляем в модель:

```php
 /**
  * Список синонимов
  */
 public function attributeLabels()
 {
     return array(
         'login'      => 'Логин',
         'passwd'  => 'Пароль',
         'passwd2' => 'Повтори пароль',
     );
 } 
```


Это так, лирическое отступление. Наш файл protected/models/User.php должен выглядеть следующим образом:
```php
<?php
 class User extends CActiveRecord
 {        

 // для капчи
 public $verifyCode;
 // для поля "повтор пароля"
 public $passwd2;
 
 public static function model($className=__CLASS__)
 {
     return parent::model($className);
 }
 
 public function tableName()
 {
     return 'user';
 }
 
 /**
  * Правила валидации
  */
 public function rules()
 {
     return array(
         // логин, пароль не должны быть больше 128-и символов, и меньше трёх
         array('login, passwd', 'length', 'max'=>128, 'min' => 3),
         // логин, пароль не должны быть пустыми
         array('login, passwd', 'required'),
         // для сценария registration поле passwd должно совпадать с полем passwd2
         array('passwd', 'compare', 'compareAttribute'=>'passwd2', 'on'=>'registration'),
         // правило для проверки капчи что капча совпадает с тем что ввел пользователь
         array('verifyCode', 'captcha', 'allowEmpty'=>!extension_loaded('gd')),
         array('login', 'match', 'pattern' => '/^[A-Za-z0-9А-Яа-я\s,]+$/u','message' => 'Логин содержит недопустимые символы.'),
    );
 }
 
 /**
  * Список атрибутов которые могут быть массово присвоены
  * в любом из наших сценариев
  *
  * @return unknown
  */
 public function safeAttributes()
 {
     return array('login', 'passwd', 'passwd2', 'verifyCode');
 }
 
 /**
  * Список синонимов
  */
 public function attributeLabels()
 {
     return array(
         'login'      => 'Логин',
         'passwd'  => 'Пароль',
         'passwd2' => 'Повтори пароль',
     );
 }

 }
```

Вот в принципе с регистрацией мы закончили. Заходим на страницу localhost/user/registration/ и пробуем.

К сожалению, урок получился достаточно затяжной, поэтому авторизацию я вынесу в отдельную часть. 
Ждите продолжение "Урок 6 : Регистрация и авторизация. Часть 2" в ближайшие дни.

Исходники:

- registration.php

- registration_ok.php
- User.php
- UserController.php