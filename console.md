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

### Controlador y acción de consola

Un comando de consola es definido como una clase controlador que extiende de [[yii\console\Controller]]. En la clase 
controlador, se definen una o varias acciones que se corresponden con los subcomandos del comando. Dentro de cada acción,
se escribe el código con las tareas que realiza ese particular subcomando.

Cuando se ejecuta un comando, hay que especificar la ruta a la correspondiente acción del controlador. Por ejemplo,
la ruta `migrate/create` especifica el subcomando correspondiente al método acción
[[yii\console\controllers\MigrateController::actionCreate()|MigrateController::actionCreate()]].
Si una ruta no contiene un ID de la acción, la acción por defecto será ejecutada.


### Opciones

Sobreescribiendo el método [[yii\console\Controller::options()]] pueden especificarse opciones para que esten disponibles
para un comando de consola (controller/actionID). El método debe devolver una lista de los nombres de las propiedades públicas
de la clase del controlador.
Cuando se ejecuta un comando, se puede especificar el valor de una opción usando la sintaxis `--OptionName=OptionValue`.
Se asignará `OptionValue` a la propiedad `OptionName` de la clase del controlador.

Si el valor por defecto de una opción es de tipo array, entonces si se establece esta opción al ejecutar el comando,
el valor de la opción será convertido a un array dividiendo la cadena de entrada por comas.


### Argumentos

Además de las opciones, un comando también puede recibir argumentos. Los argumentos se pasan como parámetros al método de la
acción correspondiente al sub-comando solicitado. El primer argumento corresponde al primer parámetro, el segundo corresponde
al segundo, etc... Si no se proporcionan suficientes argumentos, los parámetros correspondientes pueden tomar el valor
declarado por defecto, o si no tienen valor por defecto el comando finalizará con un error.

Se puede utilizar el tipo `array` para indicar que un argumento debe ser tratado como un array. El array será generado 
dividiendo la cadena de entrada por comas.

Los siguientes ejemplos muestran como declarar argumentos:

```php
class ExampleController extends \yii\console\Controller
{
    // El comando "yii example/create test" llamará "actionCreate('test')"
    public function actionCreate($name) { ... }

    // El comando "yii example/index city" llamará "actionIndex('city', 'name')"
    // El comando "yii example/index city id" llamará "actionIndex('city', 'id')"
    public function actionIndex($category, $order = 'name') { ... }

    // El comando "yii example/add test" llamará "actionAdd(['test'])"
    // El comando "yii example/add test1,test2" llamará "actionAdd(['test1', 'test2'])"
    public function actionAdd(array $name) { ... }
}
```


### Código de salida

El uso de códigos de salida es la mejor práctica de desarrollo de aplicaciones de consola. Si un comando 
devuelve `0` significa que todo esta correcto. Si es un número más grande que cero, hay un error y el número 
devuelto es el código de error que puede ser interpretado para averiguar detalles sobre el error.
Por ejemplo, `1` puede asignarse en general para un error desconocido y todos los códigos superiores son
declarados para casos específicos tales como errores de entrada, archivos perdidos, y así sucesivamente.

Para que un comando de consola devuelva un código de salida simplemente hay que devolver un entero en el
método de la acción del controlador:

```php
public function actionIndex()
{
    if (/* hay algún problema */) {
        echo "Un problema en comando!\n";
        return 1;
    }
    // hacer algo
    return 0;
}
```
