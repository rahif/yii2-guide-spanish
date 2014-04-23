Configuración
=============

Una aplicación Yii depende de componentes para realizar la mayoría de tareas comunes, como la conexión a base de datos, enrutamiento
de solicitudes del navegador y manejo de sesiones. El comportamiento de estos componentes puede ser ajustado *configurando* la aplicación.
La mayoría de componentes tiene la configuración predeterminada por defecto, por lo que no suelen requerir muchos cambios.  
Sin embargo, hay algunas opciones de configuración que deben establecerse, tales como la conexión a base de datos. 

Como se configura una aplicación depende de la plantilla de la aplicación en uso, pero hay algunos principios generales que se aplican en todos los casos Yii.

Configurando opciones en el archivo de rutina de inicio
-------------------------------------------------

Para cada aplicación en Yii hay al menos un archivo de inicio: un guión en PHP a través del cual todas las peticiones son manejadas.
Para aplicaciones web, el archivo de inicio es normalmente `index.php`; para aplicaciones de consola, el archivo de inicio es `yii`.
Ambos archivos de rutina de inicio realizan casi el mismo trabajo:

1. Configuración de las constantes comunes.
2. Incluir el propio framework Yii.
3. Incluir [Composer autoloader](http://getcomposer.org/doc/01-basic-usage.md#autoloading).
4. Leer el archivo de configuración en `$config`.
5. Crear una instancia de la aplicación, configurada por `$config`, y ejecutar esa instancia. 

Como cualquier recurso en una aplicación Yii, el archivo de inicio puede ser editado para ajustarse a lo que se necesite Un cambio típico es el valor de `YII_DEBUG`.
Esta constante debe ser `true` durante el desarrollo, pero siempre debe ser `false` en un sitio en producción.

La estructura por defecto del archivo de inicio configura `YII_DEBUG` a `false` si no esta definido:

```php
defined('YII_DEBUG') or define('YII_DEBUG', false);
```

Durante el desarrollo, se puede cambiar a `true`:

```php
define('YII_DEBUG', true); // Solo en desarrollo
defined('YII_DEBUG') or define('YII_DEBUG', false);
```

Configurando la instancia de la aplicación
-------------------------------------

Una instancia de la aplicación es configurada cuando es creada en el archivo de inicio. La configuración es normalmente
almacenada en un archivo PHP ubicado en `/config` dentro del directorio de la aplicación. El archivo tiene esta estructura al comenzar:

```php
<?php
return [
    'id' => 'applicationId',
    'basePath' => dirname(__DIR__),
    'components' => [
        // configuration of application components goes here...
    ],
    'params' => require(__DIR__ . '/params.php'),
];
```

La configuración es un gran array de parejas clave-valor. En el ejemplo anterior, las claves del array son los nombres de las propiedades de la aplicación.
Dependiendo del tipo de aplicación, se pueden configurar las propiedades de las clases [[yii\web\Application]] o [[yii\console\Application]].
Ambas clases extienden [[yii\base\Application]].

Notar que no solo se pueden configurar propiedades públicas de la clase, también cualquier propiedad accesible por un setter. Por ejemplo,
para configurar la ruta del directorio runtime, se puede usar una clave llamada `runtimePath`. No hay ninguna propiedad en la clase de la aplicación,
pero como la clase tiene el correspondiente setter llamado `setRuntimePath`, `runtimePath` viene a ser configurable.
La característica para configurar propieades por setters esta disponible en cualquier clase que extienda de [[yii\base\Object]], de la cual extienden
casi todas las clases en el framework Yii.


Configurando componentes de la aplicación
------------------------------------

La mayoría de las funcionalidades de Yii vienen de los componentes de la aplicación. Estos componentes son unidos a la instancia de la aplicación por la propiedad 
`components` de la instancia:

```php
<?php
return [
    'id' => 'applicationId',
    'basePath' => dirname(__DIR__),
    'components' => [
        'cache' => ['class' => 'yii\caching\FileCache'],
        'user' => ['identityClass' => 'app\models\User'],
        'errorHandler' => ['errorAction' => 'site/error'],
        'log' => [
            'traceLevel' => YII_DEBUG ? 3 : 0,
            'targets' => [
                [
                    'class' => 'yii\log\FileTarget',
                    'levels' => ['error', 'warning'],
                ],
            ],
        ],
    ],
    // ...
];
```

En el código anterior, cuatro componentes son configurados: `cache`, `user`, `errorHandler`, `log`. Cada clave de entrada es un ID del componente.
Los valores son subarrays utilizados para configurar ese componente. El ID del componente es también utilizado para acceder al componente en cualquier
lugar de la aplicación como `\Yii::$app->myComponent`.

El array de configuración tiene una clave especial llamada `class` que identifica la clase base del componente. El resto de las claves y valores
son utilizados para configurar las propiedades del componente de la misma forma que las claves del nivel superior configuran las propiedades
de la aplicación.

Cada aplicación tiene un predefinido conjunto de componentes. Para configurar uno de estos, la clave `class` puede ser omitida y se utilizará la clase
por defecto de Yii para ese componente. Se puede comprobar el método `coreComponents()` de la aplicación para conseguir un listado de los IDs de los
componentes y sus correspondientes clases.

Notar que Yii es bastante inteligente para configurar el componente solo cuando va a ser utilizado: por ejemplo, si se configura el componente `cache`
en el archivo de configuración pero nunca se utiliza el componente `cache` en la aplicación, ninguna instancia de ese componente será creada y no
se desperdiciará tiempo configurandola.

Configuración por defecto de los componentes de toda la clase
------------------------------------------------------

Para cada componente se puede especificar la configuración por defecto en toda la clase. Por ejemplo, si se quiere cambiar la clase utilizada para todos
los widgets `LinkPager` sin especificar la clase para cada widget utilizado, se puede hacer lo siguiente:

```php
\Yii::$container->set('yii\widgets\LinkPager', [
    'options' => [
        'class' => 'pagination',
    ],
]);
```

El código anterior debe ser ejecutado una vez antes que el widget `LinkPager` sea utilizado. Puede ser hecho en `index.php`, el archivo de configuración de la
aplicación, o en cualquier otro lugar.
