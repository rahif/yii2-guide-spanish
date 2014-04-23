La herramienta de generación de código de Yii
========================================
Yii incluye una útil herramienta, llamada Gii, que proporciona prototipos rápidos mediante la generación de fragmentos de código de uso común
así como controladores de CRUD completos.


Instalación y Configuración
---------------------------

Gii es una extensión oficial de Yii. La forma principal para instalar esta extensión es a través de
[Composer](http://getcomposer.org/download/).

Ejecutar el siguiente comando:

```
php composer.phar require --prefer-dist yiisoft/yii2-gii "*"
```

O añadir este código a la sección `require` del archivo `composer.json`:

```
"yiisoft/yii2-gii": "*"
```

Una vez que la extensión Gii ha sido instalada, habilitarla añadiendo las siguientes lineas al archivo de configuración:

```php
'modules' => [
    'gii' => [
        'class' => 'yii\gii\Module',
    ],
]
```

Acceder a Gii a través de la siguiente URL:

```
http://localhost/path/to/index.php?r=gii
```

> Nota: para acceder a Gii desde una dirección IP distinta a la local, el acceso será denegado por defecto.
Para sortear el valor por defecto, añadir las direcciones Ip permitidas a la configuración:

>
```php
'gii' => [
    'class' => 'yii\gii\Module',
    'allowedIPs' => ['127.0.0.1', '::1', '192.168.0.*', '192.168.178.20'] // ajustar según se necesite
],
```

### Plantilla básica de la Aplicación

La estructura de la aplicación de la plantilla básica es un poco diferente por lo que Gii debe ser 
configurado en `config/web.php`:

```php
// ...
if (YII_ENV_DEV)
{
    // ajustes en la configuración para el entorno 'dev'
    $config['bootstrap'][] = 'debug';
    $config['modules']['debug'] = 'yii\debug\Module';
    $config['modules']['gii'] = 'yii\gii\Module'; // <--- Aquí
}
```

Así que con el fin de ajustar la dirección IP se tiene que hacer lo siguiente:

```php
if (YII_ENV_DEV)
{
    // ajustes en la configuración para el entorno 'dev'    
    $config['bootstrap'][] = 'debug';
    $config['modules']['debug'] = 'yii\debug\Module';
    $config['modules']['gii'] = [
        'class' => 'yii\gii\Module',
        'allowedIPs' => ['127.0.0.1', '::1', '192.168.0.*', '192.168.178.20'],
    ];
}
```

Cómo utilizarlo
-------------

Cuando se abre Gii por primera vez la página de entrada deja elegir un generador.

![Página de entrada de Gii](images/gii-entry.png)

Por defecto están disponibles los siguientes generadores:

- **Generador de Modelo** - Genera una clase ActiveRecord para la tabla especificada de la base de datos.
- **Generador de CRUD** - Genera un controlador y las vistas que implementan el CRUD (Crear, Leer, Actualizar, Borrar),
  las operaciones para el modelo de datos especificado. (CRUD son las siglas en inglés)  
- **Generador de controlador** - Ayuda a generar rapidamente una nueva clase controlador, una o algunas acciones del
  controlador y sus correspondientes vistas.
- **Generador de formulario** - Genera un archivo de vista que presenta en pantalla un formulario para recoger información para
  la clase del modelo especificado.
- **Generador de módulo** - Ayuda a generar el código necesario para componer la estructura para un módulo de Yii.


Después de elegir un generador pulsando en el botón "start" se mostrará un formulario que permite configurar los
parámetros del generador. Rellenar el formulario y pulsar el botón "Preview" para obtener un avance del código generado por gii.
Dependiendo del generador elegido y si los archivos existían previamente o no, se obtendrá una salida similar a la mostrada en
la siguiente imagen:

![Gii preview](images/gii-preview.png)

Pulsando en el nombre del archivo se puede ver una vista previa del código que será generado para ese fichero.
Si el fichero existía previamente, gii también proporciona una vista con las diferencias entre el código existente
y el que será generado. En este caso hay que elegir que ficheros deben ser sobreescritos y cuales no.


> Consejo: Cuando se utiliza el generador de modelo para actualizar los modelos después de un cambio en la base de datos, puede
  copiar el código de la vista previa de gii y fusionar los cambios con su propio código. Puede usar las características de un
  IDE como PHPStorms [comparar con el portapapeles](http://www.jetbrains.com/phpstorm/webhelp/comparing-files.html)
  para esto, lo cual permite fusionar cambios relevantes y mantener otros que podrían revertir el código.
  
Después de revisar el código y seleccionar los archivos para ser generados pulsar el botón "Generate" para crear los archivos.
Si todo ha ido bien ya esta terminado. Si hay errores de que gii no es capaz de generar los archivos hay que cambiar los
permisos de los directorios para que el servidor web sea capaz de escribir en ellos y también pueda crear los archivos.

> Nota: El código generado por gii es solo una plantilla que debe ser personaliza según la aplicación. Esta ahí para ayudar en
  la creación de nuevas cosas rapidamente pero no es algo que crea código listo para usar.
  A menudo la gente usa los modelos generados por gii sin cambiar y solo los extienden para ajustar algunas partes de el.
  Esta no es la forma en que está destinado a ser utilizado. El código generado por gii puede estar incompleto o incorrecto
  y tiene que ser cambiado para ajustarse a las necesidades de la aplicación antes de su uso.
 
  

Creando plantillas personalizadas
---------------------------------

Cada generador tiene un campo de formulario que deja elegir una plantilla para el uso de la generación del código.
Por defecto gii solo proporciona una plantilla pero se puede crear una plantilla para personalizar el código.


Si miras en la carpeta `@app\vendor\yiisoft\yii2-gii\generators`, veras seis carpetas de generadores.
```
+ controller
- crud
    + default
+ extension
+ form
+ model
+ module
```
Son los nombres de los generadores. Si abres cualquiera de esas carpetas, veras la carpeta llamada `default`. La carpeta es el nombre de la plantilla.

Copia la carpeta `@app\vendor\yiisoft\yii2-gii\generators\crud\default` en otro lugar, por ejemplo `@app\myTemplates\crud\`.
Ahora abre esta carpeta y modifica cualquier plantilla que se adecue a tus necesidades, por ejemplo, añade `errorSummary` en `views\_form.php`:

```php
<?php
//...
<div class="<?= Inflector::camel2id(StringHelper::basename($generator->modelClass)) ?>-form">

    <?= "<?php " ?>$form = ActiveForm::begin(); ?>
    <?= "<?=" ?> $form->errorSummary($model) ?> <!-- AÑADIDO AQUÍ -->
    <?php foreach ($safeAttributes as $attribute) {
        echo "    <?= " . $generator->generateActiveField($attribute) . " ?>\n\n";
    } ?>
//...
```

Ahora tienes que configurar GII para que sepa de tu plantilla. Esta configuración se hace en el documento de configuración de la aplicación:

```php
// config/web.php para basic app
// ...
if (YII_ENV_DEV) {    
    $config['modules']['gii'] = [
        'class' => 'yii\gii\Module',      
        'allowedIPs' => ['127.0.0.1', '::1', '192.168.0.*', '192.168.178.20'],  
        'generators' => [ //here
            'crud' => [ //name generator
                'class' => 'yii\gii\generators\crud\Generator', //generador de clases
                'templates' => [ //configurando las plantillas
                    'myCrud' => '@app\myTemplates\crud\default', //nombre de la plantilla => dirección de la plantilla
                ]
            ]
        ],
    ];
}
```
Abre el generador CRUD y veras que en el campo `Code Template` del formulario aparecerá el resultado de nuestra plantilla.


Creando tus propios generadores
-------------------------------

Abre la carpeta de cualquier generador y verás dos documentos `form.php` y `Generator.php`. 
Uno es el formulario, el otro es la clase del generador. Para crear tu propio generador, tendras que crear o sobrescribir 
esas clases en cualquiera de las carpetas. De nuevo como en el párrafo anterior customize la configuración:

```php
//config/web.php para basic app
//..
if (YII_ENV_DEV) {    
    $config['modules']['gii'] = [
        'class' => 'yii\gii\Module',      
        'allowedIPs' => ['127.0.0.1', '::1', '192.168.0.*', '192.168.178.20'],  
         'generators' => [
            'myCrud' => [
                'class' => 'app\myTemplates\crud\Generator',
                'templates' => [
                    'my' => '@app/myTemplates/crud/default',
                ]
            ]
        ],
    ];
}
```

```php
// @app/myTemplates/crud/Generator.php
<?php
namespace app\myTemplates\crud;

class Generator extends \yii\gii\Generator
{
    public function getName()
    {
        return 'MY CRUD Generator';
    }

    public function getDescription()
    {
        return 'Mi generador crud. Igual como el nativo, pero es mío...';
    }
    
    // ...
}
```

Abre el Modulo Gii y veras que un nuevo generador aparece en él.


