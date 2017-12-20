# Урок 4 : Создаем отдельную страницу для постов (тем)

Как и обещал сегодня мы поговорим как для постов (тем) создать свою отдельную страницу. Тема этого урока как вы понимаете очень проста, и для человека немного знакомого с Yii - не актуальна. В любом случае выбрасывать момент создания отдельной страницы я не могу, поэтому кому интересно - читаем дальше...

Я надеюсь вы уже знакомы с уроком "Первый раз, первый контроллер.." и имеете на компьютере (хостинге) рабочую копию нашего проекта.

## Начнем

Для создания отдельной страницы для поста я могу выбрать несколько путей:

1. Расширить actionIndex дописав в нем проверку не висит ли в GET-е где то Id.

2. Создать отдельный экшинс

Давайте выберем второй путь чтобы не засорять actionIndex.

Откройте файл PostController.php и добавьте в него два новых экшинса:

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
                $post = Posts::model()->find("url = :url", array(
                            ':url' => $url,
                        ));
                if (!empty($post)) {
                    // Тема есть в базе
                    $this->render('view', array(
                        'post' => $post,
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

    public function actionError404 ()
    {
        $this->render('404');
    }
```

Разберем что у нас и для чего:

1. actionView - проверяет есть ли в $_GET['url'] какое то значение. Если есть - шуршым в базе и находим нужную тему. Если поста с таким url нету - выполняем экшинс actionError404.

2. actionError404 - просто рендерим на экран файл 404.php в котором будет описано мол "Такой страницы у нас нет"

Теперь давайте в views/post и создадим там два файла:

404.php:
```html
<h1>Такой страницы у нас нет</h1>
```

view.php:

```html
<table border="1" width="100%">


<?php if (!empty($post)) : ?>
    <tr>
        <td><h1><?=$post->name;?></h1>
    <tr>
        <td><?=$post->created;?>
    <tr>
        <td><?=$post->text;?>
<?php endif; ?>


</table>
```
Вот теперь в принципе всё готово.


Так как у вас в базе уже есть пару постов (http://yii.dbhelp.ru/step/?n=6), то мы можем попробовать обратиться к одному из них. Для этого в браузере загружаем localhost/post/view/?url=test и смотрим на результат. Если вы всё делали правильно, у вас должно было получиться что то вроде этого http://yii.dbhelp.ru/post/view/?url=hello-yii

Как мне кажется передавать в URL-е таким образом название поста - не красиво, поэтому надо написать простой маршрут. Подробнее почитать как организовать ЧПУ (человеко понятные урлы) на вашем сайте смотрите вот тут

Заходим в protected/config/main.php и добавляем:

```php
'components'=>array(
...   
    'urlManager'=>array(
        'showScriptName' => false,  // что бы не цеплялся index.php к ссылкам
        'urlFormat'=>'path',
        'rules'=>array(
            '<url>/post/'=>'post/view',
        ),
    ), 
 ...
```

**Обратите внимательно куда именно я добавил правила, не надо лепить просто в любое место!**
Теперь наше правило позволяет нам вместо localhost/post/view/?url=test писать localhost/test/post/

Можете по тестировать, всё должно работать.

Если вдруг не работает и к адресу добавляется "index.php" тогда,
открываем .htaccess (в корне сайта) и пишем туда:

```php
Options +FollowSymLinks
IndexIgnore */*
RewriteEngine on

# if a directory or a file exists, use it directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d

# otherwise forward it to index.php
RewriteRule . index.php
```
Еще одна мелочь которую мы с вами не написали - это ссылку с главной страницы на страницу поста. Т.е. на странице localhost/post/index/ надо сделать редирект на страницу поста, при нажатии на название.

Открываем файл views/post/_list.php
```html
<tr><td>

<table border="1" width="100%">
    <tr><td><h2>Название : <?=$post->name;?></h2>
    <tr><td><?php echo $post->created;?>
    <tr><td><?php echo mb_substr($post->text, 0, 500), "...";?>
</table>
```

и немного его меняем:

```html
<tr><td>
<table border="1" width="100%">
    <tr>
        <td><h2>
            <?php echo CHtml::link($post->name,array('post/view/', 'url'=>$post->url)); ?>
            </h2>
    <tr><td><?php echo $post->created;?>
    <tr><td><?php echo mb_substr($post->text, 0, 500), "...";?>
</table>
```

п.с. такой перенос строчек сделан специально что б они не вылезли за блок code моего блога.

Теперь заходим на главную страницу (localhost/ или localhost/post) и нажимаем на название поста.
Барабанная дробь, тун-турурун-турурурурн, все работает :)

Вот такая вот простенькая статья описала вам процесс создания отдельных страниц для постов.
