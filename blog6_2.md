# Урок 6 : Регистрация и авторизация. Часть 2

Продолжение...


Я надеюсь вы уже знакомы с прошлой частью урока, и настроились к познанию нового материала :) Давайте немного вrспомним о том что было в прошлой части и что у нас уже готово:

- форма поле должно проверять правиломрегистрации

- обработка формы. валидация полей (данных). сохранение данных
- модель, контроллер и пару отображений

В этой части урока мы полностью закончим с авторизацией и регистрацией и сможем приступить к более серьезным (и интересным) вещам :)

## Работаем с экшинсом

В прошлой части урока мы создали отдельный экшинс для регистрации, давайте сделаем нечто похожее и для авторизации. Открываем наш контроллер UserController.php и находим:

```php
/**
  * Метод входа на сайт
 * 
 * Метод в котором мы выводим форму авторизации
 * и обрабатываем её на правильность.
 */
public function actionLogin()
{
}    
```


И заменяем на :

```php
/**
 * Метод входа на сайт
 * 
 * Метод в котором мы выводим форму авторизации
 * и обрабатываем её на правильность.
 */
public function actionLogin()
{
    $form = new User();
     
    // Проверяем является ли пользователь гостем
    // ведь если он уже зарегистрирован - формы он не должен увидеть.
    if (!Yii::app()->user->isGuest) {
        throw new CException('Вы уже зарегистрированы!');
     } else {
        if (!empty($_POST['User'])) {
            $form->attributes = $_POST['User'];
            $form->verifyCode = $_POST['User']['verifyCode'];
 
                // Проверяем правильность данных
                if($form->validate('login')) {
                    // если всё ок - кидаем на главную страницу
                    $this->redirect(Yii::app()->homeUrl);
                 } 
        } 
        $this->render('login', array('form' => $form));
    }
}    
```

## Форма

Конструкция очень похожа на actionRegistration() поэтому я не делал сильно много комментариев.

Как вы уже поняли по коду - форма у нас будет находиться в файле login. Поэтому заходим views/user/ и создаем файл login.php:
```html
<h1>Авторизация</h1>

<?=CHtml::form(); ?>
<?=CHtml::errorSummary($form); ?><br>


<table id="form2" border="0" width="400" cellpadding="10" cellspacing="10">
    <tr>
        <td width="150"><?=CHtml::activeLabel($form, 'login'); ?></td>
         <td><?=CHtml::activeTextField($form, 'login') ?></td>
    </tr>
    <tr>
        <td><?=CHtml::activeLabel($form, 'passwd'); ?></td>
         <td><?=CHtml::activePasswordField($form, 'passwd') ?></td>
    <tr>
    <tr>
        <td><?php $this->widget('CCaptcha', array('buttonLabel' => '<br>[новый код]')); ?></td>
         <td><?=CHtml::activeTextField($form,'verifyCode'); ?></td>
    </tr>
    <tr>
        <td></td>
        <td><?=CHtml::submitButton('Войти', array('id' => "submit")); ?></td>
     </tr>
</table>


<?=CHtml::endForm(); ?>
```
Почти аналогичная форма как и у регистрации (только без поля повтора пароля). Хотел бы обратить внимание что на форме авторизации я оставил поле капчи (для того чтобы боты не авторизировались самостоятельно).

Теперь заходим на страницу localhost/user/login/ и видим красивую формочку :)
Можете даже попробовать ввести в неё какие то не правильные данные (указать неверный код капчи, использовать короткий логин/пароль) и убедиться что наши правила работают :)

Если вы введете верные данные - вас просто переведет на главную страницу. Механизм сохранения мы еще не писали, поэтому это нормально.

## UserIdentity

Как вы понимаете сам процесс аутентификации уже был вложен в Yii и от нас требуется лишь наследовать класс CUserIdentity и пере определить метод authenticate под свои нужды.

Если вам вдруг будет непонятно про Identity, вы можете прочитать статью про авторизацию вот тут.

В authenticate мы проверим совпадают ли введённые данные (логин/пароль) с тем что у нас есть в базе данных. Заходим в protected/ и создаем там папку components, а в ней файл UserIdentity.php:
```php
 class UserIdentity extends CUserIdentity
 {

 private $_id;
 public function authenticate()
 {
     // Есть ли указанный пользователь в базе данных
     $record=User::model()->findByAttributes(array('login'=>$this->username));
     if($record===null)
         // Если нету - сохраняем в errorCode ошибку.
         $this->errorCode=self::ERROR_USERNAME_INVALID;
     else if($record->passwd!==$this->password)
         // Проверяем совпадает ли введенный пароль с тем что у нас в базе
         // Если не совпадает - сохраняем в errorCode ошибку.
         $this->errorCode=self::ERROR_PASSWORD_INVALID;
     else
     {
         // Если все действий выше успешно пройдены - значит
         // пользователь действительно существует и пароль был
         // указан верно. 
         $this->_id=$record->id;
         // В errorCode сохраняем что ошибок нет
         $this->errorCode=self::ERROR_NONE;
     }
     return !$this->errorCode;
 }
  
 public function getId()
 {
     return $this->_id;
 }

 }
```

Давайте разберем что мы написали выше.

1. Мы создали класс UserIdentity с наследованием от CUserIdentity

2. Пере определили метод authenticate() в которой сделали проверку на наличие такого логина в базе, а после этого проверка на совпадение пароля.
3. Метод возвращает у нас errorCode. Если значение "ERROR_NONE" - значит ошибок нету.

## Модель

Теперь нам надо в моделе создать специально правило для того что бы сразу проверять введенные логин/пароль нашим методом.

Открываем protected/models/User.php и в метод rules добавляем еще одно правило:

```php
array('passwd', 'authenticate', 'on' => 'login'),
```


В конечном итоге rules у нас должны выглядеть следующим образом:

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
        // проверка пароля нашим методом authenticate
         array('passwd', 'authenticate', 'on' => 'login'),
       array('login', 'match', 'pattern' => '/^[A-Za-z0-9А-Яа-я\s,]+$/u','message' => 'Логин содержит недопустимые символы.'),
   );
}
```

Как вы уже помните - в массиве первым элементом я задал поле, которое должно быть проверенно правилом, а во втором параметре указал как должен называться метод для проверки. Теперь давайте создадим authenticate нашей модели.

Хочу заметить что в данном случае речь идет не про метод authenticate который мы создали в UserIdentity. В данном случае я просто использую точно такое же имя, но речь уже идет про метод authenticate который мы сейчас создадим в User.php

И так, ниже в этом же файле добавляем новый метод:

```php
/**
 * Собственное правило для проверки
 * Данный метод является связующем звеном с UserIdentity
 *
 * @param $attribute
  * @param $params
 */
public function authenticate($attribute,$params) 
{
    // Проверяем были ли ошибки в других правилах валидации.
    // если были - нет смысла выполнять проверку
     if(!$this->hasErrors())
    {
        // Создаем экземпляр класса UserIdentity
        // и передаем в его конструктор введенный пользователем логин и пароль (с формы)
        $identity= new UserIdentity($this->login, $this->passwd);
         // Выполняем метод authenticate (о котором мы с вами говорили пару абзацев назад)
        // Он у нас проверяет существует ли такой пользователь и возвращает ошибку (если она есть)
        // в $identity->errorCode
         $identity->authenticate();
            
            // Теперь мы проверяем есть ли ошибка..    
            switch($identity->errorCode)
            {
                // Если ошибки нету...
                 case UserIdentity::ERROR_NONE: {
                    // Данная строчка говорит что надо выдать пользователю
                    // соответствующие куки о том что он зарегистрирован, срок действий
                     // у которых указан вторым параметром. 
                    Yii::app()->user->login($identity, 0);
                    break;
                }
                case UserIdentity::ERROR_USERNAME_INVALID: {
                     // Если логин был указан наверно - создаем ошибку
                    $this->addError('login','Пользователь не существует!');
                    break;
                }
                 case UserIdentity::ERROR_PASSWORD_INVALID: {
                    // Если пароль был указан наверно - создаем ошибку
                    $this->addError('passwd','Вы указали неверный пароль!');
                     break;
                }
            }
    }
}
```

Я надеюсь логика работы вам в принципе понятна. Т.е. мы с вами написали правило (rules) которое проверяет поле passwd нашим методом authenticate. В нем мы обращаемся к классу UserIdentity и проверяем существует ли такой пользователь с таким паролем. Если все хорошо - методом login проводим "вход" пользователя.

## Выход

Когда со "входом" на сайт мы разобрались, давайте сделаем "выход (log out)".
Для этого мы уже написали каркас экшинса в контроллере, который сейчас мы дополним.

Открываем protected/controllers/UserController.php и вместо:

```php
/**
 * Метод выхода с сайта
 * 
 * Данный метод описывает в себе выход пользователя с сайта
 * Т.е. кнопочка "выход"
 */
public function actionLogout()
{
 }
```

делаем :

```php
/**
 * Метод выхода с сайта
 * 
 * Данный метод описывает в себе выход пользователя с сайта
 * Т.е. кнопочка "выход"
 */
public function actionLogout()
 {
    // Выходим
    Yii::app()->user->logout();
    // Перезагружаем страницу
    $this->redirect(Yii::app()->user->returnUrl);
}
```


Всё достаточно просто.

## Меню

Теперь что бы увидеть как это все работает - думаю есть смысл создать просто меню:

```html
<a href="<?=Yii::app()->createUrl("post/index");?>">Главная</a>
 <?php if (Yii::app()->user->isGuest): ?>
     <a href="<?=Yii::app()->createUrl("user/registration/");?>">Регистрация</a>
     <a href="<?=Yii::app()->createUrl("user/login/");?>">Вход</a>
 <?php else: ?>
     <a href="<?=Yii::app()->createUrl("user/logout/");?>">Выход (<?=Yii::app()->user->name?>)</a>
 <?php endif; ?>
```

Оно состоит из проверки является ли пользователь гостем. В зависимости от этого он будет видеть нужные пункты меню. К примеру для гостя это будет - регистрация и вход, а для авторизированного пользователя - кнопка выход.

Я не хочу сильно мудрить с выносом меню в отдельный файл и тп, поэтому мы просто добавим его в наш layouts. Открываем protected/views/layouts/main.php и добавляем в него код нашего меню:
```html
<html>
 <head>
     <title><?php echo $this->pageTitle; ?></title>
     <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
     <meta name="language" content="ru" />
     <meta name="robots" content="all" />
 </head>
 <body>


 <a href="<?=Yii::app()->createUrl("post/index/");?>">Главная</a>
 <?php if (Yii::app()->user->isGuest): ?>
     <a href="<?=Yii::app()->createUrl("user/registration/");?>">Регистрация</a>
     <a href="<?=Yii::app()->createUrl("user/login/");?>">Вход</a>
 <?php else: ?>
     <a href="<?=Yii::app()->createUrl("user/logout/");?>">Выход (<?=Yii::app()->user->name?>)</a>
 <?php endif; ?>
 
 <?php echo $content; ?>

 </html>
```
## Конец

Теперь всё работает. Заходите на localhost и смотрите сами :)

Исходники:

- main.php

- User.php
- UserController.php
- UserIdentity.php
- login.php
- структура проекта + исходники урока (rar)