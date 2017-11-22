Шаг 5: Контроллер Д/З

Рассмотрим правильный вариант решения задачи поставленной в статье "Шаг 5: Контроллер".
---

Тем кто читает все статьи за один раз и не старается выполнять упражнения самостоятельно я искрение сочувствую. Прошу вас еще раз пересмотреть свое отношение к познанию материала и все таки попробовать посидеть N-ое кол-во времени и самостоятельно пораскинуть мозгами, учитывая что все материалы для решения вы уже получили.

Тем кто выполнил упражнение своими силами и теперь хочет сверится с моим вариантом - прошу читать дальше.

Первым делом я открыл phpMyAdmin (обычно этот скрипт для удобной работы с базой уже установлен у вас на хостинге) и создал таблицу для новостей следующим SQL:

 CREATE TABLE `news` (
`id` INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY ,
`url` VARCHAR( 100 ) NULL ,
`title` VARCHAR( 255 ) NOT NULL ,
`body` TEXT NOT NULL
) ENGINE = MYISAM

ALTER TABLE `news` ADD UNIQUE(`url`)



Я делал только основные поля что бы вы поняли как это работает. После этого я заполняю базу двумя новостями:

INSERT  INTO `news` (`id`, `url`, `title`, `body`) VALUES (NULL,  'my-super-news1', 'Тестовая новость 1', 'текст новости 1 текст новости 1  текст новости 1 текст новости 1 текст новости 1 текст новости 1 '),  (NULL, 'super-puper-news2', 'Тестовая новость 2', 'текст новости 2 текст  новости 2 ');



Идея в следующем: пользователь который заходит по ссылке www.mysite.ru/news/my-super-news/ - получает на экране текст первой новости, а если зайдет на www.mysite.ru/news/super-puper-news2 тогда получает текст второй.

Создаем модель (models/News.php) для работы с нашей таблицей новостей:

<?php
class News extends CActiveRecord
{
    public static function model($className=__CLASS__)
    {
        return parent::model($className);
    }
    public function tableName()
    {
        return 'news';
    }
}

Модернизируем наш контроллер под работу с базой (controllers/HelloController.php)

<?php
class HelloController extends CController
{
   public function actionWorld()
   {
       echo 'Привет от HelloController->actionWorld()';
   }
   public function actionStepan()
   {
       echo 'Привет Степан!';
   }
   public function actionDima()
   {
       echo 'Привет Дима!';
   }
   public function actionIvan()
   {
       echo 'Привет Иван!';
   }  
   public function actionNews()
   {
         // если хотите, то $_GET['news_name'] можете обработать
         // предварительно всякими функциями для безопасноти
      $url = $_GET['news_name'];
      // проверяем есть ли новость с таким url
      if ($news = News::model()->findByAttributes(array('url' => $url))) {
          // получается мы нашли в базе запись
          // в которой значение url равняется переменной news_name
          echo $news->title ,  '<br>' , $news->body;
      } else {
          // нет такой новости, об этом и сообщаем:
        echo 'новость не существует!';
      }
   }
}

Вот и все. Заходите по тестовым ссылкам выше и наслаждайтесь работой.
Если даже после примера остались вопросы - рекомендую перечитать урок по работе с моделью.

