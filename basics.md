Conceptos básicos de Yii
=====================

Componente y Objecto
--------------------

Las clases del framework Yii usualmente extienden de una de las dos clases base [[yii\base\Object]] o [[yii\base\Component]].
Estas clases proporcionan útiles características que son añadidas automáticamente a todas las clases que las extienden.

La clase [[yii\base\Object|Object]] proporciona la [configuración y propiedades características](../api/base/Object.md).
La clase [[yii\base\Component|Component]] extiende de [[yii\base\Object|Object]] y añade 
[Manejo de eventos](events.md) y [comportamientos](behaviors.md).

[[yii\base\Object|Object]] es normalmente usada por clases que representan estructuras básicas de datos mientras que
[[yii\base\Component|Component]] es usada por componentes de la aplicación y otras clases que implementan una lógica más elevada.


Configuración de Objeto
--------------------

La clase [[yii\base\Object|Object]] introduce un camino uniforme en la configuración de objetos. Cualquier clase descendiente 
de [[yii\base\Object|Object]] debe declarar su constructor (si lo necesita) de la siguiente manera para que pueda ser
configurado adecuadamente.

```php
class MyClass extends \yii\base\Object
{
    public function __construct($param1, $param2, $config = [])
    {
        // ... inicialización antes de que la configuración sea aplicada

        parent::__construct($config);
    }

    public function init()
    {
        parent::init();

		// ... inicialización después de que la configuración ha sido aplicada
    }
}
```

En el ejemplo anterior, el último parámetro del constructor debe tomar un array de configuración el cual 
contenga parejas nombre-valor, que serán usadas para inicializar las propiedades del objeto cuando finalice el constructor.
Puede sobreescribirse el método `init()` para hacer la inicialización después de que la configuración ha sido aplicada.

Siguiendo esta convención, es posible crear y configurar nuevos objetos usando un array de configuración como el 
mostrado a continuación.

```php
$object = Yii::createObject([
    'class' => 'MyClass',
    'property1' => 'abc',
    'property2' => 'cde',
], [$param1, $param2]);
```


Path aliases
-----------

Yii 2.0 extiende el uso de path alias (alias de ruta) a rutas de fichero/directorio y URLs. Un alias debe comenzar
con un símbolo `@` para que pueda ser diferenciado de rutas de fichero/directorio o de Urls.
Por ejemplo, el alias `@yii` se refiere al directorio de instalación de Yii mientras que `@web` contiene la URL base para la actual aplicación web.
Alias de ruta son soportados en muchos lugares en el núcleo del código de Yii.
Por ejemplo `FileCache::cachePath` puede aceptar un alias de ruta o una ruta normal de directorio.

Los alias de ruta están muy relacionados con los nombres de espacio de clases. Es recomendado que un
alias de ruta sea definido para cada nombre de espacio raiz de esa forma la clase de autocarga de Yii 
puede ser usada sin ninguna configuración adicional. Por ejemplo, como `@yii` remite al directorio de instalación de Yii,
una clase `yii\web\Request` pude ser cargada automáticamente por Yii. Si se utiliza una biblioteca de terceros 
de Zend Framework, se puede definir un alias de ruta `@Zend` el cual remita a su directorio de instalación
y Yii será capaz de cargar automáticamente cualquier clase en esta biblioteca.

Los siguientes alias son predefinidos por el núcleo del framework:

- `@yii` - directorio del framework.
- `@app` - ruta base de la aplicación actual.
- `@runtime` - directorio runtime.
- `@vendor` - directorio vendor de Composer.
- `@webroot` - directorio raiz de la web de la aplicación actual.
- `@web` - URL base de la aplicación actual.

Autoloading
----------

Todas las clases, interfaces y traits (rasgos) son cargadas automáticamente en el momento que son usadas. No hay necesidad de usar `include` o `require`. 
Esto es cierto para el cargador de paquetes de composer como también para las extensiones de Yii.

El autoloader de Yii funciona según [PSR-4](https://github.com/php-fig/fig-standards/blob/master/proposed/psr-4-autoloader/psr-4-autoloader.md).
Eso significa que los espacios de nombres, clases, interfaces y traits (rasgos) deben corresponder acordemente a las rutas del sistema de ficheros y a los nombres de ficheros, 
excepto para el nombre de espacio raiz que es definido por un alias.

Por ejemplo, si el alias estándar `@app` se refiere a `/var/www/example.com/` entonces `\app\models\User` será cargado desde `/var/www/example.com/models/User.php`.
Un alias personalizado puede ser añadido usando el siguiente código.

```php
Yii::setAlias('@shared', realpath('~/src/shared'));
```

Adicionales autoloaders pueden ser registrados usando el estándar PHP `spl_autoload_register`.

Helper classes
--------------

Helper classes (Clases de ayuda) normalmente solo contienen métodos estáticos y son usadas de la forma:

```php
use \yii\helpers\Html;
echo Html::encode('Test > test');
```

Estas son algunas de las clases incluidas en el framework:

- ArrayHelper
- Console
- FileHelper
- Html
- HtmlPurifier
- Image
- Inflector
- Json
- Markdown
- Security
- StringHelper
- Url
- VarDumper
