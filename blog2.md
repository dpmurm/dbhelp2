Урок2 : Первичная настройка

Вот и начинается наконец то серьезная работа по написанию документации по блогу. Весь вчерашний день я был занят написанием небольших статей по компонентам и багам, но сегодня я постараюсь изложить побольше материала относительно блога.

Урок 2 : "Первичная настройка" - расскажет вам: о том какую структуру базы данных я подобрал для своего блога, как подключится к mySQL базе данных и многое другое.
---
Структура базы данных

Для блога я использовал базу данных mySQL и несложную, практически стандартную, структуру для блога. В данный момент он не содержит в себе всего того функционала который я хотел в него вложить и постепенно он будет пополнятся. На момент написания данного урока - в базе данных у меня 4 таблицы.

Таблица user

Эта таблица нам нужна для хранения зарегистрированных пользователей блога. Содержит она в себе поле логина, пароля и дату регистрации.

Дамп:

CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `login` varchar(250) NOT NULL,
  `passwd` varchar(250) NOT NULL,
  `created` datetime NOT NULL,
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=2 ;

Таблица post

Может считаться сердцем блога. Данная таблица содержит в себе все темы блога.

Дамп:

CREATE TABLE `post` (
  `id` int(11) NOT NULL auto_increment,
  `id_user` int(11) NOT NULL,
  `id_category` int(11) NOT NULL,
  `name` varchar(250) NOT NULL,
  `url` varchar(100) NOT NULL,
  `text` text NOT NULL,
  `active` tinyint(4) NOT NULL default '1',
  `created` datetime NOT NULL,
  PRIMARY KEY  (`id`),
  UNIQUE KEY `url` (`url`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=20 ;

Обьясненеи полей:
id_user - порядковый номер создателя сообщения.
id_category - порядковый номер категории к которой относится сообщение.
name - название или тема сообщения
url - адрес сообщения
text - сам текст сообщения
active - флаг который говорит нам скрыто сообщение или нет
created - дата создания сообщения

Таблица category

Содержит в себе простой перечень категорий - и их адресов. Дамп таблицы выглядит следующим образом:

CREATE TABLE `category` (
  `id` int(11) NOT NULL auto_increment,
  `name` varchar(250) NOT NULL,
  `url` varchar(100) NOT NULL,
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=14 ;

Всё просто и думаю в разборе не нуждается.

Таблица comments

Хранит в себе комментарии пользователей блога. На момент написания данного урока - блог не принимает сообщения от незарегистрированных пользователей, но таблица создавалась на будущее и имеет ориентированные поля на данную функцию. Что это значит, объясняю, таблица содержит в себе поле login и поле id_user. Считалось что если в будущем будет дописана возможность незарегистрированным пользователям оставлять комментарии то поле логин - будет заполнено указанным именем гостя, а поле id_user = 0.

Дамп:

CREATE TABLE `comments` (
  `id` int(11) NOT NULL auto_increment,
  `id_post` int(11) NOT NULL,
  `id_user` int(11) NOT NULL,
  `login` varchar(250) NOT NULL,
  `text` text NOT NULL,
  `created` datetime NOT NULL,
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 AUTO_INCREMENT=7 ;

id_post - номер сообщения к которому относится данный комментарий
id_user - номер автора комментария. 0 - если автор не зарегистрирован
login - логин автора сообщения. используется если автор сообщения - не зарегистрирован
text - текст сообщения
created - дата создания сообщения
Итог по структуре

Как вы видите структура базы очень простая и понятна. Все достаточно просто и с небольшим намёком на будущий функционал. Надеюсь проблем на данном этапе у вас никаких не возникло. Вам необходимо используя приведенные мною дампы - создать их у себя в базе данных. У меня база называется "zolter_blog", у вас может называться как то по другому.
Подключение Yii к MySQL

Теперь давайте перейдем к подключение Yii к нашей базе данных. Для этого нам надо войти в конфиг файл protected/config/main.php и найти там строчки отвечающие за подключение к базе данных:

        'db'=>array(
              .....
        ),

Всё что внутри данного массива вы можете удалить. После этого вы можете превратить его в нечто похожее как у меня:

        'db'=>array(
            'class'=>'system.db.CDbConnection',
            'connectionString'=>'mysql:host=localhost;dbname=zolter_blog',
            'username'=>'zolter_admin',
            'password'=>'mypassworld',
            'charset'=>'utf8'
        ),

И так что мы видим:

connectionString - содержит в себе строку DSN после host= вы можете указать адрес вашего сервера mySQL если он отличный от localhost. После dbname = вам следует указать имя вашей базы данных. Помните что чаще всего на хостингах если вы создаете базу данных test то автоматически с переди подставляется префикс вашего логина. И ваша база имеет имя не test а youlogin_test.

username - имя пользователя для работы с нашей базой. данный пользователь должен быть создан вами вместе с базой данных и обладать правами администратора (чтение/запись и удаление).

password - соотвественно пароль пользователя.

charset - кодировка соединения с базой данных. настоятельно рекомендую использовать всегда utf для ваших проектов. Yii отлично работает с данной кодировкой и проблем возникнуть не должно.

Вот в принципе и все действия которые необходимо произвести для подключения к mySQL базе данных.
Каталоги и файлы

Для создания блога я использовал стандартную структуру приложения Yii. Я создал первое приложение Yii по инструкции (читать), затем удалил все файлы внутри папок оставив только общую структуру.

У себя на хостинге файлы я разместил следующим образом:

(папка моего аккаунта)
      | -- dbhelp.ru (мой домен)
            |-- assets
            |- -css
            |-- images
            |-- js
            |-- protected
            |-- storage
            |-- themes
            |-- index.php
      | -- logs (системная папка хостинга)
            |-- ...
      | -- yii (папка с yii фреймворком)
            |-- framework
            |-- requirements
            |-- ...

Этим я добился что папка Yii находится за приделами домена и недоступна из интернета через браузер. Структуру папок блога вы можете скачать в конце данного урока (если вам лень повторить пример по созданию первого приложения на Yii).
index.php

Со структурой думаю всем все понятно. Теперь нам надо открыть файлик index.php и прописать в нем пути к фреймворку. У меня на хостинге это выглядит вот так, но у вас естественно имена папок будут другими:


// change the following paths if necessary
$yii='/hsphere/local/home/zolter/yii/framework/yii.php';
$config=dirname(__FILE__).'/protected/config/main.php';

// remove the following line when in production mode
defined('YII_DEBUG') or define('YII_DEBUG',true);

require_once($yii);
Yii::createWebApplication($config)->run();

Подводим итоги..

В этом уроке мы разобрали структуру базы данных которая будет использоваться в блоге. Также в качестве примера мы рассмотрели как подключать наше приложение к базе данных MySQL. Я надеюсь вы следовали за уроком и в базе данных у вас уже созданы необходимые таблицы.

Теперь вы можете скачать структуру каталогов блога о которой мы говорили выше. Она не содержит в себе файлов контроллера и моделей, всё это будет в следующем уроке. Пока что пожалуйста просто скопируйте архив и установите его у себя на локальной машине или хостинге. Пропишите в index.php файле путь к Yii и переходите к следующему уроку.

Архив со структурой папок вы можете скачать по адресу - http://dbhelp.ru/files/blog-folders.rar

В следующем уроке мы рассмотрим с вами как создавать простой функционал для нашего блога! 