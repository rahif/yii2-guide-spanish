Vista
=====

El componente vista es una parte importante de MVC. La vista actúa como un interfaz a la aplicación, haciendolá responsable
de la presentación de datos, muestra de formularios, y mucho más al usuario final.


Básico
------

Por defecto, Yii utiliza PHP en las plantillas de vistas para generar el contenido y los elementos. 
Una vista de una aplicación web normalmente contiene una combinación de HTML, junto con estructuras básicas de PHP como
`echo`, `foreach`, `if`, ...
El uso de código PHP complejo en las vistas es considerado como una mala práctica. 
Cuando se necesita lógica y funcionalidad compleja, dicho código debe ser trasladado a un controlador o a un widget.

La vista es normalmente llamada desde una acción de un controlador utilizando el método [[yii\base\Controller::render()|render()]]:

```php
public function actionIndex()
{
    return $this->render('index', ['username' => 'samdark']);
}
```

El primer argumento de [[yii\base\Controller::render()|render()]] es el nombre de la vista que se va a mostrar.
En el contexto del controlador, Yii buscará la vista en `views/site/` donde `site` es el ID del controlador.
Para más detalles sobre como es resuelto el nombre de la vista, ver el método [[yii\base\Controller::render()]].

El segundo argumento de [[yii\base\Controller::render()|render()]] es un array de datos de parejas clave-valor.
A través de este array, los datos pueden ser pasados a la vista, haciendo que estos datos esten disponibles en la vista 
como una variable cuyo nombre es la clave asignada en el array.

La vista para la acción del ejemplo anterior estaría en `views/site/index.php` y puede ser algo así como:

```php
<p>Hello, <?= $username ?>!</p>
```

Cualquier tipo de datos puede ser pasado a la vista, incluyendo arrays de objetos.

Más allá del método [[yii\web\Controller::render()|render()]], la clase [[yii\web\Controller]] también proporciona
otros métodos de renderización. Debajo se muestra un sumario de estos métodos:

* [[yii\web\Controller::render()|render()]]: renderiza una vista y aplica el layout al resultado renderizado.
  Este es el método comunmente más utilizado para renderizar una página completa.
* [[yii\web\Controller::renderPartial()|renderPartial()]]: renderizar una vista sin aplicar ningún layout
  Esto es utilizado a menudo para renderizar un fragmento de una página.
* [[yii\web\Controller::renderAjax()|renderAjax()]]: renderizar una vista sin aplicar ningún layout, incluyendo
  todos los archivos y scripts registrados de JS/CSS. Es utilizado comunmente para renderizar un flujo de salida HTML
  como respuesta a una petición AJAX.
* [[yii\web\Controller::renderFile()|renderFile()]]: renderizar un archivo de vista. Este método es similar a
  [[yii\web\Controller::renderPartial()|renderPartial()]] excepto que toma la ruta del archivo de la vista 
  en lugar del nombre de la vista.


Widgets
-------

bloques de construcción independientes

Los widgets son bloques de código independientes para las vistas, un camino para combinar lógica compleja, funcionalidad y presentación, todo ello dentro 
de un solo componente. Un widget:

* Puede contener programación avanzada de PHP.
* Normalmente es configurable.
* A menudo se le proporcionan datos para ser mostrados.
* Devuelve HTML para ser mostrado dentro del contexto de la vista.

Hay un buen número de widgets equipados con Yii, como son [active form](form.md), breadcrumbs, menu, y [wrappers around bootstrap component framework](bootstrap-widgets.md).
Además, hay extensiones que proporcionan más widgets, como el widget oficial de componentes [jQueryUI](http://www.jqueryui.com).


Para utilizar un widget, el archivo de vista haría lo siguiente:

```php
// Observa que hay que hacer "echo" del resultado para mostrarlo
echo \yii\widgets\Menu::widget(['items' => $items]);

// Pasar un array para inicializar las propiedades del objeto
$form = \yii\widgets\ActiveForm::begin([
    'options' => ['class' => 'form-horizontal'],
    'fieldConfig' => ['inputOptions' => ['class' => 'input-xlarge']],
]);
... campos del formulario aquí ...
\yii\widgets\ActiveForm::end();
```

En el primer ejemplo del código anterior, el método [[yii\base\Widget::widget()|widget()]] es utilizado para invocar
un widget que solo emite un contenido.
En el segundo ejemplo, [[yii\base\Widget::begin()|begin()]] y [[yii\base\Widget::end()|end()]] es utilizado por un
widget que envuelve el contenido entre llamadas a los métodos con su propia salida.
En el caso del formulario esta salida es la etiqueta `<form>` con algunas propiedades configuradas.


Seguridad
--------

Uno de los principales principios de seguridad es escapar siempre la salida. Si se viola ello conduce a la ejecución de un script y,
muy probablemente, a un "cross-site scripting" conocido como XSS el cual puede llevar a  una filtración de contraseñas de administrador,
haciendo que un usuario realice acciones automáticamente.

Yii proporciona una buena herramienta configurada para ayudar a escapar la salida. Lo más básico para escapar un texto sin marcado.
Puede tratar, en ese caso, de la siguiente manera:

```php
<?php
use yii\helpers\Html;
?>

<div class="username">
    <?= Html::encode($user->name) ?>
</div>
```

Cuando se quiere renderizar HTML la tarea se vuelve compleja por lo que se puede delegar esta tarea a la excelente 
librería [HTMLPurifier](http://htmlpurifier.org/) la cual es encapsulada en Yii como un helper [[yii\helpers\HtmlPurifier]]:


```php
<?php
use yii\helpers\HtmlPurifier;
?>

<div class="post">
    <?= HtmlPurifier::process($post->text) ?>
</div>
```

HTMLPurifier realiza un trabajo excelente haciendo la salida segura, pero no es muy rápido por lo que se puede considerar
[cachear el resultado](caching.md).

Lenguajes de plantilla alternativos
-------------------------------

Hay extensiones oficiales para [Smarty](http://www.smarty.net/) y [Twig](http://twig.sensiolabs.org/). Con el fin de
profundizar más examinar la sección de la guía [Utilizando motores de plantillas](template.md)

Utilizando el objeto vista en las plantillas
---------------------------------------

Una instancia del componente [[yii\web\View]] esta disponible en las plantillas de la vista como la variable `$this`.
Utilizandolo en las plantillas se pueden realizar muchas cosas útiles incluyendo configuración del título y metas de la página,
registro de scripts y acceder al contexto.


### Conmfigurar el título de la página

Un lugar común para configurar el título de la página son las plantillas de la vista. Desde que se puede acceder al objeto vista
con `$this`, configurar un título se vuelve tan fácil como:

```php
$this->title = 'My page title';
```

### Añadiendo etiquetas meta

Añadir etiquetas meta como `encoding`, `description`, `keywords` también es fácil con el objeto vista:

```php
$this->registerMetaTag(['encoding' => 'utf-8']);
```

El primer argumento es un array de parejas clave valor de las opciones de la etiqueta `<meta>`. El código anterior produciría la salida:

```html
<meta encoding="utf-8">
```

Algunas veces hay necesidad de tener solo una etiqueta de un tipo. En este caso es neceario especificar el segundo argumento:

```html
$this->registerMetaTag(['name' => 'description', 'content' => 'This is my cool website made with Yii!'], 'meta-description');
$this->registerMetaTag(['name' => 'description', 'content' => 'This website is about funny raccoons.'], 'meta-description');
```

Si hay multiples llamadas con el mismo valor del segundo argumento (`meta-description` en este caso), esta última reemplazará
la antigua y sólo se representará una etiqueta:

```html
<meta name="description" content="This website is about funny raccoons.">
```

### Registering link tags

La etiqueta `<link>` es útil en muchos casos como en la personalización del favicon, apuntando a un RSS o delegando OpenID
a otro servidor. El objeto vista en Yii tiene un método para trabajar con estos casos:

```php
$this->registerLinkTag([
    'title' => 'Lives News for Yii Framework',
    'rel' => 'alternate',
    'type' => 'application/rss+xml',
    'href' => 'http://www.yiiframework.com/rss.xml/',
]);
```

El código anterior mostrará el siguiente resultado:

```html
<link title="Lives News for Yii Framework" rel="alternate" type="application/rss+xml" href="http://www.yiiframework.com/rss.xml/" />
```

De la misma manera que con las etiquetas meta se pueden especificar un argumento adicional para asegurar que solo hay un enlace de un
tipo registrado.

### Registrando CSS

Se puede registrar CSS utilizando [[yii\web\View::registerCss()|registerCss()]] o [[yii\web\View::registerCssFile()|registerCssFile()]].
El primero registra un bloque de código CSS mientras que el segundo registra un archivo CSS externo. Por ejemplo,


```php
$this->registerCss("body { background: #f00; }");
```

El código anterior mostrará el siguiente resultado en la sección head de la página:

```html
<style>
body { background: #f00; }
</style>
```

Si se quieren especificar propiedades adicionales de la etiqueta style, pasar un array de parejas clave-valor al tercer argumento.
Si se necesita asegurar que solo hay una etiqueta style utilizar el cuarto argumento como se mencionó en la etiqueta meta.

```php
$this->registerCssFile("http://example.com/css/themes/black-and-white.css", [BootstrapAsset::className()], ['media' => 'print'], 'css-print-theme');
```

El código anterior añadirá un enlace a un archivo CSS en la sección head de la página.


* El primer argumento especifica el archivo CSS a ser registrado.
* El segundo argumento especifica que el archivo CSS depende de [[yii\bootstrap\BootstrapAsset|BootstrapAsset]], lo que significa que se agregará
  después de los archivos CSS en [[yii\bootstrap\BootstrapAsset|BootstrapAsset]]. Sin esta especificación de dependencia, el orden relativo 
  entre este archivo y los archivos CSS [[yii\bootstrap\BootstrapAsset|BootstrapAsset]] no estarían definidas.
* El tercer argumento especifica los atributos para la etiqueta resultante `<link>`.
* El último argumento especifica un ID de identificación para este archivo CSS. Si no es proporcionado, la URL del archivo CSS será utilizada en su lugar.


Es fuertemente recomendado el uso [asset bundles](assets.md) para registrar archivos CSS externos antes que utilizar 
[[yii\web\View::registerCssFile()|registerCssFile()]]. El uso de asset bundles (son archivos CSS/JS/images...) permite combinar y comprimir
múltiples archivos CSS, lo cual es conveniente para sitios web de alto tráfico.


### Registrando scripts

Con el objeto [[yii\web\View]] se pueden registrar scripts. Hay dos métodos dedicados para ello:
[[yii\web\View::registerJs()|registerJs()]] para scripts en linea y
[[yii\web\View::registerJsFile()|registerJsFile()]] para scripts externos.
Los scripts en linea son útiles para configuración y código generado dinámicamente.
El método para añadir un script en linea puede ser utilizado como se muestra a continuación:


```php
$this->registerJs("var options = ".json_encode($options).";", View::POS_END, 'my-options');
```

El primer argumento es el código actual de JS que se quiere incluir en la página. El segundo argumento
determina la posición en la que se incluirá el script dentro de la página. Posibles valores son:

- [[yii\web\View::POS_HEAD|View::POS_HEAD]] para la sección head.
- [[yii\web\View::POS_BEGIN|View::POS_BEGIN]] inmediatamente después de la etiqueta `<body>`.
- [[yii\web\View::POS_END|View::POS_END]] inmediatamente antes de cerrar la etiqueta `</body>`.
- [[yii\web\View::POS_READY|View::POS_READY]] para la ejecución de código en el evento `ready` del documento. Esto registrará [[yii\web\JqueryAsset|jQuery]] automáticamente.
- [[yii\web\View::POS_LOAD|View::POS_LOAD]] para la ejecución de código en el evento `load` del documento. Esto registrará [[yii\web\JqueryAsset|jQuery]] automáticamente.

El último argumento es un ID único del script que es utilizado para identificar el bloque de código y reemplazar uno existente 
con el mismo ID en lugar de añadir uno nuevo. Si no se proporciona, el mismo código JS será utilizado como el ID.

Un script externo puede ser añadido como se muestra a continuación:

```php
$this->registerJsFile('http://example.com/js/main.js', [JqueryAsset::className()]);
```

The arguments for [[yii\web\View::registerJsFile()|registerJsFile()]] are similar to those for
[[yii\web\View::registerCssFile()|registerCssFile()]]. In the above example,
we register the `main.js` file with the dependency on `JqueryAsset`. This means the `main.js` file
will be added AFTER `jquery.js`. Without this dependency specification, the relative order between
`main.js` and `jquery.js` would be undefined.

Like for [[yii\web\View::registerCssFile()|registerCssFile()]], it is also highly recommended that you use
[asset bundles](assets.md) to register external JS files rather than using [[yii\web\View::registerJsFile()|registerJsFile()]].


### Registrando asset bundles

Como fue mencionado anteriormente es preferido utilizar asset bundles en lugar de utilizar CSS y JavaScript directamente.
Para obtener detalles de como definir asset bundles ver la sección [administración de asset](assets.md) de la guía.
En cuanto al uso ya definido de asset bundles, es muy sencillo:

```php
\frontend\assets\AppAsset::register($this);
```

### Layout

Un layout es una forma muy conveniente de representar la parte de la página que es común para todo o al menos para la
mayoría de las páginas generadas por la aplicación. Normalmente incluye la sección `<head>`, el pie de página, 
un menú principal y elementos similares.
Se puede encontrar un buen ejemplo de layout en la [plantilla básica de una aplicación](apps-basic.md).
Aquí se muestra un layout muy básico sin widgets ni elementos extra.

```php
<?php
use yii\helpers\Html;
?>
<?php $this->beginPage() ?>
<!DOCTYPE html>
<html lang="<?= Yii::$app->language ?>">
<head>
    <meta charset="<?= Yii::$app->charset ?>"/>
    <title><?= Html::encode($this->title) ?></title>
    <?php $this->head() ?>
</head>
<body>
<?php $this->beginBody() ?>
    <div class="container">
        <?= $content ?>
    </div>
    <footer class="footer">© 2013 me :)</footer>
<?php $this->endBody() ?>
</body>
</html>
<?php $this->endPage() ?>
```

En el ejemplo anterior hay una parte de código. Lo primero de todo, `$content` es una variable que contiene el resultado
de las vistas renderizadas con el método `$this->render()` del controlador.

Se importa un helper [[yii\helpers\Html|Html]] (una clase de ayuda) mediante la sentencia estándar de PHP `use`. Este helper
es normalmente utilizado en la mayoría de las vistas donde es necesario escapar los datos de salida.

Algunos métodos especiales como  [[yii\web\View::beginPage()|beginPage()]]/[[yii\web\View::endPage()|endPage()]],
[[yii\web\View::head()|head()]], [[yii\web\View::beginBody()|beginBody()]]/[[yii\web\View::endBody()|endBody()]]
desencadenan eventos de renderizado de la página que son utilizados para registro de scripts, enlaces y procesar la página de muchas 
otras maneras.
Siempre incluir estos métodos en el layout para que el trabajo de renderizado sea procesado correctamente.

### Vistas parciales

A menudo se necesita reutilizar algún código HTML en muchas vistas y a menudo es también simple crear un widget
con todas las funciones para ello. En este caso pueden utilizarse vistas parciales.

Una vista parcial es una vista también. Ella reside en uno de los directorios bajo `views` y por convención su nombre comienza por `_`.
Por ejemplo, se necesita renderizar una lista de perfiles de usuario y, al mismo tiempo, mostrar el perfil individual en otros sitios.

Primero se necesita definir una vista parcial para el perfil del usuario en `_profile.php`:

```php
<?php
use yii\helpers\Html;
?>

<div class="profile">
    <h2><?= Html::encode($username) ?></h2>
    <p><?= Html::encode($tagline) ?></p>
</div>
```

Para usarlo en la vista `index.php` donde se muestra una lista de usuarios:

```php
<div class="user-index">
    <?php
    foreach ($users as $user) {
        echo $this->render('_profile', [
            'username' => $user->name,
            'tagline' => $user->tagline,
        ]);
    }
    ?>
</div>
```

De la misma forma puede reutilizarse en otra vista mostrando el perfil de un usario:

```php
echo $this->render('_profile', [
    'username' => $user->name,
    'tagline' => $user->tagline,
]);
```

Al llamar al método `render()` para renderizar una vista parcial dentro de una vista, pueden utilizarse diferentes formatos
para referirse a la vista parcial.
El formato utilizado normalmente es el también llamado nombre de vista relativa el cual es el que se muestra en el ejemplo anterior.
El archivo de la vista parcial es relativo al directorio que contiene la vista actual. Si la vista parcial esta localizada bajo un
subdirectorio, debe incluirse el nombre del subdirectorio en el nombre de la vista parcial, p.ej., `public/_profile`. 

También pueden utilizarse alias de ruta para especificar una vista. por ejemplo, `@app/views/common/_profile`.
Y también pueden utilizarse nombres de vistas absolutos, P.ej., `/user/_profile`, `//user/_profile`.
Un nombre de vista absoluto comienza con una barra simple o dos barras simples. Si comienza con una barra simple,
el archivo de la vista será buscado bajo la ruta de las vistas del módulo activo actual. De otra manera, será buscado
bajo la ruta de las vistas de la aplicación.


### Accediendo al contexto

Las vistas son generalmente utilizadas por un controlador o por un widget. En ambos casos el objeto que llamó a la
renderización de la vista esta disponible en la vista como `$this->context`. Por ejemplo si se necesita imprimir la
ruta de petición interna actual en una vista renderizada por un controlador puede utilizarse lo siguiente:

```php
echo $this->context->getRoute();
```

### Cachear bloques

Para aprender sobre cachear fragmentos de la vista, por fevor mirar en la sección de la guía [cach](caching.md).

Personalñizando el componente Vista
-------------------------------

Desde que la vista es tambien un componente de la aplicación llamado `view` puede ser reemplazado con un componente que
extienda de [[yii\base\View]] o [[yii\web\View]]. 
Se puede hacer a través del archivo de configuración de la aplicación `config/web.php`:

```php
return [
    // ...
    'components' => [
        'view' => [
            'class' => 'app\components\View',
        ],
        // ...
    ],
];
```
