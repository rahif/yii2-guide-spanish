Instalación
============

Hay 2 caminos para instalar el framework Yii:

* Via [Composer](http://getcomposer.org/) (recomendado)
* Descargar una plantilla de la aplicación conteniendo todos los requerimientos del sitio, incluyendo el propio framework


Instalando via Composer
---------------------

El camino recomendado para instalar Yii es utilizar el gestor de paquetes [Composer](http://getcomposer.org/). Si no dispone
de composer, puede descargarlo de [http://getcomposer.org/](http://getcomposer.org/) o ejecutar el siguiente comando:

```
curl -s http://getcomposer.org/installer | php
```

Se recomienda encarecidamente una instalación global de Composer.

Para problemas o más información, ver la guia oficial de Composer:

* [Linux](http://getcomposer.org/doc/00-intro.md#installation-nix)
* [Windows](http://getcomposer.org/doc/00-intro.md#installation-windows)

Con Composer instalado, puede crearse un sitio Yii utilizando una de las plantillas de Yii preparadas para usar.
Según sus necesidades, elegir la plantilla adecuada puede ayudar a inicializar un proyecto.

Actualmente, hay 2 plantillas de la aplicación disponibles:

- [Plantilla básica de la aplicación](https://github.com/yiisoft/yii2-app-basic) - Solo un plantilla básica para la parte pública.
- [Plantilla avanzada de la aplicación](https://github.com/yiisoft/yii2-app-advanced) - Consiste en una parte pública, una restringida,
  unos recursos para la parte de consola, partes comunes (código compartido) y soporte para entornos.

Para instrucciones de la instalación de estas plantillas, ver las páginas de los enlaces proporcionados a continuación.
Para leer más sobre las ideas detrás de estas plantillas de la aplicación usos propuestos, remitir a los documentos de
[Plantilla básica de la aplicación](apps-basic.md) y [Plantilla avanzada de la aplicación](apps-advanced.md).

Si no se quiere usar una plantilla y se quiere comenzar desde el principio hay disponible información en el siguiente documento
[creando la estructura de una aplicación propia](apps-own.md). Esto solo es recomendado para usuarios avanzados.


Instalando desde un zip
--------------------

La instalación desde un archivo zip requiere dos pasos:

   1. Descarga de una plantilla de la aplicación desde [yiiframework.com](http://www.yiiframework.com/download/).
   2. Descomprimir el archivo descargado

Si solo se quieren los archivos del framework Yii puede descargarse un archivo ZIP directamente desde 
[github](https://github.com/yiisoft/yii2-framework/releases).
Para crear una aplicación se pueden seguir los pasos descritos en [creating your own application structure](apps-own.md).
Esto solo es recomendado para usuarios avanzados.

> Consejo: El propio framework Yii no necesita ser instalado bajo un directorio accesible via web.
Una aplicación Yii tiene un script de entrada que es usualmente el único archivo que debe ser expuesto
a los usuarios web (es decir, colocado dentro del directorio web). Otros scripts PHP, incluyendo los 
del framework Yii, deben ser protegidos del acceso via web para prevenir posibles ataques por hackers.


Requiremientos
-------------

Después de instalar Yii, puede querer verificar si el servidor cumple los requerimientos de Yii.
Puede hacerlo ejecutando el script comprobador de requerimientos en un navegador o desde la linea de comandos.

Si se ha instalado una plantilla de una aplicación Yii via zip o Composer se encontrará un archivo `requirements.php`
en el directorio base de la aplicación.

Con el fin de ejecutar este script en la línea de comandos utilice el siguiente comando:

```
php requirements.php
```

Con el fin de ejecutar este script en un navegador, debe asegurarse que sea accesible por el servidor web y
acceder a `http://hostname/path/to/yii-app/requirements.php` desde el navegador.
En caso de usar linux se puede crear un "enlace blando" para hacerlo accesible utilizando el siguiente comando:

```
ln -s requirements.php ../requirements.php
```

Para la aplicación con la plantilla avanzada el archivo `requirements.php` esta 2 niveles arriba por lo que se tiene
que utilizar `ln -s requirements.php ../../requirements.php`.


Yii 2 requiere PHP 5.4.0 o superior. Yii ha sido probado con el [Servidor HTTP Apache](http://httpd.apache.org/) y
el [Servidor HTTP Nginx](http://nginx.org/) en Windows y Linux.
Yii también puede funcionar en otros servidores web y en otras plataformas, siempre que PHP 5.4 o superior sea soportado.

Configuración recomendada de Apache
-------------------------------

Yii esta preparado para funcionar en una configuración por defecto del servidor web Apache. Como una medida de seguridad, Yii
viene con los archivos `.htaccess` en las carpetas del framework Yii para denegar el acceso a recursos restringidos.

Por defecto, peticiones para páginas en un sitio basado en Yii van a través del archivo de inicio, normalmente llamado `index.php`,
y situado en el directorio web de la aplicación.
El resultado serán las URLs en el formato `http://hostname/index.php/controller/action/param/value`.

Para ocultar el archivo de inicio en las URLs, añadir las instrucciones `mod_rewrite` al archivo `.htaccess` en la raiz del servidor
(o añadir las instrucciones a la configuración del host virtual en el archivo `httpd.conf` de Apache, en la sección `Directory`
para la directorio raiz de la web). Las instrucciones aplicables son:


~~~
RewriteEngine on

# If a directory or a file exists, use it directly
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
# Otherwise forward it to index.php
RewriteRule . index.php
~~~


Configuración recomendada de Nginx
------------------------------

Yii también puede ser utilizado con el popular servidor web [Nginx](http://wiki.nginx.org/), en tanto PHP haya sido instalado
con [FPM SAPI](http://php.net/install.fpm). Debajo se muestra un ejemplo de la configuración de un host para un sitio
basado en Yii con Nginx.
La configuración dice al servidor que envie todas las peticiones para recursos no existentes a través del archivo de inicio,
dando como resultado las direcciones URL "más bonitas", sin necesidad de referencias a `index.php`.


```
server {
    set $yii_bootstrap "index.php";
    charset utf-8;
    client_max_body_size 128M;

    listen 80; ## listen for ipv4
    #listen [::]:80 default_server ipv6only=on; ## listen for ipv6

    server_name mysite.local;
    root        /path/to/project/web;
    index       $yii_bootstrap;

    access_log  /path/to/project/log/access.log  main;
    error_log   /path/to/project/log/error.log;

    location / {        
        # Redirigir todo lo que no es un archivo real al archivo de inicio de yii incluyendo los argumentos
        try_files $uri $uri/ /$yii_bootstrap?$args;
    }
    
    # descomentar para evitar llamadas de procesamiento por Yii de archivos inexistentes
    #location ~ \.(js|css|png|jpg|gif|swf|ico|pdf|mov|fla|zip|rar)$ {
    #    try_files $uri =404;
    #}
    #error_page 404 /404.html;

    location ~ \.php$ {
        include fastcgi.conf;
        fastcgi_pass   127.0.0.1:9000;
        #fastcgi_pass unix:/var/run/php5-fpm.sock;
    }

    location ~ /\.(ht|svn|git) {
        deny all;
    }
}
```

Cuando esta configuración es utilizada, debe configurarse `cgi.fix_pathinfo=0` en el archivo `php.ini` con el fin de evitar innecesarias
llamadas `stat()` del sistema .

Notar que cuando un servidor HTTPS esta en ejecución es necesario añadir `fastcgi_param HTTPS on;` para que Yii debidamente detecte
si la conexión es segura.
