Comportamientos
=============

Un comportamiento (también conocido como *mixin*) puede ser usado para mejorar la funcionalidad de un componente existente sin modificar
el código del componente. En particular, un comportamiento puede "inyectar" sus métodos y propiedades públicas en el componente, haciendolas
directamente accesibles desde el propio componente. Un comportamiento puede también responder a eventos lanzados en el componente,
además de interceptar la ejecución normal del código.
A diferencia de [traits en PHP](http://www.php.net/traits), los comportamientos se pueden unir a las clases en tiempo de ejecución.

Usando comportamientos
-------------------

Un comportamiento puede ser unido a cualquier clase que extienda de [[yii\base\Component]] desde el código o por configuración
de la aplicación.

### Uniendo comportamientos mediante el método `behaviors`

Para unir un comportamiento a una clase se puede implementar el método `behaviors` del componente.
Como un ejemplo, Yii proporciona el comportamiento [[yii\behaviors\TimestampBehavior]] para automáticamente actualizar campos
con marcas de tiempo cuando grabamos un modelo [[yii\db\ActiveRecord|Active Record]].

```php
use yii\behaviors\TimestampBehavior;

class User extends ActiveRecord
{
    // ...

    public function behaviors()
    {
        return [
            'timestamp' => [
                'class' => TimestampBehavior::className(),
                'attributes' => [
                    ActiveRecord::EVENT_BEFORE_INSERT => ['created_at', 'updated_at'],
                    ActiveRecord::EVENT_BEFORE_UPDATE => 'updated_at',
                ],
            ],
        ];
    }
}
```

En el ejemplo anterior, el nombre `timestamp` puede ser usado para referenciar el comportamiento a través del componente. Por ejemplo, 
`$user->timestamp` obtiene la instancia del comportamiento. El array correspondiente es la configuración usada para crear el
objeto [[yii\behaviors\TimestampBehavior|TimestampBehavior]].

Más allá de responder a los eventos de inserción y actualización del registro activo, `TimestampBehavior` también proporciona 
un método `touch()` que puede asignar la actual marca de tiempo a un atributo específico. 
Como se ha mencionado, se puede acceder a este método directamente a través del componente de la siguiente manera:

```php
$user->touch('login_time');
```

Si no se necesita acceder al objeto comportamiento, o el comportamiento no necesita ser personalizado, se puede
usar el siguiente formato simplificado al especificar el comportamiento.

```php
use yii\behaviors\TimestampBehavior;

class User extends ActiveRecord
{
    // ...

    public function behaviors()
    {
        return [
            TimestampBehavior::className(),
            // or the following if you want to access the behavior object
            // 'timestamp' => TimestampBehavior::className(),
        ];
    }
}
```

### Uniendo comportamientos dinámicamente

Otro camino para unir un comportamiento a un componente es llamando al método `attachBehavior` como se muestra a continuación:

```php
$component = new MyComponent();
$component->attachBehavior();
```

### Uniendo comportamientos por configuración

Se puede unir un comportamiento a un componente en su array de configuración. La sintaxis es:

```php
return [
    // ...
    'components' => [
        'myComponent' => [
            // ...
            'as tree' => [
                'class' => 'Tree',
                'root' => 0,
            ],
        ],
    ],
];
```

En la configuración anterior `as tree` representa la unión de un comportamiento llamado `tree`, y el array será pasado a [[\Yii::createObject()]]
para crear el objeto comportamiento.

Creando comportamientos propios
---------------------------

Para crear un comportamiento se debe definir una clase que extienda de [[yii\base\Behavior]].

```php
namespace app\components;

use yii\base\Behavior;

class MyBehavior extends Behavior
{
}
```

Para hacerlo personalizable, como [[yii\behaviors\TimestampBehavior]], se añaden propiedades públicas:

```php
namespace app\components;

use yii\base\Behavior;

class MyBehavior extends Behavior
{
    public $attr;
}
```

Ahora, cuando se utiliza el comportamiento, se puede configurar el atributo en el cual te gustaría que fuese aplicado:

```php
namespace app\models;

use yii\db\ActiveRecord;

class User extends ActiveRecord
{
    // ...

    public function behaviors()
    {
        return [
            'mybehavior' => [
                'class' => 'app\components\MyBehavior',
                'attr' => 'member_type'
            ],
        ];
    }
}
```

Los comportamientos son normalmente escritos para ejecutar acciones cuando ocurren ciertos eventos. En el siguiente ejemplo, se implementa el
método `events` para asignar un manejador de eventos:

```php
namespace app\components;

use yii\base\Behavior;
use yii\db\ActiveRecord;

class MyBehavior extends Behavior
{
    public $attr;

    public function events()
    {
        return [
            ActiveRecord::EVENT_BEFORE_INSERT => 'beforeInsert',
            ActiveRecord::EVENT_BEFORE_UPDATE => 'beforeUpdate',
        ];
    }

    public function beforeInsert() {
        $model = $this->owner;
        // Use $model->$attr
    }

    public function beforeUpdate() {
        $model = $this->owner;
        // Use $model->$attr
    }
}
```
