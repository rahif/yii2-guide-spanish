Memoria caché
============

La memoria caché (desde ahora caché) es una forma barata y efectiva de mejorar la eficiencia de una aplicación web. Almacenando datos estáticos
en caché y sirviendolos cuando son solicitados, la aplicación ahorra el tiempo que tarda en generar los datos desde el principio.
Almacenar datos en caché es una de las mejores maneras de mejorar la eficiencia de una aplicación, casi obligatorio en un sitio web de gran tamaño.

Conceptos básicos
---------------

Usar caché en Yii requiere configurar y acceder al componente cache de la aplicación. La siguiente configuración 
de la aplicación especifica un componente de caché que utiliza [memcached](http://memcached.org/) con dos
servidores de caché. Nota, esta configuración debe ser hecha en el fichero localizado en el alias `@app/config/web.php`
si se esta utilizando la plantilla básica de la aplicación.

```php
'components' => [
    'cache' => [
        'class' => '\yii\caching\MemCache',
        'servers' => [
            [
                'host' => 'server1',
                'port' => 11211,
                'weight' => 100,
            ],
            [
                'host' => 'server2',
                'port' => 11211,
                'weight' => 50,
            ],
        ],
    ],
],
```

Cuando la aplicación esta funcionando, el componente cache puede ser accedido mediante llamadas a `Yii::$app->cache`.

Yii proporciona varios componentes de caché que pueden almacenar datos en diferentes medios. A continuación
se muestra un listado con los componentes de caché disponibles:

* [[yii\caching\ApcCache]]: utiliza la extensión de PHP [APC](http://php.net/manual/en/book.apc.php). Esta opción puede
  ser considerada como la más rápida de entre todas las disponibles para una aplicación centralizada. (p. ej. un servidor,
  no dedicado balance de carga, etc).

* [[yii\caching\DbCache]]: utiliza una tabla de base de datos para almacenar los datos. Por defecto, se creará y usará
  como base de datos [SQLite3](http://sqlite.org/) en el directorio runtime. Se puede especificar explicitamente que base
  de datos va a ser utilizada configurando la propiedad `db`.

* [[yii\caching\DummyCache]]: dummy cache (caché tonta) que no almacena en caché nada. El propósito de este componente
  es simplificar el código necesario para chequear la disponibilidad de caché. Por ejemplo, durante el desarrollo o
  si el servidor no tiene soporte de caché actualmente, puede utilizarse este componente de caché. Cuando este disponible
  un soporte en caché, puede cambiarse el componente correspondiente. En ambos casos, puede utilizarse el mismo código
  `Yii::$app->cache->get($key)` para recuperar un dato sin la preocupación de que `Yii::$app->cache` pueda ser `null`.

* [[yii\caching\FileCache]]: utiliza un fichero estándar para almacenar los datos. Esto es adecuado para almacenar
  grandes bloques de datos (como páginas).

* [[yii\caching\MemCache]]: utiliza las extensiones de PHP [memcache](http://php.net/manual/en/book.memcache.php)
  y [memcached](http://php.net/manual/en/book.memcached.php). Esta opción puede ser considerada como la más rápida
  cuando la caché es manejada en una aplicación distribuida (p. ej. con varios servidores, con balance de carga, etc..)

* [[yii\redis\Cache]]: implementa un componente de caché basado en [Redis](http://redis.io/) que almacenan pares 
  clave-valor (requiere la versión 2.6.12 de redis).

* [[yii\caching\WinCache]]: utiliza la extensión de PHP [WinCache](http://iis.net/downloads/microsoft/wincache-extension)
  ([ver también](http://php.net/manual/en/book.wincache.php)).

* [[yii\caching\XCache]]: utiliza la extensión de PHP [XCache](http://xcache.lighttpd.net/).

* [[yii\caching\ZendDataCache]]: utiliza
  [Zend Data Cache](http://files.zend.com/help/Zend-Server-6/zend-server.htm#data_cache_component.htm)
  como el medio fundamental de caché.

Consejo: Porque todos estos componentes de caché extienden de la misma clase base [[yii\caching\Cache]], se puede cambiar a un 
tipo diferente sin modificar el código que esta utilizando la memoria caché.

Almacenar datos en caché puede ser utilizado en diferentes niveles.
En el nivel más bajo, almacenar un dato, como una variable. 
En un nivel superior, almacenar en caché un fragmento de una página que es generado por el script de la vista.
En el nivel más alto, almacenar la página completa en caché y sirviendola de caché cuando es necesitada. 

En las siguientes subsecciones se elabora como usar caché en estos niveles.

Nota, por definición, la caché es un medio de almacenamiento volátil. No se asegura la existencia de datos
en caché incluso si los datos no han expirado. Por lo tanto, no usar caché como un medio de almacenamiento
persistente (p. ej. no guardar datos de sesión u otra información importante)

Data Caching
------------

Data caching es almacenar algunas variables de PHP en caché y recuperarlas de ahí mas tarde. Para este propósito,
la clase base del componente cache [[yii\caching\Cache]] proporciona dos métodos que son utilizados la mayoría de las veces:
[[yii\caching\Cache::set()|set()]] y [[yii\caching\Cache::get()|get()]].
Nota, solo variables y objetos serializables pueden ser almacenados en caché con éxito.

Para almacenar una variable `$value` en caché, se elige una clave única `$key` y se llama a [[yii\caching\Cache::set()|set()]] 
para almacenarla:

```php
Yii::$app->cache->set($key, $value);
```

Los datos almacenados en caché permanecerán para siempre a menós que sean borrados por alguna política de caché
(p. ej. si el espacio de caché esta lleno los datos más antiguos son eliminados). Para cambiar este comportamiento,
se puede suministrar un parámetro de expiración cuando se llama a [[yii\caching\Cache::set()|set()]] de esta manera los datos
serán eliminados de la caché después de un cierto periodo de tiempo:

```php
// Mantener en caché al menos 45 segundos
Yii::$app->cache->set($key, $value, 45);
```

Cuando se necesite acceder a esta variable más tarde (en la misma o en una diferente petición web), [[yii\caching\Cache::get()|get()]]
es llamada con la clave `$key` para ser recuperada de caché. Si el valor recuperado es `false`, significa que el valor no esta disponible
en caché y debe volver a ser calculado.

```php
public function getCachedData()
{    
    $key = /* key es una clave y debe ser única para el acceso a caché */;
    $value = Yii::$app->cache->get($key);
    if ($value === false) {
        $value = /* volver a calcular el valor aquí porque el dato no esta en caché y hay que guardarlo para usos posteriores*/;
        Yii::$app->cache->set($key, $value);
    }
    return $value;
}
```

Este es el patrón común de un dato arbitrario en caché para uso general.

Cuando se elige la clave para que una variable sea almacenada en caché, debe asegurarse que sea única entre todas las claves
utilizadas por la aplicación en el almacenamiento de caché. **NO** es requerido que la clave sea única entre todas las aplicaciones
porque el componente de cache es suficientemente inteligente para diferenciar las claves de diferentes aplicaciones.

Algunos almacenamientos de caché, como MemCache, APC, soportan la recuperación de multiples valores almacenados en caché
en modo por lotes, lo cual puede reducir la sobrecarga envuelta en recuperar los datos almacenados. Un método llamado 
[[yii\caching\Cache::mget()|mget()]] es proporcionado para aprovechar esta característica. En el caso de que el almacenamiento
de caché no soporte esta característica, [[yii\caching\Cache::mget()|mget()]] lo simulará.

Para borrar un valor de caché llamar a [[yii\caching\Cache::delete()|delete()]]; y para borrar todo de caché,
llamar a [[yii\caching\Cache::flush()|flush()]].

Se debe ser extremadamente cuidadoso con las llamandas a [[yii\caching\Cache::flush()|flush()]] ya que borra los datos
almacenados en caché de otras aplicaciones si la caché es compartida entre las aplicaciones.

Nota, porque [[yii\caching\Cache]] implementa `ArrayAccess`, un componente de caché puede ser usado como un array.
Debajo hay algunos ejemplos:

```php
$cache = Yii::$app->cache;
$cache['var1'] = $value1;  // equivalente a: $cache->set('var1', $value1);
$value2 = $cache['var2'];  // equivalente a: $value2 = $cache->get('var2');
```

### Dependencia de caché

Además de configurar el tiempo de expiración, los datos almacenados en caché pueden también ser invalidados conforme
a algunos cambios en las dependencias. Por ejemplo, si se almacena en caché el contenido de un archivo y el archivo cambia,
los datos en caché deben ser invalidados y hay que leer el contenido del archivo en lugar de los datos almacenados en caché.

Una dependencia es representada como una instancia de [[yii\caching\Dependency]] o su clase hija. Pasar la instancia
de la dependencia con los datos para que sea almacenada en caché cuando se llama a [[yii\caching\Cache::set()|set()]].

```php
use yii\caching\FileDependency;

// el valor expirará en 30 segundos
// puede ser invalidado antes si el fichero dependiente es modificado
Yii::$app->cache->set($id, $value, 30, new FileDependency(['fileName' => 'example.txt']));
```

Ahora si se recupera $value de caché mediante una llamada a `get()`, la dependencia será evaluada y si ha cambiado,
devolverá el valor falso, indicando que el dato necesita volver a calcularse.

Debajo se muestra un sumario de las dependencias de caché disponibles:

- [[yii\caching\FileDependency]]: la dependencia invalida el dato en caché si el tiempo de la última modificación del fichero ha cambiado.
- [[yii\caching\GroupDependency]]: marca un elemento de datos almacenado en caché con un nombre de grupo. Puede invalidarse todos
  los elementos de datos almacenados en caché con el mismo nombre de grupo a la vez llamando a [[yii\caching\GroupDependency::invalidate()]]. 
- [[yii\caching\DbDependency]]: la dependencia invalida el dato en caché si el resultado de la consulta SQL especificada ha cambiado.
- [[yii\caching\ExpressionDependency]]: la dependencia invalida el dato en caché si el resultado de la expresión en PHP ha cambiado.

### Consultas en Caché

Para almacenar en caché el resultado de varias consultas a base de datos se puede envolver mediante llamadas a 
[[yii\db\Connection::beginCache()]] y [[yii\db\Connection::endCache()]]:

```php
$connection->beginCache(60); // almacenar en caché todos los resultados de las consultas durante 60 segundos.
// código de consultas a la base de datos aquí...
$connection->endCache();
```

Fragmento en Caché
----------------

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.fragment

### Opciones de Caché

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.fragment#caching-options

### Anidamiento de Caché

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.fragment#nested-caching

Contenido dinámico
----------------

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.dynamic

Página en Caché
-------------

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.page

### Salida en Caché

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.page#output-caching

### HTTP en Caché

TBD: http://www.yiiframework.com/doc/guide/1.1/en/caching.page#http-caching
