Валидация формы средствами Yii

Эта статья устарела т.к. была написана для yii версии 1.0.х; Если вы используете более новую версию - у вас могут возникнуть ошибки из-за несовместимости. Обычно ответы на все вопросы работы на 1.1.х были описаны в комментариях ниже статьи. 

Сегодня мы будем учиться делать форму с валидацией на Yii. Хочу сразу заметить что сам фреймворк даёт нам огромный функционал для валидации и создания наших веб форм.

Давайте разберем с вами как на Yii можно создать простую форму для регистрации (поле логин, пароль) и проверить данные с формы перед тем как добавить в таблицу.
---

Для начала договоримся что у нас в базе должна быть табличка user с которой мы и будем работать:

CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `login` varchar(250) NOT NULL,
  `passwd` varchar(250) NOT NULL,
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=11 ;

Как видите она содержит в себе всего три поля: номер записи, логин, пароль.

Теперь нам надо создать модель Users.php:

class Users extends CActiveRecord
    {        
        public $passwd2;
            
            public static function model($className=__CLASS__)
            {
                return parent::model($className);
            }
            public function tableName()
            {
                return 'user';
            }
            public function rules()
            {
                return array(
                );
            }
            public function attributeLabels()
            {
                return array(
                    'login'    => 'Логин',
                    'passwd' => 'Пароль',
                );
            }    
            public function safeAttributes()
            {
                  return array(
                    'login',
                    'passwd',
                    'passwd2',
                  );
            }
    }



Тут у нас тоже ничего необычного:

    В tableName я указал что мы используем таблицу с именем «User»
    В attributeLabels указал синонимы для полей таблицы. Это удобно что б наша форма выглядела по человечески. Т.е. в форме теперь вместо «login» у нас будет «Логин» и т.п. Если непонятно зачем это надо и что оно делает – удалите и посмотрите какой вид приймет ваша форма.
    Также я переопределил метод «rules» который содержит в себе правила валидации. Пока что метод пустой т.к. правила валидации мы рассмотрим чуть дальше.
    safeAttributes– это метод который позваляет нам указать название полей таблицы которые мы допускаем для изменения через форму.  Данный метод необходим для безопасности и не стоит некогда его упускать.

Теперь в принципе когда модель у нас есть – надо приступить к созданию формы и контроллера.

Заходим в protected/controllers и создаем там UserController.php:


    class UserController extends CController
    {
        public function actionIndex()
        {
            $form = new Users();
            
            if (!empty($_POST['Users'])) 
            {
                $form->attributes=$_POST['Users'];
                
                if($form->validate()) {
                    // Валидация прошла успешно
                    // Добавляем данные в базу
                    $form->save();
                }
            }
            $this->render('reg', array('form' => $form));
        }
    }

Для простоты и удобства я создал один экшинс который будет выводить на экран нам форму для регистрации, и если данные из формы были получены – тут же будет проверять их.

Хочу обратить ваше внимание на следующую конструкцию

$form->attributes=$_POST['Users'];

Данная запись выполняет так сказать массовое присваивание элементов форм экземпляру класса модели ($form). Т.е. простыми словами одна эта строчка совершает следующие действия:

foreach($_POST['Users'] as $name=>$value)
{
   $form ->$name=$value;
}

Еще момент, метод validate проверяет удволитворяют ли данные $form условиям валидации. Если данные нарушают хоть одно из правил – форма не будет сохранена, а на экран будут показы ошибки.

Идем дальше, создаем форму в отображении. Для этого заходим в protected/views/user и создаем там файлик reg.php:

<?php echo CHtml::form(); ?>
<?php echo CHtml::errorSummary($form); ?><br>

    <table border="0" width="400" cellpadding="10" cellspacing="10">
    
    <tr>
        <td width="150"><?php echo CHtml::activeLabel($form, 'login'); ?>
        <td><?php echo CHtml::activeTextField($form, 'login') ?>
        
    <tr>
        <td><?php echo CHtml::activeLabel($form, 'passwd'); ?>        
        <td><?php echo CHtml::activePasswordField($form, 'passwd') ?>
        
    <tr>
        <td><?php echo CHtml::activeLabel($form, 'passwd2'); ?>
        <td><?php echo CHtml::activePasswordField($form, 'passwd2') ?>
        
    <tr>
        <td>
        <td><?php echo CHtml::submitButton('Войти'); ?>
    
<?php echo CHtml::endForm(); ?>

Как вы заметили вместо создания элементов на чистом Html-е мы используем класс помошника CHtml. Данная статья не включает в себя описание работы класса CHtml но в двух словах расскажу что к чему:

    activeTextField – создает текстовое поле (input type=text)
    activePasswordField – создает поле типа password (input type=password)
    submitButton – создаем субмит кнопочку
    errorSummary – место где будут выводиться ошибки валидации (если такие будут)
    form…endForm – начало и конец нашей формы

Yii Framework Blog img http://dbhelp.ruformvalidation2 Вот в принципе у нас всё и готово (кроме правил валидации). Давайте теперь зайдем на страницу с нашей формой и посмотрим что получилось (user/index). Если вы заполните форму и нажмете «регистрация» – вы добавите нового пользователя в таблицу.

Теперь давайте придумаем правила (rules) для нашей формы по которым мы будем контролировать данные которые нам ввели:

    Поле логин, пароль и повтор пароля не должны быть пустыми
    Поле пароль должно совпадать с полем повторить пароль
    Поле логин и пароль должны быть больше трёх символов.
    Поле логин также должно быть не больше десяти символов.

Заходим в нашу модель protected/models/Users.php:
И в метод rules добавялем по порядку правила которые мы придумали выше.

    array('login, passwd', 'required'),

    в первом параметре мы перечисляем название полей формы, вторым параметром – способ валидации. В данном случае 'required' говорит о том что наши два поля не должны быть пустыми

    array('passwd', 'compare', 'compareAttribute'=>'passwd2'),


    таким образом мы указываем что поле passwd должно быть равно с passwd2.

    array('login,passwd', 'length', 'min' => 3),
    array('login ', 'length', 'max' => 10),


Теперь если сгруппировать это всё вместе то метод rules должен выглядеть следующим образом:

public function rules()
{
    return array(
        array('login, passwd, passwd2', 'required'),
        array('passwd', 'compare', 'compareAttribute'=>'passwd2'),
        array('login, passwd', 'length', 'min' => 3),
        array('login ', 'length', 'max' => 10),
    );
}

Больше нам с вами некакого кода писать не надо. Теперь при нажатии кнопки «регистрации» в нашем контроллере отработает строчка:

if ($form->validate) {

Которая в случае корректно введенных данных – сохранит их в базу, а в случае если данные не будут удволитворять правилам  – выведет ошибки на экран.

Надеюсь данная статья оказалась вам полезной.

Пользуйтесь Yii и будет вам счастье =)
