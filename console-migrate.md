Migración de base de datos
=======================

Como el código fuente, la estructura de una base de datos evoluciona impulsada por el desarrollo y mantenimiento de la aplicación.
Por ejemplo, durante el desarrollo, una nueva tabla puede ser añadida; O después de poner en marcha la aplicación, puede descubrirse que un nuevo índice es requerido.
Es importante mantener el rastro de estos cambios estructurales de la base de datos (llamados **migraciones**), al igual que se hace un seguimiento de los cambios en el código fuente mediante un control de versiones.
Si el código fuente y la base de datos dejan de estar sincronizadas, aparecerán errores, o la aplicación en general puede dejar de funcionar.
Por esta razón, Yii proporciona una herramienta de migración de base de datos que puede mantener el rastro del historial de las migraciones, aplicar nuevas migraciones, o revertir las existentes.

Los siguientes pasos muestran como es utilizada la migración de base de datos por un equipo durante el desarrollo:

1. Tim crea una nueva migración (p.ej. crea una nueva tabla, cambia la definición de una columna, etc.).
2. Tim confirma la nueva migración dentro del código fuente del sistema de control de versiones (p. ej. Git, Mercurial).
3. Doug actualiza su repositorio desde el sistema de control de versiones y recibe la nueva migración.
4. Doug aplica la migración a su base de datos de desarrollo local, de este modo sincroniza su base de datos para reflejar los cambios que hizo Tim.

Yii mantiene la migración de base de datos con `yii migrate`, herramienta de linea de comandos. Esta herramienta proporciona:

* Crear nuevas migraciones.
* Aplicar, revertir y rehacer migraciones.
* Mostrar un histórico de migraciones y migraciones nuevas (migraciones aún no aplicadas).


Creando migraciones
-----------------

Para crear una nueva migración, ejecutar el siguiente comando:

```
yii migrate/create <name>
```

El parámetro requerido `name` especifica una muy breve descripción de la migración. Por ejemplo, si la migración crea una nueva tabla llamada *news*, se utilizará el comando:

```
yii migrate/create create_news_table
```

Como se verá a continuación el parámetro `name` es utilizado como parte del nombre de una clase de PHP en la migración. Por lo tanto, solo debe contener letras, dígitos y/o carácteres de subrayado.

El comando anterior creará un nuevo archivo llamado `m101129_185401_create_news_table.php`. Este archivo será creado dentro del directorio `@app/migrations`.
Inicialmente el archivo de la migración será creado con el siguiente código:

```php
class m101129_185401_create_news_table extends \yii\db\Migration
{
    public function up()
    {
    }

    public function down()
    {
        echo "m101129_185401_create_news_table cannot be reverted.\n";
        return false;
    }
}
```

Fijarse que el nombre de la clase es el mismo que el nombre del archivo, y sigue el patrón `m<timestamp>_<name>`, donde:

* `<timestamp>` se refiere a la marca de tiempo UTC (con el formato `yymmdd_hhmmss`) cuando la migración es creada.
* `<name>` es tomado del parámetro `name` del comando.

En la clase, el método `up()` debe contener el código que implementa la migración actual de la base de datos. En otras palabras, el método `up()` ejecuta el código que realmente cambia la base de datos.
El método `down()` puede contener el código que revierta los cambios que realizados por `up()`.

Algunas veces, es imposible para el método `down()` deshacer la migración de base de datos.
Por ejemplo, si la migración borra filas de una tabla o la tabla entera, esos datos no pueden ser recuperados por el método `down()`.
En estos casos, la migración es llamada irreversible, es decir, la base de datos no se puede revertir a un estado anterior.
Cuando una migración es irreversible, como en el código generado anteriormente, el método `down()` devuelve `false` para indicar que la migración no puede ser revertida.

Como ejemplo, se muestra la migración sobre la creación de una nueva tabla.

```php

use yii\db\Schema;

class m101129_185401_create_news_table extends \yii\db\Migration
{
    public function up()
    {
        $this->createTable('news', [
            'id' => 'pk',
            'title' => Schema::TYPE_STRING . ' NOT NULL',
            'content' => Schema::TYPE_TEXT,
        ]);
    }

    public function down()
    {
        $this->dropTable('news');
    }

}
```

La clase base [\yii\db\Migration] abre una conexión a la base de datos por la propiedad `db`. Puede ser utilizado para manipular los datos y el esquema de una base de datos.

Los tipos de columna utilizados en este ejemplo son tipos abstractos que serán reemplazados por Yii con los correspondientes tipos dependiendo del sistema de gestión de base de datos.
Puede utilizarse para escribir migraciones independientes de la base de datos. 
Por ejemplo, `pk` será reemplazado por `int(11) NOT NULL AUTO_INCREMENT PRIMARY KEY` para MySQL y por `integer PRIMARY KEY AUTOINCREMENT NOT NULL` para sqlite.

Ver la documentación de [[yii\db\QueryBuilder::getColumnType()]] para más detalles y una lista de los tipos disponibles.
También pueden utilizarse las constantes definidas en [[yii\db\Schema]] para definir los tipos de columna.


Migraciones con transacciones
--------------------------

Cuando se realizan complejas migraciones de base de datos, normalmente se quiere asegurar que la migración completa se realizó con éxito o falló para mantener la consistencia e integridad de la base de datos.
Para acometer esta meta, se pueden utilizar transacciones. Pueden utilizarse los métodos especiales `safeUp` y `safeDown` para este propósito.

```php

use yii\db\Schema;

class m101129_185401_create_news_table extends \yii\db\Migration
{
    public function safeUp()
    {
        $this->createTable('news', [
            'id' => 'pk',
            'title' => Schema::TYPE_STRING . ' NOT NULL',
            'content' => Schema::TYPE_TEXT,
        ]);

        $this->createTable('user', [
            'id' => 'pk',
            'login' => Schema::TYPE_STRING . ' NOT NULL',
            'password' => Schema::TYPE_STRING . ' NOT NULL',
        ]);
    }

    public function safeDown()
    {
        $this->dropTable('news');
        $this->dropTable('user');
    }

}
```

Si el código realiza más de un cambio se recomienda utilizar los métodos `safeUp` y `safeDown`.

> Nota: No todos los DBMS soportan transacciones y algunas preguntas de base de datos no pueden estar en una transacción.
> En este caso, se tendrán que implementar los métodos `up()` y `down()` en su lugar.
> Para MySQL, algunas sentencias SQL pueden causar [commit implicito](http://dev.mysql.com/doc/refman/5.1/en/implicit-commit.html).


Aplicando migraciones
-------------------

Para aplicar todas las nuevas migraciones disponibles (p.ej., poner al día la base de datos local), ejecutar el siguiente comando:

```
yii migrate
```

El comando mostrará la lista de todas las nuevas migraciones. Si se confirma aplicar las migraciones, ejecutará el método `up()` en cada nueva clase de migración,
una detrás de otra, en el orden del valor de la marca de tiempo proporcionada en el nombre de la clase.

Después de aplicar una migración, la herramienta de migración mantendrá un registro en una tabla de la base de datos llamada `migration`.
Esto permite a la herramienta identificar que migraciones han sido aplicadas y cuales no.
Si la tabla `migration` no existe, la herramienta automáticamente la creará en la base de datos especificada por el componente de la aplicación `db`.

Algunas veces, puede quererse solo aplicar una o unas pocas nuevas migraciones. Puede utilikzarse el siguiente comando:


```
yii migrate/up 3
```

Este comando aplicará tres migraciones nuevas. Cambiar el valor 3 permitirá cambiar el número de migraciones que van a ser aplicadas.

También puede migrarse la base de datos a una versión específica con el siguiente comando:


```
yii migrate/to 101129_185401
```

Eso es, utilizar la parte del valor de la marca de tiempo del nombre de una migración para especificar la versión a la que se quiere migrar la base de datos.
Si hubiese múltiples migraciones entre la última migración aplicada y la migración especificada, todas serán aplicadas.
Si la migración especificada ha sido aplicada antes, entonces todas las migraciones aplicadas después serán revertidas (será descrito en la próxima sección).


Revertiendo migraciones
--------------------

Para revertir la última o varias migraciones ya aplicadas, puede utilizarse el siguiente comando:


```
yii migrate/down [step]
```

donde el parámetro opcional `step` especifica cuantas migraciones serán revertidas. Por defecto su valor es 1, indicando que revertirá la última migración aplicada.

Como se describió anteriormente, no todas las migraciones pueden ser revertidas. Si una migración no puede ser revertida lanzará una excepción y parará el proceso completo de revertido.


Rehaciendo migraciones
--------------------

Rehacer migraciones quiere decir primero revertir y entonces aplicar las migraciones especificadas.
Esto puede ser realizado con el siguiente comando:


```
yii migrate/redo [step]
```

donde el parámetro opcional `step` especifica cuantas migraciones se ván a rehacer. Por defecto su valor es 1, indicando que se va a rehacer la última migración aplicada.


Mostrando la información de las migraciones
--------------------------------------

Mas allá de aplicar y revertir migraciones, la herramienta de migraciones también puede mostrar el historial de las migraciones y las nuevas migraciones disponibles para ser aplicadas.


```
yii migrate/history [limit]
yii migrate/new [limit]
```

donde el parámetro opcional `limit` especifica el número de migraciones que se van a mostrar. Si `limit` no es especificado, serán mostradas todas las migraciones disponibles.

El primer comando muestra todas las migraciones que han sido aplicadas, mientras que el segundo comado muestra las migraciones que no han sido aplicadas.


Modificando el historial de migraciones
----------------------------------

A veces, puede querer modificarse el historial de migración a una versión de migración específica sin llegar a aplicar o revertir las migraciones pertinentes.
Esto sucede a menudo durante el desarrollo de una nueva migración. Puede utilizarse el siguiente comando para lograr este objetivo.


```
yii migrate/mark 101129_185401
```

Este comando es muy similar al comando `yii migrate/to`, excepto que solo modifica el historial de la tabla de migración a la versión especificada sin aplicar o revertir las migraciones.


Personalizando el comando migrate
-----------------------------

Hay algunos caminos para personalizar el comado `migrate`.


### Utilizar las opciones de la linea de comandos

El comado de migración viene con cuatro opciones que pueden ser especificadas por la linea de comandos: 


* `interactive`: boolean, especifica si efectuar las migraciones en un modo interactivo.  
  Por defecto su valor es `true`, es decir, al usuario se le pedirá confirmación cuando
  realice una migración específica. Se puede configurar a `false` para que las migraciones
  puedan ejecutarse en un proceso en segundo plano.

* `migrationPath`: string, especifies el directorio de almacenamiento de todas las clases de migración.
  Debe ser especificado en terminos de alias de ruta, y el directorio correspondiente debe existir.
  Si no es especificado, se utilizará el subdirectorio `migrations` situado en la ruta base de la aplicación.

* `migrationTable`: string, especifica el nombre de la tabla de la base de datos para almacenar
  la información del historial de migraciones. Por defecto su valor es `migration`. La estructura
  de la tabla es `version varchar(255) primary key, apply_time integer`.

* `connectionID`: string, especifica el ID del componente de la base de datos de la aplicación.
  Por defecto su valor es `db`.

* `templateFile`: string, especifica la ruta del archivo para ser servido como el código
   plantilla para la generación de las clases de migración. Esto debe ser especificado en terminos
   de un alias de ruta (p.ej. `application.migrations.template`). Si no esta configurado, 
   se utilizará una plantilla interna. Dentro de la plantilla, el símbolo `{ClassName}`
   será reemplazado con el nombre de la clase de la migración actual.


Para especificar estas opciones, ejecutar el comando `migrate` utilizando el siguiente formato:

```
yii migrate/up --option1=value1 --option2=value2 ...
```

Por ejemplo, si se quiere migrar un módulo `forum` cuyos archivos de migración están en el directorio `migrations` dentro del módulo, se puede utilizar el siguiente comando:

```
yii migrate/up --migrationPath=@app/modules/forum/migrations
```


### Configuración global del comando

Mientras las opciones de la linea de comandos permite configurar el comado de migración sobre la marcha, algunas veces se puede querer configurar el comando de forma global.
Por ejemplo, se puede querer utilizar una tabla diferente para almacenar el historial de las migraciones, o se puede querer utilizar una plantilla de migración personalizada.
Todo esto puede realizarse modificando la configuración del archivo del comando de consola como se muestra a continuación:

```php
'controllerMap' => [
    'migrate' => [
        'class' => 'yii\console\controllers\MigrateController',
        'migrationTable' => 'my_custom_migrate_table',
    ],
]
```

Ahora al ejecutarse el comando `migrate`, la configuración anterior tomará efecto sin requerir introducir las opciones desde la linea de comandos.
Otras opciones del comando pueden ser también configuradas de esta manera.
