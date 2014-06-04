Plantilla básica de la aplicación
=============================

La aplicación Yii que utiliza la plantilla básica se ajusta perfectamente para pequeños proyectos o tan solo para aprender el framework.

La plantilla básica de la aplicación incluye cuatro páginas: homepage, about , contact and login.

La página contact muestra un formulario de contacto que los usuarios pueden rellenar para enviar sus consultas al administrador del sitio web.
Asumiendo que el sitio tiene acceso a un servidor de correo y que la dirección de email del administrador ha sido introducida en el archivo de
configuración , el formulario de contacto funcionará. Lo mismo vale para la página de acceso (login), la cual permite a los usuarios ser autenticados
antes de acceder al contenido privado.

Instalación
------------

La instalación del framework requiere [Composer](http://getcomposer.org/). Si no se tiene composer aún en el sistema es posible descargarlo
desde [http://getcomposer.org/](http://getcomposer.org/), o ejecutar el siguiente comando en Linux/Unix/MacOS:

~~~
curl -s http://getcomposer.org/installer | php
~~~

Para crear una aplicación que utilice la plantilla básica ejecuta el siguiente comando:

~~~
php composer.phar create-project --prefer-dist --stability=dev yiisoft/yii2-app-basic /path/to/yii-application
~~~

Ahora configura el directorio raiz de los documentos del servidor web a /ruta/a/aplicacion-yii/web y debe ser posible acceder a la aplicación
mediante la URL `http://localhost/`.

Estructura del directorio
----------------------

La plantilla básica de una aplicación no divide mucho los directorios. Aquí se muestra la estructura básica:

- `assets` - archivos públicos de la aplicación.
  - `AppAsset.php` - definición de los activos de una aplicación (archivos públicos) como son CSS, JavaScript etc. Comprobar [Administración de assets](assets.md) para
    más detalles.
- `commands` - controladores de consola.
- `config` - archivos de configuración.
- `controllers` - controladores web.
- `models` - modelos de la aplicación.
- `runtime` - archivos de registro, archivos de estados temporales, archivos de caché.
- `views` - plantillas de las vistas.
- `web` - webroot.

El directorio raiz contiene un conjunto de ficheros.

- `.gitignore` contiene una lista de directorios ignorados por el sistema de versiones git. Si un código fuente no debe estar en
  el repositorio, añadelo en este archivo.
- `codeception.yml` - configuración de Codeception.
- `composer.json` - configuración de Composer. (descrita en detalle más abajo).
- `LICENSE.md` - información de la licencia. Poner la licencia del proyecto aquí. Especialmente si el código es libre (opensourcing).
- `README.md` - información básica sobre la instalación de la aplicación. Considerar reemplazar con información del nuevo proyecto
  y cambios en el proceso de instalación.
- `requirements.php` - archivo que comprueba los requerimientos de Yii.
- `yii` - archivo de arranque para iniciar la aplicación desde consola.
- `yii.bat` - lo mismo para Windows.


### config

Este directorio contiene los archivos de configuración:

- `console.php` - configuración de la aplicación utilizada al ejecutar comandos de consola.
- `params.php` - parámetros generales de la aplicación.
- `web.php` - configuración de la aplicación utilizada por el servidor web.
- `web-test.php` - configuración de la aplicación utilizada al ejecutar pruebas funcionales.

Todos estos archivos devuelven un array que es utilizado para configurar las correspondientes propiedades de la aplicación-
Comprobar la sección de la guía [Configuración](configuration.md) para más detalles.

### views

El directorio views contiene las plantillas de las vistas que utiliza la aplicación. En la plantilla básica de la aplicación:

```
layouts
    main.php
site
    about.php
    contact.php
    error.php
    index.php
    login.php
```

`layouts` contiene los diseños HTML p.ej. los marcadores de la página excepto el contenido: doctype, head section, main menu, footer etc.
El resto normalmente son vistas del controlador. Por convención están situadas en subdirectorios que coinciden con el id
del controlador. Las vistas de `SiteController` están en el directorio `site`. Los nombres de las vistas normalmente se 
hacen coincidir con los nombres de las acciones del controlador.
Las vistas parciales a menudo son nombradas comenzando con el carácter de subrayado.

### web

Es el directorio raiz de la web y al que Normalmente apunta un servidor web.

```
assets
css
index.php
index-test.php
```

`assets` contiene archivos publicos como CSS, JavaScript etc. El proceso de publicación es automático por lo que no es necesario
hacer nada con este directorio salvo asegurarse que Yii tiene suficientes permisos como para escribir en el.

`css` contiene archivos CSS planos y es utilizado para los archivos CSS globales que no van a ser comprimidos ni unidos mediante
el administrador de activos (assets manager).

`index.php` es el archivo de inicio de la aplicación web y su punto de entrada. 

`index-test.php` es el punto de entrada para pruebas funcionales (functional testing).

Configurando Composer
-------------------

Después de instalar la plantilla básica de la aplicación es una buena idea ajustar las opciones por defecto de `composer.json`,
archivo que puede ser encontrado en el directorio raiz:

```json
{
    "name": "yiisoft/yii2-app-basic",
    "description": "Yii 2 Basic Application Template",
    "keywords": ["yii", "framework", "basic", "application template"],
    "homepage": "http://www.yiiframework.com/",
    "type": "project",
    "license": "BSD-3-Clause",
    "support": {
        "issues": "https://github.com/yiisoft/yii2/issues?state=open",
        "forum": "http://www.yiiframework.com/forum/",
        "wiki": "http://www.yiiframework.com/wiki/",
        "irc": "irc://irc.freenode.net/yii",
        "source": "https://github.com/yiisoft/yii2"
    },
    "minimum-stability": "dev",
    "require": {
        "php": ">=5.4.0",
        "yiisoft/yii2": "*",
        "yiisoft/yii2-swiftmailer": "*",
        "yiisoft/yii2-bootstrap": "*",
        "yiisoft/yii2-debug": "*",
        "yiisoft/yii2-gii": "*"
    },
    "scripts": {
        "post-create-project-cmd": [
            "yii\\composer\\Installer::setPermission"
        ]
    },
    "extra": {
        "writable": [
            "runtime",
            "web/assets"
        ],
        "executable": [
            "yii"
        ]
    }
}
```

Para empezar, actualizar la información básica. Cambiar los valores de las palabras clave `name`, `description`, `keywords`,
`homepage` and `support` para que se ajusten al nuevo proyecto.

Ahora la parte interesante. Se pueden añadir más paquetes a la aplicación en la sección `require`.
Todos estos paquetes vienen de [packagist.org](https://packagist.org/) así que siéntete libre de navegar por el sitio web 
buscando código útil. 

Después de cambiar `composer.json` ejecutar `php composer.phar update --prefer-dist` para realizar los cambios, esperar hasta que
los paquetes sean descargados e instalados y ya están listos para ser utilizados.
La autocarga de clases será manejada automáticamente.
