yii2-giix
===========

Extended models and CRUDs for Gii, the code generator of Yii2 Framework

**PROJECT IS IN DEVELOPMENT STAGE!**


What is it?
-----------

Yii2-Giix provides templates for model and CRUD generation with relation support and a sophisticated UI.
A main project goal is porting many features and learnings from yii2-giiant, gtc, giix, awecrud and others into one solution.


Installation
------------

The preferred way to install this extension is through [composer](http://getcomposer.org/download/).

    composer.phar require veyselsahin/yii2-giix:"*"

The generators are registered automatically in the application bootstrap process, if the Gii module is enabled

Usage
-----

Visit your application's Gii (eg. `index.php?r=gii` and choose one of the generators from the main menu screen.

For basic usage instructions see the [Yii2 Guide section for Gii](http://www.yiiframework.com/doc-2.0/guide-tool-gii.html).

### Command Line Batches

You can run batches of base-model and CRUD generation with the build in batch command:

    ./yii giix-batch --tables=profile,social_account,user,token

It will process the given tables, for more details see `./yii help giix-batch`.


Features
--------

### Model generator

- generates separate model classes to customize and base models classes to regenerate
- table prefixes can be stipped off model class names (not bound to db connection setting)

### CRUD generator

- model, view and controller locations can be customized to use subfolders
- horizontal and vertical form layout
- action button class customization
- input, attribute, column and relation customization with provider queue
- callback provider to inject any kind of code for inputs, attributes and columns via dependency injection

#### Providers

- *CallbackProvider* universal provider to modify any input, attribute or column with highly flexible callback functions
- *RelationProvider* renders code for relations (eg. links, dropdowns)
- *EditorProvider* renders RTE, like `Ckeditor` as input widget
- *DateTimeProvider* renders date inputs

Use custom generators, model and crud templates
----------------------------
```

    $config['modules']['gii'] = [
            'class'      => 'yii\gii\Module',
            'allowedIPs' => ['127.0.0.1'],
            'generators' => [
                // generator name
                'rb-model' => [
                    //generator class
                    'class'     => 'veyselsahin\giix\model\Generator',
                    //setting for out templates
                    'templates' => [
                        // template name => path to template
                        'rbModel' =>
                            '@app/giiTemplates/model/default',
    
                    ]
                ]
            ],
        ];
    
```

Customization with providers
----------------------------

In many cases you want to exchange i.e. some inputs with a customized version for your project.
Examples for this use-case are editors, file-uploads or choosers, complex input widget with a modal screen, getting
data via AJAX and so on.

With Yii2-Giix Providers you can create a queue of instances which may provide custom code depending on more complex
rules. Take a look at some existing [giix providers](https://github.com/veyselsahin/yii2-giix/tree/develop/crud/providers).

Configure providers, add this to your provider list in the form:

    \veyselsahin\giix\crud\providers\EditorProvider,
    \veyselsahin\giix\crud\providers\SelectProvider,

And configure the settings of the provider, add setting via dependecy injection this to your application config, eg. in `console/config/bootstrap.php`:

    \Yii::$container->set(
        'veyselsahin\giix\crud\providers\EditorProvider',
        [
            'columnNames' => ['description']
        ]
    );

This will render a Ckeditor widget for every column named `description`.

    <?= $form->field($model, 'description')->widget(
    \dosamigos\ckeditor\CKEditor::className(),
    [
        'options' => ['rows' => 6],
        'preset' => 'basic'
    ]) ?>


### Universal `CallbackProvider`

Configuration via DI container:

```
\Yii::$container->set(
    'veyselsahin\giix\crud\providers\CallbackProvider',
    [

        'activeFields'  => [

           /**
            * Generate a checkbox for specific column (model attribute)
            */
           'common\models\Foo.isAvailable' => function ($attribute, $generator) {
               $data = \yii\helpers\VarDumper::export([0 => 'Nein', 1 => 'Ja']);
               return <<<INPUT
\$form->field(\$model, '{$attribute}')->checkbox({$data});
INPUT;
           },
        ],


        'columnFormats' => [

           /**
            * generate custom HTML in column
            */
           'common\models\Foo.html' => function ($attribute, $generator) {

               return <<<FORMAT
[
    'format' => 'html',
    'label'=>'FOOFOO',
    'attribute' => 'item_id',
    'value'=> function(\$model){
        return \yii\helpers\Html::a(\$model->bar,['/crud/item/view', 'id' => \$model->link_id]);
    }
]
FORMAT;
           },


           /**
            * hide all text fields in grid
            */
           '.+' => function ($column, $model) {
                    if ($column->dbType == 'text') {
                        return false;
                    }
           },

           /**
            * hide system fields in grid
            */
           'created_at$|updated_at$' => function () {
                   return false;
           },

        ]
    ]
);
```

Extras
------

A detailed description how to use MySQL workbench for model updates and migration see [here](docs/using-mysql-workbench.md).


Links
-----

- [GitHub](https://github.com/veyselsahin/yii2-giix)
- [Packagist](https://packagist.org/packages/veyselsahin/yii2-giix)


php yii.php  giix-batch   --modelDb=cafees --modelBaseClass=common\\models\\VysModel   --modelNamespace=backend\\models   --tables=gruplar,sinav,sinav_sonuc

php yii.php  giix-batch   --modelDb=cafees --modelBaseClass=common\\models\\VysModel   --modelNamespace=backend\\models 
