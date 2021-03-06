# Как работать с флеш-сообщениями

Флеш-сообщения используются для того, что бы сохранить в сессии некий текст, и после отображения его пользователю - сразу удалить.

**Понятие "флеш-сообщение" никак не связано с технологие Flash от Adobe**


Таким образом флеш-сообщения доступны только в текущем и следующем запросе. Флеш-сообщения могут быть установлены при помощи метода setFlash() класса CWebUser.

Например, вы хотите сообщить пользователю что "его пароль успешно изменен", для в нужном экшинсе контроллера добавляем следующий код:

```php
Yii::app()->user->setFlash('success',"Пароль изменен");
```

Таким образом мы сохраняем в сессии переменную "success" со значением "Пароль изменен". Если вы хотите использовать больше чем одно флеш-сообщение - просто используйте уникальные значения "имени".

Теперь в файле отображения (там же где находится наша форма изменения пароля, к примеру) добавляем код проверки на наличие флеш-сообщения и его отображения:
```php
<?php if(Yii::app()->user->hasFlash('success')):
    echo Yii::app()->user->getFlash('success'); 
endif; ?>
```
Для проверки существования сообщения мы использовали метод hasFlash(), для получения текста сообщения - getFlash();

Таким образом после того как флеш-сообщение было созданно в контроллере - оно будет показано в отображении.

Теперь добавим немного "красоты". Обернем наше сообщение в класс "info" что бы использовать эффект плавного затухания. Получим:
```php
<?php if(Yii::app()->user->hasFlash('success')):?>
<div class="info">
    <?php echo Yii::app()->user->getFlash('success'); ?>
</div>
<?php endif; ?>
```
В экшинсе после создания флеш сообщения, добавим следующий код:

```php
Yii::app()->clientScript->registerScript(
   'myHideEffect',
   '$(".info").animate({opacity: 1.0}, 3000).fadeOut("slow");',
   CClientScript::POS_READY
);
```

По этому примеру вы можете использовать и другие возможности jQuery или javascript для обработки сообщения.

## Реальный пример использования

Контроллер:
```php
<?php
public function actionReg() {
    $model = new User('reg');
            
        if (!empty($_POST['User'])) {
            // получили данные с формы
            $model->attributes = $_POST['User'];               

            if ($model->validate()) {
                
                $message = "Спасибо за регистрацию!";
                $this->sendHtmlMail(...);

                $model->insert();
            
                // данные в базу добавлены, можна и флеш-сообщение показать:
                
                Yii::app()->clientScript->registerScript(
                   'myHideEffect',
                   '$(".flash-success").animate({opacity: 1.0}, 5000).fadeOut("slow");',
                   CClientScript::POS_READY
                );

                Yii::app()->user->setFlash('success',"На указанный email было отправленно письмо для подтверждения регистрации!");
                
            } else {
                // просто выводим форму
            }
        }
        
    $this->render('reg', array('model' => $model));
}
```

Отображение (view/controller/reg):
```html
<h1>Регистрация</h1>
<?php if(Yii::app()->user->hasFlash('success')):?>
<div class="flash-success">
    <?php echo Yii::app()->user->getFlash('success'); ?>
</div>
<?php endif; ?>
```
Дальше идет код формы регистрации...

Ссылки:

[How to work with flash messages](http://www.yiiframework.com/wiki/21/) [англ]
