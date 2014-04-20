Console applications
====================

Yii ofrece un soporte completo de consola. Una aplicación de consola en Yii está estructurada en una forma muy similar 
a una aplicación web. Consiste de uno o más [[yii\console\Controller]] (a menudo referidos como comandos), cada uno
de los cuales tiene una o más acciones.

Uso
---

Se puede ejecutar una acción de un controlador usando la siguiente sintaxis:

```
yii <route> [--option1=value1 --option2=value2 ... argument1 argument2 ...]
```

Por ejemplo, [[yii\console\controllers\MigrateController::actionCreate()|MigrateController::actionCreate()]]
con [[yii\console\controllers\MigrateController::$migrationTable|MigrateController::$migrationTable]] 
puede ser llamado desde la línea de comandos de la siguiente manera:

```
yii migrate/create --migrationTable=my_migration
```

En el ejemplo anterior `yii` es la rutina de inicio de la aplicación de consola descrita a continuación.

Rutina de entrada
---------------

La rutina de entrada de una aplicación de consola normalmente es `yii`, localizada en la raiz del directorio de la
aplicación y contiene un código similar al siguiente:

```php
#!/usr/bin/env php
<?php
/**
 * Yii console bootstrap file.
 *
 * @link http://www.yiiframework.com/
 * @copyright Copyright (c) 2008 Yii Software LLC
 * @license http://www.yiiframework.com/license/
 */

defined('YII_DEBUG') or define('YII_DEBUG', true);

// fcgi doesn't have STDIN and STDOUT defined by default
defined('STDIN') or define('STDIN', fopen('php://stdin', 'r'));
defined('STDOUT') or define('STDOUT', fopen('php://stdout', 'w'));

require(__DIR__ . '/vendor/autoload.php');
require(__DIR__ . '/vendor/yiisoft/yii2/Yii.php');

$config = require(__DIR__ . '/config/console.php');

$application = new yii\console\Application($config);
$exitCode = $application->run();
exit($exitCode);

```

Esta rutina es una parte de la aplicación que puede ajustarse con total libertad. La constante `YII_DEBUG` puede ser configurada
a `false` si no se quiere ver el seguimiento de errores de la pila y se quiere mejorar el rendimiento total. En ambos casos con 
la plantilla básica y la avanzada es habilitada para proporcionar un entorno más amigable al desarrollador.


Configuración
-------------

Como puede verse en el código anterior, una aplicación de consola utiliza su propio archivo de configuración llamado `console.php`.
En este archivo, debe especificarse como configurar los componentes y propiedades de la aplicación.

Si la aplicación web y la aplicación de consola comparten la mayor parte de la configuración, puede considerarse mover la
parte común a un archivo separado, e incluir este archivo en ambos archivos de configuración de la aplicación, tal y como
es hecho en la plantilla "avanzada" de la aplicación.

Algunas veces, se puede querer ejecutar un comando de consola utilizando una configuración de la aplicación que es diferente
de la especificada en la rutina de entrada. Por ejemplo, puede quererse utilizar el comando `yii migrate` para mejorar unos
test de base de datos los cuales están configurados en cada conjunto de pruebas individuales. Para hacerlo, simplemente hay que
especificar el archivo de configuración personalizado de la aplicación con la opción `appconfig`, como se indica a continuación:

```
yii <route> --appconfig=path/to/config.php ...
```


Creando comandos de consola propios
-------------------------------

### Console Controller and Action

Un comando de consola es definido como una clase controlador que extiende de [[yii\console\Controller]]. En la clase 
controlador, se definen una o varias acciones que se corresponden con los subcomandos del comando. Dentro de cada acción,
se escribe el código con las tareas que realiza ese particular subcomando.

Cuando se ejecuta un comando, hay que especificar la ruta a la correspondiente acción del controlador. Por ejemplo,
la ruta `migrate/create` especifica el subcomando correspondiente al método acción
[[yii\console\controllers\MigrateController::actionCreate()|MigrateController::actionCreate()]].
Si una ruta no contiene un ID de la acción, la acción por defecto será ejecutada.


### Opciones

By overriding the [[yii\console\Controller::options()]] method, you can specify options that are available
to a console command (controller/actionID). The method should return a list of public property names of the controller class.
When running a command, you may specify the value of an option using the syntax `--OptionName=OptionValue`.
This will assign `OptionValue` to the `OptionName` property of the controller class.

If the default value of an option is of array type, then if you set this option while running the command,
the option value will be converted into an array by splitting the input string by commas.

### Arguments

Besides options, a command can also receive arguments. The arguments will be passed as the parameters to the action
method corresponding to the requested sub-command. The first argument corresponds to the first parameter, the second
corresponds to the second, and so on. If there are not enough arguments are provided, the corresponding parameters
may take the declared default values, or if they do not have default value the command will exit with an error.

You may use `array` type hint to indicate that an argument should be treated as an array. The array will be generated
by splitting the input string by commas.

The follow examples show how to declare arguments:

```php
class ExampleController extends \yii\console\Controller
{
    // The command "yii example/create test" will call "actionCreate('test')"
    public function actionCreate($name) { ... }

    // The command "yii example/index city" will call "actionIndex('city', 'name')"
    // The command "yii example/index city id" will call "actionIndex('city', 'id')"
    public function actionIndex($category, $order = 'name') { ... }

    // The command "yii example/add test" will call "actionAdd(['test'])"
    // The command "yii example/add test1,test2" will call "actionAdd(['test1', 'test2'])"
    public function actionAdd(array $name) { ... }
}
```


### Exit Code

Using exit codes is the best practice of console application development. If a command returns `0` it means
everything is OK. If it is a number greater than zero, we have an error and the number returned is the error
code that may be interpreted to find out details about the error.
For example `1` could stand generally for an unknown error and all codes above are declared for specific cases
such as input errors, missing files, and so forth.

To have your console command return with an exit code you simply return an integer in the controller action
method:

```php
public function actionIndex()
{
    if (/* some problem */) {
        echo "A problem occured!\n";
        return 1;
    }
    // do something
    return 0;
}
```
