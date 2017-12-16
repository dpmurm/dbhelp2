# Урок 8 : Подключаем дизайн

![blog](/images/blog8/b7d144b36ec4764cbd6be3a30c4f844b.png)

Я вот подумал что пора привести наш блог в нормальный вид и прицепить к нему дизайн. Поэтому в этом уроке мы поговорим о том что такое темы, и попробуем подключить одну из них..


В [Yii фреймворке](http://www.yiiframework.com/) "темой" считается папка определенной структуры содержащая необходимые файлы для отображения дизайна. Это могут быть css/javascript файлы, макеты, файлы представлений и др. Все темы хранятся в папке themes в корне вашего приложения. Название папки темы и является её именем.

Пример структуры папки для темы test:
```php
test/
   views
    layouts
        main.php
   images
```
В папке views должны находиться переделанные под конкретную тему файлы представлений.

![blog](/images/blog8/801db9e4010fcdf4058a8597e0afdc57.jpg)

## Теория

Как вы помните сейчас весь дизайн нашего блога находиться в папке protected/views. Давайте попробуем переделать его в отдельную тему.

Заходим в папку themes и создаем там папку classic, а в ней папки: views, images. Теперь нам надо скопировать папку protected/views в папку themes/classic/views. После всех манипуляций папка themes должна выглядеть следующим образом:

```php
classic/
   views
    layouts
        main.php
    post
        ...
    user
        ...
   images
```

(я не расписывал файлы в папках post, user из-за их большого кол-ва, но они должны там быть).

Теперь в конфиге приложения нам надо указать тему по умолчанию. Для этого открываем файл config/main.php и добавляем туда строчку:

```php
... 
    'sourceLanguage' => 'ru',
    'language' => 'en',
    # -------------------------------
    'theme' => 'classic',
    # -------------------------------
...
```
Мы добавляем параметр «theme» с названием темы по умолчанию. В данном случае это «classic». Теперь давайте немного разберем как работают темы и для чего все таки нам папка protected/views.

Я надеюсь что на данном этапе вы произвели все изменения и попробовали войти на localhost. Если вы всё делали правильно — то визуально ничего не изменилось. На самом же деле теперь при подключении дизайна используются файлы из папки themes/classic/views, а не protecteed/views.

Это очень важный момент который вы должны понимать. Т.к. теперь для изменения каких-то элементов на странице — надо редактировать файлы темы, а не файлы из папки protected/views/

## Как это всё работает?

![blog](/images/blog8/6c438ca64c61662ce3258bdfc3d7e761.jpg)

Теперь давайте разберем зачем нам всё таки папка protected/views? Очень просто. Она является неким запасным вариантом. Давайте представим что у нас на сайте используется десять разных тем, и в каждой из тем есть одинаковая форма регистрации. Эта форма регистрации к примеру находиться в файле user/registration.php и полностью аналогичная для каждой из тем. Так вот что бы не пришлось дублировать этот файл десять раз для каждой из тем — мы можем просто поместить его в protected/views/user/registation.php. Если в приложении произойдет попытка загрузить отображение которого НЕТУ в папке текущей темы — оно автоматически загрузиться с protected/views/. Очень важно что бы вы это поняли, потому что дальше это поможет уменьшить дублирование кода.

## Качаем

Теперь давайте начнем прикручивать новый дизайн. Первым делом зайдем на страницу онлайн просмотра и нажмем кнопку «download zip». Когда архив будет закачен на ваш компьютер — разархивируйте его. Картинки из папки images поместите в папку themes/classic/images, а остальные файлы в папку  themes/classic/views. Структура нашей темы должна приобрести следующий вид:

```php
classic/
   views
        layouts
            main.php
        post
            ...
        user
            ...
        index.html
        license.txt
   images
         ...
   default.css
```
Фаил стилей находиться на одном уровне с папкой images специально что бы не заниматься прописыванием путей к картинкам в default.css. В идеальном варианте его конечно лучше поместить в отдельную папку css.

Как вы понимаете в файле index.html у нас находиться общий шаблон дизайна, его то мы и должны прицепить. Для этого открываем страницу нашего лаяута (layouts/main.php) и копируем в его начало весь файл index.html.

Файл  layouts/main.php отвечает за вывод общего макета для всех страниц нашего приложения (по умолчанию). Поэтому весь макет нового дизайна надо прикручивать именно к этому файлу

В конечном итоге файл layouts/main.php должен выглядеть вот так:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<!--
Design by Free CSS Templates
http://www.freecsstemplates.org
Released for free under a Creative Commons Attribution 2.5 License

Title      : Featuring
Version    : 1.0
Released   : 20090625
Description: A two-column fixed-width template suitable for small websites.
-->

<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="content-type" content="text/html; charset=utf-8" />
<title>Featuring by Free Css Templates</title>
<meta name="keywords" content="" />
<meta name="description" content="" />
<link href="default.css" rel="stylesheet" type="text/css" />
</head>
<body>
<div id="wrapper">
    <div id="header">
        <div id="logo">
            <h1><a href="#">Featuring</a></h1>
            <h2><a href="http://www.freecsstemplates.org/">By Free CSS Templates</a></h2>
        </div>
        <!-- end div#logo -->
        <div id="menu">
            <ul>
                <li class="active"><a href="#">Home</a></li>
                <li><a href="#">Products</a></li>
                <li><a href="#">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </div>
        <!-- end div#menu -->
    </div>
    <!-- end div#header -->
    <div id="page">
        <div id="page-bgtop">
            <div id="content">
                <div class="post">
                    <h2 class="title"><a href="#">Lorem Ipsum Dolor Volutpat</a></h2>
                    <p class="byline">Posted by FreeCssTemplates</p>
                    <div class="entry">
                        <p><img src="images/img06.jpg" alt="Yii Framework Blog img " width="564" height="294" /></p>
                        <p>This is <strong>Featuring</strong>, a free, fully standards-compliant CSS template designed by <a href="http://www.freecsstemplates.org/">Free CSS Templates</a>, released for free under the <a href="http://creativecommons.org/licenses/by/2.5/">Creative Commons Attribution 2.5</a> license.  The photo used in this template is from <a href="http://www.pdphoto.org">PDPhoto.org</a>. You're free to use this template for anything as long as you link back to <a href="http://www.freecsstemplates.org/">my site</a>. Enjoy :)</p>
                    </div>
                    <div class="meta">
                        <p class="links"><a href="#" class="comments">Comments (32)</a></p>
                    </div>
                </div>
                <div class="post">
                    <h2 class="title"><a href="#">Lorem Ipsum Dolor Volutpat</a></h2>
                    <p class="byline">Posted by FreeCssTemplates</p>
                    <div class="entry">
                        <p><img src="images/img06.jpg" alt="Yii Framework Blog img " width="564" height="294" /></p><p>Curabitur tellus. Phasellus tellus turpis, iaculis in, faucibus lobortis, posuere in, lorem. Donec a ante. Donec neque purus, adipiscing id, eleifend a, cursus vel, odio. Vivamus varius justo sit amet leo. Morbi sed libero. Vestibulum blandit augue at mi. Praesent fermentum lectus eget diam. Nam cursus, orci sit amet porttitor iaculis, ipsum massa aliquet nulla, non elementum mi elit a mauris.</p>
                    </div>
                    <div class="meta">
                        <p class="links"><a href="#" class="comments">Comments (32)</a></p>
                    </div>
                </div>
            </div>
            <!-- end div#content -->
            <div id="sidebar">
                <ul>
                    <li id="search">
                        <h2>Search</h2>
                        <form method="get" action="">
                            <fieldset>
                            <input type="text" id="seach-text" name="s" value="" />
                            <input type="submit" id="search-submit" value="Search" />
                            </fieldset>
                        </form>
                    </li>
                    <li>
                        <h2>Lorem Ipsum</h2>
                        <ul>
                            <li><a href="#">Fusce dui neque fringilla</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                        </ul>
                    </li>
                    <li>
                        <h2>Volutpat Dolore</h2>
                        <ul>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                        </ul>
                    </li>
                    <li>
                        <h2>Bebindum Mauris </h2>
                        <ul>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                        </ul>
                    </li>
                    <li>
                        <h2>Phasellus Tellus</h2>
                        <ul>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                        </ul>
                    </li>
                </ul>
            </div>
            <!-- end div#sidebar -->
            <div style="clear: both; height: 1px"></div>
        </div>
    </div>
    <!-- end div#page -->
    <div id="footer">
        <p id="legal">Copyright &copy; 2007 Featuring. All Rights Reserved. Designed by <a href="http://www.freecsstemplates.org/">Free CSS Templates</a>.</p>
        <p id="links"><a href="#">Privacy Policy</a> | <a href="#">Terms of Use</a></p>
    </div>
    <!-- end div#footer -->
</div>
<!-- end div#wrapper -->
</body>
</html>


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
Попробуйте загрузить теперь localhost. Должно отобразиться много текста и не какой графики (т.к. пути к css и картинкам мы еще не прописали). Если вы прокрутите страницу в самый низ — увидите список постов.

Теперь нам надо указать где находиться css файл, и все картинки. В отображении мы можем использовать конструкцию Yii::app()->theme->baseUrl которая возвращает путь к папке текущей темы. Его то мы и должны использовать перед подключением картинок (в img) и css/js файлов.

Файл main.php после некоторых изменений должен выглядеть вот так:

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<!--
Design by Free CSS Templates
http://www.freecsstemplates.org
Released for free under a Creative Commons Attribution 2.5 License

Title      : Featuring
Version    : 1.0
Released   : 20090625
Description: A two-column fixed-width template suitable for small websites.

-->
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="content-type" content="text/html; charset=utf-8" />
<title>Featuring by Free Css Templates</title>
<meta name="keywords" content="" />
<meta name="description" content="" />
<link href="<?php echo Yii::app()->theme->baseUrl; ?>/default.css" rel="stylesheet" type="text/css" />
</head>
<body>
<div id="wrapper">
    <div id="header">
        <div id="logo">
            <h1><a href="#">Featuring</a></h1>
            <h2><a href="http://www.freecsstemplates.org/">By Free CSS Templates</a></h2>
        </div>
        <!-- end div#logo -->
        <div id="menu">
            <ul>
                <li class="active"><a href="#">Home</a></li>
                <li><a href="#">Products</a></li>
                <li><a href="#">About</a></li>
                <li><a href="#">Contact</a></li>
            </ul>
        </div>
        <!-- end div#menu -->
    </div>
    <!-- end div#header -->
    <div id="page">
        <div id="page-bgtop">
            <div id="content">
                <div class="post">
                    <h2 class="title"><a href="#">Lorem Ipsum Dolor Volutpat</a></h2>
                    <p class="byline">Posted by FreeCssTemplates</p>
                    <div class="entry">
                        <p><img src="<?php echo Yii::app()->theme->baseUrl; ?>/images/img06.jpg" alt="Yii Framework Blog img " width="564" height="294" /></p>
                        <p>This is <strong>Featuring</strong>, a free, fully standards-compliant CSS template designed by <a href="http://www.freecsstemplates.org/">Free CSS Templates</a>, released for free under the <a href="http://creativecommons.org/licenses/by/2.5/">Creative Commons Attribution 2.5</a> license.  The photo used in this template is from <a href="http://www.pdphoto.org">PDPhoto.org</a>. You're free to use this template for anything as long as you link back to <a href="http://www.freecsstemplates.org/">my site</a>. Enjoy :)</p>
                    </div>
                    <div class="meta">
                        <p class="links"><a href="#" class="comments">Comments (32)</a></p>
                    </div>
                </div>
                <div class="post">
                    <h2 class="title"><a href="#">Lorem Ipsum Dolor Volutpat</a></h2>
                    <p class="byline">Posted by FreeCssTemplates</p>
                    <div class="entry">
                        <p><img src="<?php echo Yii::app()->theme->baseUrl; ?>/images/img06.jpg" alt="Yii Framework Blog img " width="564" height="294" /></p><p>Curabitur tellus. Phasellus tellus turpis, iaculis in, faucibus lobortis, posuere in, lorem. Donec a ante. Donec neque purus, adipiscing id, eleifend a, cursus vel, odio. Vivamus varius justo sit amet leo. Morbi sed libero. Vestibulum blandit augue at mi. Praesent fermentum lectus eget diam. Nam cursus, orci sit amet porttitor iaculis, ipsum massa aliquet nulla, non elementum mi elit a mauris.</p>
                    </div>
                    <div class="meta">
                        <p class="links"><a href="#" class="comments">Comments (32)</a></p>
                    </div>
                </div>
            </div>
            <!-- end div#content -->
            <div id="sidebar">
                <ul>
                    <li id="search">
                        <h2>Search</h2>
                        <form method="get" action="">
                            <fieldset>
                            <input type="text" id="seach-text" name="s" value="" />
                            <input type="submit" id="search-submit" value="Search" />
                            </fieldset>
                        </form>
                    </li>
                    <li>
                        <h2>Lorem Ipsum</h2>
                        <ul>
                            <li><a href="#">Fusce dui neque fringilla</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                        </ul>
                    </li>
                    <li>
                        <h2>Volutpat Dolore</h2>
                        <ul>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                        </ul>
                    </li>
                    <li>
                        <h2>Bebindum Mauris </h2>
                        <ul>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                        </ul>
                    </li>
                    <li>
                        <h2>Phasellus Tellus</h2>
                        <ul>
                            <li><a href="#">Eget tempor eget nonummy</a></li>
                            <li><a href="#">Nec metus sed donec</a></li>
                            <li><a href="#">Magna lacus bibendum mauris</a></li>
                            <li><a href="#">Velit semper nisi molestie</a></li>
                        </ul>
                    </li>
                </ul>
            </div>
            <!-- end div#sidebar -->
            <div style="clear: both; height: 1px"></div>
        </div>
    </div>
    <!-- end div#page -->
    <div id="footer">
        <p id="legal">Copyright &copy; 2007 Featuring. All Rights Reserved. Designed by <a href="http://www.freecsstemplates.org/">Free CSS Templates</a>.</p>
        <p id="links"><a href="#">Privacy Policy</a> | <a href="#">Terms of Use</a></p>
    </div>
    <!-- end div#footer -->
</div>
<!-- end div#wrapper -->
</body>
</html>


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
Теперь если вы загрузите localhost — увидите уже более приятное зрелище. Перед вами должна отобразиться страница дизайна с непонятными темами на английском (но зато с картинками), а в низу — список наших постов.

Следующее что мы сделаем:

1. Удалим вывод тем на англ. языке. Вместо этого на их место сделаем вывод нашего $content

2. Удалим ненужные пункты меню
3. Подключим в самый верх — наше меню
4. Поставим наш pageTitle

Смотрим что у меня получилось теперь из файла main.php:

```HTML
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<!--
Design by Free CSS Templates
http://www.freecsstemplates.org
Released for free under a Creative Commons Attribution 2.5 License

Title      : Featuring
Version    : 1.0
Released   : 20090625
Description: A two-column fixed-width template suitable for small websites.

-->
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<title><?php echo $this->pageTitle; ?></title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <meta name="language" content="ru" />
    <meta name="robots" content="all" />
<link href="<?php echo Yii::app()->theme->baseUrl; ?>/default.css" rel="stylesheet" type="text/css" />
</head>

<body>
<div id="wrapper">
    <div id="header">
        <div id="logo">
            <h1><a href="#">YiiBlog</a></h1>
            <h2><a href="http://www.freecsstemplates.org/">By www.DbHelp.ru</a></h2>
        </div>
        <!-- end div#logo -->
        <div id="menu">
            <ul>
                <li><a href="<?=Yii::app()->createUrl("post/index/");?>">Главная</a></li>
                <?php if (Yii::app()->user->isGuest): ?>
                    <li><a href="<?=Yii::app()->createUrl("user/registration/");?>">Регистрация</a></li>
                    <li><a href="<?=Yii::app()->createUrl("user/login/");?>">Вход</a></li>
                <?php else: ?>
                    <li><a href="<?=Yii::app()->createUrl("user/logout/");?>">Выход (<?=Yii::app()->user->name?>)</a>
                <?php endif; ?>
            </ul>
        </div>
        <!-- end div#menu -->
    </div>
    <!-- end div#header -->
    <div id="page">
        <div id="page-bgtop">
            <div id="content">
                <?php echo $content; ?>
            </div>
            <!-- end div#content -->
            <div id="sidebar">
                <ul>
                    <li>
                        <h2>Правый блок меню</h2>
                        <ul>
                            <li><a href="#">ссылка</a></li>
                            <li><a href="#">ссылка</a></li>
                            <li><a href="#">ссылка</a></li>
                            <li><a href="#">ссылка</a></li>
                        </ul>
                    </li>
                </ul>
            </div>
            <!-- end div#sidebar -->
            <div style="clear: both; height: 1px"></div>
        </div>
    </div>
    <!-- end div#page -->
    <div id="footer">
        <p id="legal">Designed by <a href="http://www.freecsstemplates.org/">Free CSS Templates</a>.</p>
    </div>
    <!-- end div#footer -->
</div>
<!-- end div#wrapper -->
</body>
</html>
```
Хочу вас порадовать — это последний вариант файла main.php :) Теперь если вы зайдете на localhost — увидите что наши посты теперь выводятся в нужном месте. Вот только в дизайн они немного плохо вписываются.

За вывод постов отвечают файлы index.php и _list.php. Как вы помните вывод постов у нас идет в табличном виде. А в дизайне который мы подключаем вывод сделан следующим образом:
```html
<div class="post">
<h2 class="title"><a href="#">тема</a></h2>
<p class="byline">правый текст</p>
<div class="entry">
    <p>текст</p>
</div>
<div class="meta">
    <p class="links"><a href="#" class="comments">ссылка</a></p>
</div>
</div>
```
Т.е. Все на дивах. Поэтому нам надо сделать точно также (что бы было красиво). Поэтому в файлах index.php и _list.php вырезаем весь табличный вывод и приводим его к примеру выше.

Файл index.php у вас должен выглядеть вот так:
```php
<?php
if (!empty($posts))
foreach ($posts as $key => $val) {
    $this->renderPartial('_list',array(
        'post'=>$val,
    )); 
}
?>
<?php $this->widget('CLinkPager',array(
            'pages'=>$pages, 
            'maxButtonCount' => 5, 
            'header' => '<b>Перейти к странице:</b><br><br>',
    )); ?>
```
А файл _list.php:
```html
<div class="post">
<h2 class="title"><a href="<?=$this->createUrl('post/view/', array('url' => $post->url));?>"><?=$post->name;?></a></h2>
<p class="byline"><?php echo $post->created;?></p>
<div class="entry">
    <p><?php echo mb_substr($post->text, 0, 150), "...";?></p>
</div>
<div class="meta">
    <p class="links"><a href="<?=$this->createUrl('post/view/', array('url' => $post->url));?>">Подробнее</a></p>
</div>
</div>
```
![blog](/images/blog8/92a5db542db01b7f56f01b6fa9b1481d.jpg)

Я надеюсь что до этого момента всё было понятно. Теперь заходим на страницу localhost и радуемся :) Если у вас всё получилось правильно — перед вами будет главная страница уже в новом дизайне. При этом темы будут выводиться красиво в нужном месте :) Попробуйте меню, проверьте всё ли работает. Страница регистрации и авторизации должна была автоматически стать под новый дизайн.

После того как я зашел на страницу подробного описания поста — мне не понравился наш вывод рамки. Сильно грузит, неправда ли? Поэтому я решил border приравнять нулю.

Для этого я открыл файл view.php и строчку:

```html
<table border="1" width="100%">
```

заменил на :

```html
<table border="0" width="100%">
```

## Результат

Теперь смело можем говорить что у нас всё получилось. Сам материал который был описан в этом уроке лёгкий для понимания, но тяжёлый для объяснения. Я постарался описать как можно подробнее что бы вам было понятно, но уверен что получилось все равно не идеально. Поэтому если будут вопросы — не стесняйтесь, пишите в комментариях или через форму «Обратная связь». Также вы можете прочитать статью [«Темы оформления»](http://www.yiiframework.com/doc/guide/1.1/ru/topics.theming) из официальной документации (правда там очень не много информации).

Пока что наше приложение будет использовать одну тему, поэтому рассматривать варианты переключения между темами я не буду (в этом уроке). Это очень легко и я уверен вы сами поймете как это сделать (параметр theme).

![blog](/images/blog8/b7d144b36ec4764cbd6be3a30c4f844b.png)

![blog](/images/blog8/6c5dea1d9f56fc999445c6669b71e916.png)


![blog](/images/blog8/9c12407246fbb250ac8fb758d358b1a7.png)

## Задание на дом

Для закрепления материала я очень советую вам найти понравившийся дизайн и попробовать его самостоятельно прикрутить к приложению. Все ваши темы вы можете посылать мне на zolter.od@gmail.com и я добавлю их в этот пост для других пользователей.

**Ссылки**

- Живой просмотр блога

- Скачать отдельно тему
- Скачать архив с исходниками всего блога + исходники текущего урока