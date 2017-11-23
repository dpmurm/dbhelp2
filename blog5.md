Урок 5 : Многоязычность

Я надеюсь вы согласитесь со мной что каждый серьезный сайт (проект) должен поддерживать многоязычность. Мы с вами не имеем право навязывать пользователю язык на котором он должен смотреть наш сайт. Пока наш блог проект не зашел слишком далеко - стоит оглянуться и заняться интернационализацией...
---

А вы уже читали "Урок 4 : Создаем отдельную страницу для постов (тем)"? Если нет, тогда дальше читать не стоит :)

Хочу обратить внимание что Yii отлично поддерживает интернационализацию (I18N) и что перевести проект на другие языки проблем не составит.

Первым делом в конфиг файле  надо указать текущий язык скриптов проекта (на каком языке у нас будут написаны исходники), ну и сразу можем указать какой язык на сайте будет по умолчанию.

Заходим в protected/config/main.php и добавляем две строчки:

<?php
 return array(
...
     'sourceLanguage' => 'ru',
     'language' => 'en',
...

    sourceLanguage - язык скриптов.
    language - язык нашего сайта по умолчанию

Я выбрал английский язык по умолчанию чтобы видеть разницу в проделанной работе.

Теперь заходим в protected и создаем там папку messages. В папке messages создаем папку en, а в папке en создаем файл main.php:

<?php
     return array(
         'русский вариант' => 'английский вариант',
     );

В этом самом файле мы и делаем все переводы нашего сайта на английский язык. К примеру сюда же кидаются переводы для кнопочек, элементов форм и прочей фигни. В народе файл messages/en/main.php называют словарём. Таких словарей у нас может быть много и использовать их будем в зависимости от ситуации.

Я взял на себя смелость немного перевести элементы блога и файл messages/en/main.php у меня теперь выглядит следующим образом:

<?php
     return array(
         'Перейти к странице:' => 'Select page:',
     );

Так получилось что в тех скриптах блога что мы с вами уже написали - только одна русская фраза фигурирует в отображении (post/index.php), её я и перевёл. Все остальные наши данные выводятся с базы данных и в переводе на данном этапе не нуждаются.

Теперь давайте откроем файл views/post/index.php и укажем там что фразу "Перейти к странице" надо переводить (если язык пользователя не русский).

Все переводы осуществляются одной командой:

Yii::t("словарь", "что_переводим")

Как вы понимаете что б перевести фразу "Перейти к странице:" нам достаточно указать:

Yii::t("main", "Перейти к странице:");

    Первым параметром мы указали название нашего рабочего словаря (messages/en/main.php)
    Второй параметр - это фраза которую надо переводить.

Теперь после небольшой теории - переходим к практике.

Файл views/post/index.php до интернационализации:

 <table border="0" width="100%" cellpadding="10" cellspacing="10">
     <?php
     if (!empty($posts))
         foreach ($posts as $key => $val) {
             $this->renderPartial('_list',array(
                 'post'=>$val,
             )); 
         }
     ?>
 </table>
 
 <?php $this->widget('CLinkPager',array(
             'pages'=>$pages, 
             'maxButtonCount' => 5, 
             'header' => '<b>Перейти к странице:</b><br><br>',
     )); ?>


Файл views/post/index.php после интернационализации:

 <table border="0" width="100%" cellpadding="10" cellspacing="10">
     <?php
     if (!empty($posts))
         foreach ($posts as $key => $val) {
             $this->renderPartial('_list',array(
                 'post'=>$val,
             )); 
         }
     ?>
 </table>
 
 <?php $this->widget('CLinkPager',array(
             'pages'=>$pages, 
             'maxButtonCount' => 5, 
             'header' => '<b>'.Yii::t("main", "Перейти к странице:").'</b><br><br>',
     )); ?>


Изменилась всего одна строчка. Старая фраза просто обернулась методом Yii::t.

На этом наше с вами участие заканчивается. Дальше Yii сама проверяет какой язык пользователем был выбран на сайте.
Давайте рассмотрим как работает на простых примерах

    Если язык пользователя совпадает с языком скриптов (указанно в config/main.php) - тогда перевод не осуществляется. Т.е. на экран выводиться просто фраза которая указана вторым параметром в Yii::t.
    Если язык пользователя английский - Yii открывает словарь main (messages/en/main.php) и ищет в нём фразу указанную вторым параметром. Если такой фразы не найдено - выводиться оригинальное название на родном языке. Если фраза найдена - значит она и подставляется при выводе.
    Если у пользователя выбран язык для которого мы не писали перевода (к примеру язык "ua", а папки messages/ua нет) тогда на экран будет выведена фраза на оригинальном языке скриптов

Вот в принципе и все премудрости которые надо знать.
В дальнейшем при разработке я сразу буду использовать Yii::t, так что больше к тому "откуда растут ноги" мы возвращаться не будем.

Вкратце про интернационализацию я уже писал тут, но еще больше вы можете прочитать в официальной документации на русском языке!