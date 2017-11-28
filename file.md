# Как загружать файлы с помощью модели

Первым делом необходимо объявить переменную в моделе для хранения имени файла . Также стоит незабыть указать правило валидации (в rules) для нашего поля. Там вы можете задать расширения файлов которые могут быть загружены.


class Item extends CActiveRecord
{
    public $image;
    // ... other attributes
 
    public function rules()
    {
        return array(
            array('image', 'file', 'types'=>'jpg, gif, png'),
        );
    }
}

После этого в контроллере стоит обьявить экшинс который будет загружать форму и обрабатывать данные из неё.

class ItemController extends CController
{
    public function actionCreate()
    {
        $model=new Item;
        if(isset($_POST['Item']))
        {
            $model->attributes=$_POST['Item'];
            $model->image=CUploadedFile::getInstance($model,'image');
            if($model->save())
            {
                $model->image->saveAs('path/to/localFile');
                // redirect to success page
            }
        }
        $this->render('create', array('model'=>$model));
    }
}

И наконец, создание файла отображения и добавление в него элемента для загрузки файла.

<?php echo CHtml::form('','post',array('enctype'=>'multipart/form-data')); ?>
...
<?php echo CHtml::activeFileField($model, 'image'); ?>
...
</form>

Данная статья является мои переводом http://www.yiiframework.com/doc/cookbook/2/
