# Урок 7 : Комментарии

Сегодня разговор пойдет о том как сделать комментарии на нашем блоге.


В этой статье мы рассмотрим с вами как добавить формочку комментариев к нам в блог, и безопасно обрабатывать входные данные. Простой план сегодняшнего занятия выглядит следующим образом:

1. Создаем форму (отображение) для добавления комментария

2. Подключаем форму на страницу
3. Подключаем вывод всех комментариев по посту внизу страницы
4. Работа с контроллером. Добавляем поиск всех комментариев
5. Работа с контроллером. Механизм проверки что пришли данные с формы
6. Создаем модель Comments
7. Пробуем

В принципе ничего замысловатого. Ну что, начнем?

## Введение

Форма добавления комментариев будет выглядеть примерно точно также как у меня на dbhelp.ru. Как вы понимаете комментарии у нас смогут оставлять как зарегистрированные так и не зарегистрированные пользователи.

### Создаем форму для добавления комментария

Открываем protected/views/post/ и создаем там файл _commentsForm.php:
```html
<h1 id="respond">

<?php echo CHtml::form(); ?>
<?php echo CHtml::errorSummary($form); ?>

    <?php echo CHtml::activeTextArea($form, 'text', array('cols' => 50, 'rows' => 8)) ?>
    <table border=0 id="captcha">
    <tr>
        <td valign="top"><?php $this->widget('CCaptcha', array('buttonLabel' => '')); ?>
        <td valign="top">
            <table border=0>
            <tr>
                <td valign="top"><?php echo CHtml::activeLabel($form, 'verifyCode'); ?></td>
                <td valign="top"><?php echo CHtml::activeTextField($form,'verifyCode'); ?></td>
            </tr>
            <tr id="login">
                <td valign="top"><?php echo CHtml::activeLabel($form, 'login'); ?></td>
                <td valign="top"><?php echo CHtml::activeTextField($form, 'login') ?></td>
            </tr>
            </table>
        </td> 
    </tr>
    </table>
    
    <p><?php echo CHtml::submitButton("Ок, отправить!", array('id' => "submit")); ?></p>

<?php echo CHtml::endForm($form); ?>

Тут же внизу я добавляю немного javascript (jQuery) кода для красоты, который чуть ниже мы рассмотрим подробнее:


<script>
    $(document).ready(function() {
        $("#captcha").hide();
        $("#guestlogin").hide();
            
                $("#Comments_text").click(function(){
                    <?php if (Yii::app()->user->isGuest): ?>    
                        $("#guestlogin").show();
                    <?php endif; ?>
                        $("#captcha").show();
                });
    });
</script>
```

Данный код отображает поля "капча" и "имя" при клике на поле ввода текста.
Также хочу заметить что поле "имя" выводиться только если пользователь гость. (для этого в коде поставлена соответствующая проверка).
.
Как вы видите код написан с использованием jQuery, поэтому его необходимо предварительно подключить на сайт. Я не агитирую использовать именно jQ для таких вещей, просто привёл пример того как это делаю я. Интересная вещь, мой Yii сам увидел что страница использует jQuery код и сам подключил его! Так что в ручную вам добавлять какой либо код подключения библиотеки не придется, всё делает фреймворк.

Если всё же jQuery у вас автоматически не подключился используйте следующие строчки:

```php
$cs=Yii::app()->clientScript;
$cs->registerCoreScript('jquery');
```

добавить их надо в PostController -> actionView()


Подключаем вывод всех комментариев по посту внизу страницы

Я надеюсь вы не забыли что кроме формы добавления комментариев, нам надо бы еще вывести все комментарии для темы. (т.е. которые написали уже до вас)
Так как мы начали уже работать с отображением, давайте не будем далеко ходить и организуем файл вывода (предполагая что список всех комментариев у нас находиться в переменной $comments).

Т.е. что я предлагаю сделать: сейчас создадим файл отображения который будет выводить нам список всех комментариев которые уже есть. Как вы понимаете с контроллера к нам еще не какие данные не приходят в отображение (мы еще не дошли до работы с контроллером), поэтому мы должны представить что переменная $comments уже содержит в себе список нужных нам комментариев.

Это способ разработки от конца к началу. Мы сначала создаем файл который будет обрабатывать данные которые должны прийти (отображение), а потом создаем файл который эти данные будет передавать (экшинс контроллера).

Откройте файл protected/views/post/view.php. Именно этот файл отвечает за вывод страницы по конкретному посту. В него нам надо добавить вывод списка всех комментариев и вывод формы добавления. Я это сделал следующим образом:

```html
<table border="1" width="100%">

    <?php if (!empty($post)) : ?>
        <tr><td><h1><?=$post->name;?></h1></td></tr>
        <tr><td><?=$post->created;?></td></tr>
        <tr><td><?=$post->text;?></td></tr>
        <tr>
            <td>
                <h3>Комментарии</h3>
                <table border="0" width="100%" cellpadding="10" cellspacing="10">
                    <?php
                      if (!empty($comments))
                        foreach ($comments as $key => $val) {
                            $this->renderPartial('_comments',array(
                                'comment'=>$val,
                            )); 
                        }
                    ?>
                </table>
                
                <div align="center">
                    <?php echo $this->renderPartial('_commentsForm', array('form' => $comm_form));?>
                </div>
                
            </td>
        </tr>
    <?php endif; ?>

</table>
```

У нас добавилось еще одно поле в таблицу - комментарии. После него в цикле мы каждый комментарий загружаем через отображением _comments (в которое мы передаем данные текущего комментария). Данный подход я уже использовал вот в этом уроке.

Чуть ниже я подключил файл _commentsForm который мы с вами создали чуть выше. Я надеюсь с этим всё понятно.
По поводу переменных $comments и $comm_form - вы всё поймете когда мы будем работать с контроллером. (хотя я надеюсь вы уже сейчас понимаете для чего они примерно надо и что в себе должны содержать).

Давайте перейдем к созданию файла protected/views/post/_comments.php:
```html
<tr>
    <td>
        <table border="1" width="100%">
            <tr><td><?php echo $comment->login; ?></td></tr>
            <tr><td><?php echo nl2br(htmlentities($comment->text, null, 'UTF-8')); ?></td></tr>
        </table>
    </td>
</tr>
```

Работа с контроллером. Добавляем поиск всех комментариев

Теперь было бы отлично зайти в наш контроллер PostController и немного подредактировать метод actionView():
```php
public function actionView ()
        {
            $this->pageTitle = "";

            if (!empty($_GET['url'])) {
                // На всякий случай удаляю пробелы и устанавливаю
                // максимальную длину для url в 100 символов.
                $url = substr(trim($_GET['url']), 0, 100);
                
                // Только англ. буквы и цифры в url
                if(preg_match("/^[a-zA-Z0-9\-\_]+$/", $url)) {
                   $post = Posts::model()->find("url = :url", array(':url' => $url));
                   
                    // Тема есть в базе
                    if (!empty($post)) {
                        
                        // Для того чтобы сгенерировать форму добавления комментария
                        $comm_form = new Comments();
                        
                        // Создаем критерию отбора данных
                        $com_criteria              = new CDbCriteria;
                        // Нам нужны все записи где id_post = текущему посту
                        $com_criteria->condition = 'id_post = :id';
                        // Вместо :id посдтавляем реальное значение
                        $com_criteria->params     = array(':id' => $post->id);
                        // Сортируем данные. Что бы новые отзывы - были вверху
                        $com_criteria->order     = 'created ASC';
                        // Передаем критерию в findAll
                        $comments = $comm_form->findAll($com_criteria);
                        
                        $this->render('view', array(
                            'post'         => $post,
                            // результат передаем в отображение.
                            'comments'    => $comments,
                            'comm_form' => $comm_form,
                        ));
                        
                    } else {
                        // Такой темы в базе нет. 404?
                        Yii::app()->runController('post/error404');
                    }
                } else {
                    Yii::app()->runController('post/error403');
                }
            } else {
                // $_GET['url'] пустое. Выводим главную страницу
                Yii::app()->runController('post/index');
            }
        }
```
Мы делаем простую выборку комментариев текущего поста.

### Работа с контроллером. Механизм проверки что пришли данные с формы

Теперь давайте проверим пришли ли данные с формы и выполним валидацию. Поэтому в метод actionView() добавляем еще немного кода, и теперь он выглядит вот так:

```php
        public function actionView ()
        {
            $this->pageTitle = "";

            if (!empty($_GET['url'])) {
                // На всякий случай удаляю пробелы и устанавливаю
                // максимальную длину для url в 100 символов.
                $url = substr(trim($_GET['url']), 0, 100);
                
                // Только англ. буквы и цифры в url
                if(preg_match("/^[a-zA-Z0-9\-\_]+$/", $url)) {
                   $post = Posts::model()->find("url = :url", array(':url' => $url));
                   
                    // Тема есть в базе
                    if (!empty($post)) {
                        
                        // Для того чтобы сгенерировать форму добавления комментария
                        $comm_form = new Comments();
                        
                        // Если был отправлен новый комментарий...
                        if (!empty($_POST['Comments']))
                        {
                            $comm_form->attributes = $_POST['Comments'];
                            $comm_form->id_post       = (int)$post->id;
                            $comm_form->created    = date('Y-m-d H:i:s');
                            
                                if (!Yii::app()->user->isGuest) {
                                    // Если не гость - заполняем запись действиельным
                                    // айдишником владельца комментария и ником
                                    $comm_form->id_user = Yii::app()->user->id;
                                    $comm_form->login     = Yii::app()->user->name;
                                } else {
                                    // Если гость...
                                    $comm_form->id_user = 0;
                                }
                            
                            // Валидация специально идет после присваивания значению login
                            // того что ввёл гость через форму. Я это сделал для того чтобы
                            // введенные данные были проверены через rules
                            if ($comm_form->validate('add')) {
                                // ну теперь можем делать save..
                                $comm_form->save();
                            } else {
                                // ошибка при добавлении комментария.
                                // не прошла валидация..
                            }
                        }
                        
                        // Создаем критерию отбора данных
                        $com_criteria              = new CDbCriteria;
                        // Нам нужны все записи где id_post = текущему посту
                        $com_criteria->condition = 'id_post = :id';
                        // Вместо :id посдтавляем реальное значение
                        $com_criteria->params     = array(':id' => $post->id);
                        // Сортируем данные. Что бы новые отзывы - были вверху
                        $com_criteria->order     = 'created ASC';
                        // Передаем критерию в findAll
                        $comments = $comm_form->findAll($com_criteria);
                        
                        $this->render('view', array(
                            'post'         => $post,
                            // результат передаем в отображение.
                            'comments'    => $comments,
                            'comm_form' => $comm_form,
                        ));
                        
                    } else {
                        // Такой темы в базе нет. 404?
                        Yii::app()->runController('post/error404');
                    }
                } else {
                    Yii::app()->runController('post/error403');
                }
            } else {
                // $_GET['url'] пустое. Выводим главную страницу
                Yii::app()->runController('post/index');
            }
        }
```
Я надеюсь комментариев в коде вам достаточно что бы понять суть работы. 

### Создаем модель Comments

Теперь мы перешли к одному из главных моментов - создание модели.
Создаем файл Comments в папке models:
```php
<?php
class Comments extends CActiveRecord
{        
    public $verifyCode;
    public $guestlogin = "Гость";
    
    public static function model($className=__CLASS__)
    {
        return parent::model($className);
    }
    public function tableName()
    {
        return 'comments';
    }
    public function relations()
    {
        return array(
            // делаем привязку комментария к пользователю. что бы иметь возможность
            // получить его логин вместо id_user
            'user' => array(self::BELONGS_TO, 'User', 'id_user'),
        );
    }
}
```

Тут вы должны были увидеть такую вещь с которой мы раньше НЕ работали. В relations задаются связи нашей модели с другими моделями. Для лучшего понимания скажу что это некий JOIN который будет выполняться автоматически когда мы делаем select.

В данном случае я связываю комментарий к моделью User по полю id_user. Мне это нужно при выводе комментариев (что бы знать текущий логин пользователя который оставил комментарий вместо его id . Подробнее прочитать про relations вы может в статье Реляционная Active Record [рус.]

Теперь нам надо бы добавить safeAttributes (для защиты от хаЦкеров) и attributeLabels (для красоты отображения имен полей). Добавляем в Comments.php :


```php
       public function     attributeLabels()
    {
        return array(
            'verifyCode' => 'Код',
            'login'      => 'Имя',
        );
    }    
    public function safeAttributes()
    {
        return array('verifyCode', 'text', 'login');
    }
```


Ну а теперь пожалуй самое важное - rules. Правил для проверки данных у нас будет в принципе немного, но без них форма была бы очень не безопасна:


```php
 public function rules()
    {
        return array(
            // логин может содержать только англ и рус буквы, плюс цифры.
            array('login', 'match', 'pattern' =>'/^[A-Za-z0-9А-Яа-я\_\-\s,]+$/u','message' => 'Логин содержит недопустимые символы.'),
            // логин, пароль не должны быть больше 128-и символов, и меньше трёх
            array('login', 'length', 'max'=>128, 'min' => 3),
            // проверка капчи
            array('verifyCode', 'captcha', 'allowEmpty'=>!extension_loaded('gd')),
        );
    }
```


Хочу заметить что логин я проверяю по той причине что гость у нас в форме может написать всё что угодно.

Для удобства файл Comments.php в конечном итоге у вас должен выглядеть следующим образом:

```php
<?php
class Comments extends CActiveRecord
{        
    public $verifyCode;
    public $guestlogin = "Гость";
    
    public static function model($className=__CLASS__)
    {
        return parent::model($className);
    }
    public function tableName()
    {
        return 'comments';
    }
    public function relations()
    {
        return array(
            // делаем привязку комментария к пользователю. что бы иметь возможность
            // получить его логин вместо id_user
            'user' => array(self::BELONGS_TO, 'User', 'id_user'),
        );
    }
    public function attributeLabels()
    {
        return array(
            'verifyCode' => 'Код',
            'login'      => 'Имя',
        );
    }    
    public function safeAttributes()
    {
        return array('verifyCode', 'text', 'login');
    }
    public function rules()
    {
        return array(
            // логин может содержать только англ и рус буквы, плюс цифры.
            array('login', 'match', 'pattern' =>'/^[A-Za-z0-9А-Яа-я\_\-\s,]+$/u','message' => 'Логин содержит недопустимые символы.'),
            // логин, пароль не должны быть больше 128-и символов, и меньше трёх
            array('login', 'length', 'max'=>128, 'min' => 3),
            // проверка капчи
            array('verifyCode', 'captcha', 'allowEmpty'=>!extension_loaded('gd')),
        );
    }
}

```
### Пробуем

Теперь смело заходим к себе на localhost на страницу одной из тем, и проверяем как работаю комментарии.
Как это работает у меня - вы можете посмотреть на yii.dbhelp.ru

Вот и все. Надеюсь у вас получилось со всем разобраться.

Кстати хочу посоветовать вам одну отличную методику которой часто пользуюсь для усвоения материала. Первый раз прочитайте статью и выполняйте все действий. Когда поймете что вы в принципе поняли как это дело работает - просто удалите всё. Закройте блог, и попробуйте всё повторить самостоятельно. Если с первого раза не получиться - подсмотрите в блог, и попробуйте продолжить дальше самостоятельно.

 


Исходники
*_comments.php*
*_commentsForm.php*
*Comments.php*
*PostController.php*
*view.php*
*структура проекта + исходники урока (rar)*


