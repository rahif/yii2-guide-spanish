Manejo de Errores
===============

El manejo de errores en Yii es diferente al manejo de errores en PHP plano. Lo primero de todo, Yii convierte todos los
errores no fatales a *exceptions*:

```php
use yii\base\ErrorException;
use Yii;

try {
    10/0;
} catch (ErrorException $e) {
    Yii::warning("Tried dividing by zero.");
}

// La ejecución puede continuar
```

Como puede verse en el código anterior se pueden manejar errores utilizando un bloque `try`-`catch`.

Por otra parte, incluso un fatal error es renderizado en Yii una manera agradable. Esto significa que en modo debug, se 
pueden trazar las causas de un fatal error pudiendo identificar la causa del problema más rápidamente.


Renderización de los errores en una acción de controlador dedicado
-----------------------------------------------------------

La página de error de Yii es fantástica cuando se esta desarrollando un sitio web, y es aceptable para sitios en producción 
mientras `YII_DEBUG` tenga el valor `true` en el archivo de inicio `index.php`.
Esta página puede personalizarse para hacerla más adecuada a un determinado proyecto.

La forma más fácil para crear una página de error personalizada es utilizar una acción de controlador dedicado para 
el renderizado del error. 
En primer lugar, hay que configurar el componente `errorHandler` en la configuración de la aplicación:


```php
// ...
'components' => [
    // ...
    'errorHandler' => [
        'errorAction' => 'site/error',
    ],
]
```

Tomando la configuración anterior, si un error ocurre, Yii ejecutará la acción `error` del controlador `site`.
Esa acción debe buscar una excepcion y si la encuentra renderizar el archivo de vista adecuado, pasando la excepción. 

```php
public function actionError()
{
    $exception = \Yii::$app->errorHandler->exception;
    if ($exception !== null) {
        return $this->render('error', ['exception' => $exception]);
    }
}
```

A continuación, crear el archivo `views/site/error.php`, en el cual se puede utilizar la excepción para identificar el error.
El objeto excepción tiene las siguientes propiedades:

- `statusCode`: El código del estado HTTP (p. ej. 403, 500). Solo disponible para [[yii\web\HttpException|HTTP exceptions]].
- `code`: El código de la excepción.
- `message`: El mensaje de error.
- `file`: El nombre del archivo PHP donde se ha producido el error.
- `line`: El número de línea del código donde se ha producido el error.
- `trace`: La pila de llamadas del error.


Renderización de los errores sin una acción de controlador dedicado
-----------------------------------------------------------

En lugar de crear una acción especializada dentro del controlador Site, podría indicarse a Yii que clase debe ser utilizada
para el manejo de errores:


```php
public function actions()
{
    return [
        'error' => [
            'class' => 'yii\web\ErrorAction',
        ],
    ];
}
```

Después de asociar la clase con el error como se ha indicado previamente, definir el archivo `views/site/error.php`,
el cual será utilizado automáticamente. Tres variables serán pasadas a la vista:

- `$name`: El nombre del error
- `$message`: El mensaje del error
- `$exception`: La excepción que esta siendo manejada.

El objeto `$exception` tendrá las mismas propiedades que las resaltadas arriba.
