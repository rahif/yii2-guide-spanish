Helper Classes
==============

Yii provee muchas clases que ayudan a simplificar tareas comunes de codificación, como manipulación de cadenas o arrays,
generación de código HTML, y mucho más. Estas helper classes (clases de ayuda) están organizadas bajo el espacio de nombres `yii\helpers` 
y son todas clases estáticas (solo contienen propiedades y métodos estáticos y no deben ser instanciados).

Usa una helper class llamando directamente a uno de sus métodos estáticos.

```
use yii\helpers\ArrayHelper;

$c = ArrayHelper::merge($a, $b);
```

Extendiendo Helper classes (Clases de ayuda)
------------------------

Para hacer más fácil extender helper classes Yii rompe cada helper class en dos clases: una clase base (p. ej. `BaseArrayHelper`)
y una clase específica (p. ej. `ArrayHelper`). Al usar un helper debe usarse solo la versión específica, nunca la clase base.

Si se quiere personalizar un helper, realizar los siguientes pasos (usando `ArrayHelper` como un ejemplo):

1. Nombra a la nueva clase igual que la clase específica que provee Yii, incluyendo el espacio de nombre: `yii\helpers\ArrayHelper`
2. Extiende la nueva clase de la clase base: `class ArrayHelper extends \yii\helpers\BaseArrayHelper`.
3. En la nueva clase, sobreescribe cualquier método o propiedad si es necesario o añade nuevos métodos o propiedades.
4. Para que la aplicación utilice la nueva versión del helper class incluye la siguiente linea de código en el script de incio:

```php
Yii::$classMap['yii\helpers\ArrayHelper'] = 'path/to/ArrayHelper.php';
```

Paso 4 anterior indica a la clase autoloader de Yii para que carge la nueva versión del helper class en lugar del incluido en la distribución de Yii.
> Consejo: Puede usarse `Yii::$classMap` para reemplazar cualquier clase del core (núcleo) con una versión personalizada, no solo las helper classes. 
