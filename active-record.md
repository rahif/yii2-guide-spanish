Active Record
=============

El [Active Record](http://en.wikipedia.org/wiki/Active_record_pattern) (Registro Activo) provee de una interfaz de programación orientada a objetos para acceder a tablas en las base de datos. 

Una clase Active Record está directamente asociada a una tabla de la base de datos, una instancia de la clase a una fila de la tabla vinculada, y un atributo de la instancia Active Record representa el valor de una columna en la fila. El patrón Active Record existe para que en vez de escribir sentencias SQL, puedas manipular los registros de las tablas en bases de datos, de una manera más orientada a objetos.

Por ejemplo, asumiendo que `Customer` sea una clase Active Record que está asociada con la tabla `customer` y que `name` es una columna de la mencionada tabla. Podrías escribir el siguiente código para insertar una nueva fila en la tabla `customer`:  


```php
$customer = new Customer();
$customer->name = 'Qiang';
$customer->save();
```

El código sería equivalente a usar la siguiente sentencia SQL, la cual es mucho menos intuitiva, más propenso a errorres, y que quizás tenga un problema de incompatibilidad al utilizar diferentes sistemas de gestión de bases de datos (DBMS):


```php
$db->createCommand('INSERT INTO customer (name) VALUES (:name)', [
    ':name' => 'Qiang',
])->execute();
```
A continuación se muestra la lista de las bases de datos que actualmente están soportadas por el Active Record de Yii:

* MySQL 4.1 o posterior: a través de [[yii\db\ActiveRecord]]
* PostgreSQL 7.3 o posterior: a través de [[yii\db\ActiveRecord]]
* SQLite 2 y 3: a través de [[yii\db\ActiveRecord]]
* Microsoft SQL Server 2010 o posterior: a través de [[yii\db\ActiveRecord]]
* Oracle: a través de [[yii\db\ActiveRecord]]
* CUBRID 9.1 o posterior: a través de [[yii\db\ActiveRecord]]
* Sphinx: a través de [[yii\sphinx\ActiveRecord]], requiere la extensión `yii2-sphinx` 
* ElasticSearch: a través de [[yii\elasticsearch\ActiveRecord]], requiere la extensión `yii2-elasticsearch`
* Redis 2.6.12 o posterior: a través de [[yii\redis\ActiveRecord]], requiere la extensión `yii2-redis`
* MongoDB 1.3.0 o posterior: a través de [[yii\mongodb\ActiveRecord]], requiere la extensión `yii2-mongodb`

Como puedes comprobar, Yii proporciona soporte Active Record para bases de datos relacionales, así como bases de datos NoSQL. 

En este tutorial, vamos a describir principalmente el uso de Active Record para bases de datos relacionales.  

Sin embargo, la mayoría del contenido aquí descrito es también aplicable a Active Record para bases de datos NoSQL.


Declarar Clases Active Record
-----------------------------

Para declarar una clase Active necesitas extender [[yii\db\ActiveRecord]] e implementar el método `tableName` que devuelve el nombre de la tabla en la base de datos asociada a la clase:

```php
namespace app\models;

use yii\db\ActiveRecord;

class Customer extends ActiveRecord
{
    /**
     * @return string el nombre de la tabla asociada con esta clase ActiveRecord.
     */
    public static function tableName()
    {
        return 'customer';
    }
}
```


Acceso a la Columna de Datos
----------------------------
Active Record asigna cada columna de la fila de la tabla a un atributo del objeto Active Record. Un atributo se comporta como una propiedad pública del objeto. El nombre del atributo es igual al nombre de la columna correspondiente y distingue entre mayúsculas y minúsculas.

Para leer el valor de una columna, puedes utilizar la siguiente sintaxis:


```php
// "id" y "email" son los nombres de las columnas en la tabla asociada con el objeto Active Record $customer
$id = $customer->id;
$email = $customer->email;
```

Para cambiar el valor de una columna, asigna un nuevo valor a la propiedad asociada y guarda el objeto:

```php
$customer->email = 'jane@example.com';
$customer->save();
```


Conexión a la Base de Datos
---------------------------
Active Record usa [[yii\db\Connection|DB connection]] para intercambiar datos con la base de datos. De forma predeterminada, utiliza el component de aplicación `db` como conexión. Como se explica en Database basics](database-basics.md), puedes configurar el componente `db` en el archivo de configuración de la siguiente forma: 

```php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=localhost;dbname=testdb',
            'username' => 'demo',
            'password' => 'demo',
        ],
    ],
];
```
Si utilizas multiples base de datos en tu aplicación y quieres utilizar una conexión BD diferente para tu clase Active Record, puedes sobreescribir el método [[yii\db\ActiveRecord::getDb()|getDb()]]:

```php
class Customer extends ActiveRecord
{
    // ...

    public static function getDb()
    {
        return \Yii::$app->db2;  // usa el componente de aplicación "db2" 
    }
}
```


Ejecutando Consultas a la Base de Datos
---------------------------------------
Active Record ofrece dos métodos de entrada para la creación de consultas DB y la correspondiente inserción de datos en instancias Active Record: 

 - [[yii\db\ActiveRecord::find()]]
 - [[yii\db\ActiveRecord::findBySql()]]
 
Ambos métodos devuelven una instancia [[yii\db\ActiveQuery]], la cual extiende de [[yii\db\Query]], y por lo tanto soporta el mismo grupo de potentes y flexibles métodos de construcción de consultas, tales como `where()`, `join()`, `orderBy()`, etc. Los siguientes ejemplos muestran algunas de sus posibilidades.

```php
// devuelve todos los clientes *activos* y ordenarlos por su ID:
$customers = Customer::find()
    ->where(['status' => Customer::STATUS_ACTIVE])
    ->orderBy('id')
    ->all();

// devuelve un solo cliente cuyo ID es 1:
$customer = Customer::find()
    ->where(['id' => 1])
    ->one();

// devuelve el número de clientes *activos*:
$count = Customer::find()
    ->where(['status' => Customer::STATUS_ACTIVE])
    ->count();

// para indexar el resultado por los IDs de clientes:
$customers = Customer::find()->indexBy('id')->all();
// el array (matriz) $customers estará indexado por los IDs de clientes

// devuelve todos los clientes usando una sentencia SQL:
$sql = 'SELECT * FROM customer';
$customers = Customer::findBySql($sql)->all();
```

> Consejo: En el código anterior `Customer::STATUS_ACTIVE` es una constante definida en `Customer`. Es una buena práctica utilizar nombres de constantes significativas en lugar de cadenas codificadas o números en el código.

Yii provee de dos métodos de acceso directo para devolver instancias Active Record usando coincidencias en el valor de la clave principal o por un conjunto de valores de columnas: `findOne()` y `findAll()`. El primero devuelve la primera instancia que coincida mientras que la última devuelve todas ellas. Por ejemplo,


```php
// para devolver un solo cliente cuyo ID sea 1:
$customer = Customer::findOne(1);

// para devolver un cliente *activo* cuyo ID es 1:
$customer = Customer::findOne([
    'id' => 1,
    'status' => Customer::STATUS_ACTIVE,
]);

// para devolver clientes cuyos ID sean 1, 2 o 3:
$customers = Customer::findAll([1, 2, 3]);

// para devolver clientes cuyo status sea "deleted":
$customer = Customer::findAll([
    'status' => Customer::STATUS_DELETED,
]);
```


### Recuperar Datos en Arrays

A veces, cuando procesas grandes cantidades de datos, desees usar arrays para almacenar los datos recuperados y así ahorrar memoria. Esto se puede conseguir usando `asArray()`:

```php
// para recuperar clientes en arrays en vez de objetos `Customer`:
$customers = Customer::find()
    ->asArray()
    ->all();
// cada elemento de $customers será un array con una relación nombre-valor

```  


### Recuperar Datos en Lotes

En [Query Builder](query-builder.md), explicamos como puedes usar *batch query* para mantener tu memoria bajo límite cuando debas consultar una gran cantidad de datos de la base de datos. Puedes utilizar la misma técnica en Active Record. Por ejemplo,


```php
// recuperar 10 clientes a la vez
foreach (Customer::find()->batch(10) as $customers) {
    // $customers es un array de 10 o menos de objetos Customer
}
// recuperar 10 clientes a la vez e iterar en bucle uno a uno
foreach (Customer::find()->each(10) as $customer) {
    // $customer es un objeto Customer
}
// Consulta de Lote en carga diligente
foreach (Customer::find()->with('orders')->each() as $customer) {
}
```


Manipulando Registros de la Base de Datos
-----------------------------------------
AR (Registro Activo o Active Record) tiene los siguientes métodos para insertar, actualizar y eliminar una fila de la tabla asociada con una sola instancia AR:
a single Active Record instance:

- [[yii\db\ActiveRecord::save()|save()]]
- [[yii\db\ActiveRecord::insert()|insert()]]
- [[yii\db\ActiveRecord::update()|update()]]
- [[yii\db\ActiveRecord::delete()|delete()]]

AR tambien provee de los siguientes métodos estáticos que se aplican a toda la tabla asociada con la clase Active Record. Tienes que tener un cuidado extremo al utilizar estos métodos ya que afectan a toda la tabla.

Por ejemplo, `deleteAll()` eliminaría TODOS los registros de la tabla:

- [[yii\db\ActiveRecord::updateCounters()|updateCounters()]]
- [[yii\db\ActiveRecord::updateAll()|updateAll()]]
- [[yii\db\ActiveRecord::updateAllCounters()|updateAllCounters()]]
- [[yii\db\ActiveRecord::deleteAll()|deleteAll()]]

Los siguientes ejemplos muestran cómo usar estos métodos:

```php
// para insertar un nuevo registro Customer
$customer = new Customer();
$customer->name = 'James';
$customer->email = 'james@example.com';
$customer->save();  // equivalent to $customer->insert();

// para actualizar un registro existente tipo Customer
$customer = Customer::findOne($id);
$customer->email = 'james@example.com';
$customer->save();  // equivalente a $customer->update();

// para eliminar un registro existente tipo Customer
$customer = Customer::findOne($id);
$customer->delete();

// para incrementar la edad de TODOS los registros tipo customer por 1
Customer::updateAllCounters(['age' => 1]);
```

> Información: El método `save()` llamará `insert()` o `update()`, dependiendo que la instancia AR sea nueva o no (internamente chequeará el valor de [[yii\db\ActiveRecord::isNewRecord]]).
> Si un AR es instanciado a través del operador `new`, llamando `save()` insertará una línea en la tabla; llamando `save()` en un AR devuelto de la base de datos, entonces actualizará la correspondiente fila en la tabla.


### Introducción de Datos y Validación

Puesto que la clase ActiveRecord se extiende de [[yii\base\Model]], soporta todas las características de introducción y validación de datos tal y como se describe en [Model](model.md). Por ejemplo, podrías declarar reglas de validación sobrescribiendo el método [[yii\base\Model::rules()|rules()]]; podrías asignar masivamente los datos de entrada de un usuario a una instancia de AR; y también podrías llamar [[yii\base\Model::validate()|validate()]] para activar la validación de datos.

Cuando llamas `save()`, `insert()` o `update()`, estos métodos llamarán automáticamente [[yii\base\Model::validate()|validate()]]. Si la validación falla, la correspondiente operación para guardar datos será cancelada.

El siguiente ejemplo muestra cómo usar un Active Record para recoger/validar datos de entrada del usuario y guardarlos en la base de datos:

```php
// creando un nuevo registro
$model = new Customer;
if ($model->load(Yii::$app->request->post()) && $model->save()) {
    // los datos de usuario son recogidos, validados y guardados
}

// actualizando un registro cuya clave principal es $id
$model = Customer::findOne($id);
if ($model === null) {
    throw new NotFoundHttpException;
}
if ($model->load(Yii::$app->request->post()) && $model->save()) {
    // los datos de usuario son recogidos, validados y guardados
}
```


### Cargando Valores Predeterminados

Las columnas de tu tabla podrían estar definidas con valores predeterminados. Algunas veces, quizás quieras pre-popular tu formulario Web para un AR con esos valores. Para hacerlo, llama el método `loadDefaultValues()` antes de randerizar el formulario:


```php
$customer = new Customer();
$customer->loadDefaultValues();
// ... mostrar formulario HTML para $customer ...
```


Ciclos de Vida del Active Record
--------------------------------
Es importante comprender los ciclos de vida del AR cuando es usado para manipular registros en la base de datos. Estos ciclos de vida están tipicamente asociados con sus eventos correspondientes, los cuales te permitirán inyectar código para interceptar o responder a esos eventos. Son especialmente útilies para desarrollar comportamientos AR (los llamados [Behaviors](behaviors.md)).

Al crear una nueva instancia Active Record, tendremos los siguientes ciclos de vida:

1. constructor
2. [[yii\db\ActiveRecord::init()|init()]]: activará un evento [[yii\db\ActiveRecord::EVENT_INIT|EVENT_INIT]]

Cuando consultemos datos a través del método [[yii\db\ActiveRecord::find()|find()]], tendremos los siuientes ciclos de vida por CADA populada instancia de AR:

1. constructor
2. [[yii\db\ActiveRecord::init()|init()]]: activará un evento [[yii\db\ActiveRecord::EVENT_INIT|EVENT_INIT]]
3. [[yii\db\ActiveRecord::afterFind()|afterFind()]]: activará un evento [[yii\db\ActiveRecord::EVENT_AFTER_FIND|EVENT_AFTER_FIND]]

Cuando usemos [[yii\db\ActiveRecord::save()|save()]] para insertar o actualizar un AR, tendremos los siguientes ciclos de vida:

1. [[yii\db\ActiveRecord::beforeValidate()|beforeValidate()]]: activará un evento [[yii\db\ActiveRecord::EVENT_BEFORE_VALIDATE|EVENT_BEFORE_VALIDATE]] 
2. [[yii\db\ActiveRecord::afterValidate()|afterValidate()]]: activará un evento [[yii\db\ActiveRecord::EVENT_AFTER_VALIDATE|EVENT_AFTER_VALIDATE]] 
3. [[yii\db\ActiveRecord::beforeSave()|beforeSave()]]: activará un evento [[yii\db\ActiveRecord::EVENT_BEFORE_INSERT|EVENT_BEFORE_INSERT]] o un evento [[yii\db\ActiveRecord::EVENT_BEFORE_UPDATE|EVENT_BEFORE_UPDATE]] 
4. realizar la inserción de datos actual o la actualización
5. [[yii\db\ActiveRecord::afterSave()|afterSave()]]: activará un evento [[yii\db\ActiveRecord::EVENT_AFTER_INSERT|EVENT_AFTER_INSERT]] o un evento [[yii\db\ActiveRecord::EVENT_AFTER_UPDATE|EVENT_AFTER_UPDATE]]. 

Y, finalmente, cuando llamemos [[yii\db\ActiveRecord::delete()|delete()]] para eliminar ActiveRecord,tendremos los siguientes ciclos de vida:

1. [[yii\db\ActiveRecord::beforeDelete()|beforeDelete()]]: activará un evento [[yii\db\ActiveRecord::EVENT_BEFORE_DELETE|EVENT_BEFORE_DELETE]]
2. realizar la eliminación de datos actual
3. [[yii\db\ActiveRecord::afterDelete()|afterDelete()]]: activará un evento [[yii\db\ActiveRecord::EVENT_AFTER_DELETE|EVENT_AFTER_DELETE]]


Working with Relational Data
----------------------------

You can use ActiveRecord to also query a table's relational data (i.e., selection of data from Table A can also pull
in related data from Table B). Thanks to ActiveRecord, the relational data returned can be accessed like a property
of the ActiveRecord object associated with the primary table.

For example, with an appropriate relation declaration, by accessing `$customer->orders` you may obtain
an array of `Order` objects which represent the orders placed by the specified customer.

To declare a relation, define a getter method which returns an [[yii\db\ActiveQuery]] object that has relation
information about the relation context and thus will only query for related records. For example,

```php
class Customer extends \yii\db\ActiveRecord
{
    public function getOrders()
    {
        // Customer has_many Order via Order.customer_id -> id
        return $this->hasMany(Order::className(), ['customer_id' => 'id']);
    }
}

class Order extends \yii\db\ActiveRecord
{
    // Order has_one Customer via Customer.id -> customer_id
    public function getCustomer()
    {
        return $this->hasOne(Customer::className(), ['id' => 'customer_id']);
    }
}
```

The methods [[yii\db\ActiveRecord::hasMany()]] and [[yii\db\ActiveRecord::hasOne()]] used in the above
are used to model the many-one relationship and one-one relationship in a relational database.
For example, a customer has many orders, and an order has one customer.
Both methods take two parameters and return an [[yii\db\ActiveQuery]] object:

 - `$class`: the name of the class of the related model(s). This should be a fully qualified class name.
 - `$link`: the association between columns from the two tables. This should be given as an array.
   The keys of the array are the names of the columns from the table associated with `$class`,
   while the values of the array are the names of the columns from the declaring class.
   It is a good practice to define relationships based on table foreign keys.

After declaring relations, getting relational data is as easy as accessing a component property
that is defined by the corresponding getter method:

```php
// get the orders of a customer
$customer = Customer::findOne(1);
$orders = $customer->orders;  // $orders is an array of Order objects
```

Behind the scene, the above code executes the following two SQL queries, one for each line of code:

```sql
SELECT * FROM customer WHERE id=1;
SELECT * FROM order WHERE customer_id=1;
```

> Tip: If you access the expression `$customer->orders` again, it will not perform the second SQL query again.
The SQL query is only performed the first time when this expression is accessed. Any further
accesses will only return the previously fetched results that are cached internally. If you want to re-query
the relational data, simply unset the existing one first: `unset($customer->orders);`.

Sometimes, you may want to pass parameters to a relational query. For example, instead of returning
all orders of a customer, you may want to return only big orders whose subtotal exceeds a specified amount.
To do so, declare a `bigOrders` relation with the following getter method:

```php
class Customer extends \yii\db\ActiveRecord
{
    public function getBigOrders($threshold = 100)
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id'])
            ->where('subtotal > :threshold', [':threshold' => $threshold])
            ->orderBy('id');
    }
}
```

Remember that `hasMany()` returns an [[yii\db\ActiveQuery]] object which allows you to customize the query by
calling the methods of [[yii\db\ActiveQuery]].

With the above declaration, if you access `$customer->bigOrders`, it will only return the orders
whose subtotal is greater than 100. To specify a different threshold value, use the following code:

```php
$orders = $customer->getBigOrders(200)->all();
```

> Note: A relation method returns an instance of [[yii\db\ActiveQuery]]. If you access the relation like
an attribute (i.e. a class property), the return value will be the query result of the relation, which could be an instance of [[yii\db\ActiveRecord]],
an array of that, or null, depending the multiplicity of the relation. For example, `$customer->getOrders()` returns
an `ActiveQuery` instance, while `$customer->orders` returns an array of `Order` objects (or an empty array if
the query results in nothing).


Relations with Pivot Table
--------------------------

Sometimes, two tables are related together via an intermediary table called [pivot table][]. To declare such relations,
we can customize the [[yii\db\ActiveQuery]] object by calling its [[yii\db\ActiveQuery::via()|via()]] or
[[yii\db\ActiveQuery::viaTable()|viaTable()]] method.

For example, if table `order` and table `item` are related via pivot table `order_item`,
we can declare the `items` relation in the `Order` class like the following:

```php
class Order extends \yii\db\ActiveRecord
{
    public function getItems()
    {
        return $this->hasMany(Item::className(), ['id' => 'item_id'])
            ->viaTable('order_item', ['order_id' => 'id']);
    }
}
```

The [[yii\db\ActiveQuery::via()|via()]] method is similar to [[yii\db\ActiveQuery::viaTable()|viaTable()]] except that
the first parameter of [[yii\db\ActiveQuery::via()|via()]] takes a relation name declared in the ActiveRecord class
instead of the pivot table name. For example, the above `items` relation can be equivalently declared as follows:

```php
class Order extends \yii\db\ActiveRecord
{
    public function getOrderItems()
    {
        return $this->hasMany(OrderItem::className(), ['order_id' => 'id']);
    }

    public function getItems()
    {
        return $this->hasMany(Item::className(), ['id' => 'item_id'])
            ->via('orderItems');
    }
}
```

[pivot table]: http://en.wikipedia.org/wiki/Pivot_table "Pivot table on Wikipedia"


Lazy and Eager Loading
----------------------

As described earlier, when you access the related objects the first time, ActiveRecord will perform a DB query
to retrieve the corresponding data and populate it into the related objects. No query will be performed
if you access the same related objects again. We call this *lazy loading*. For example,

```php
// SQL executed: SELECT * FROM customer WHERE id=1
$customer = Customer::findOne(1);
// SQL executed: SELECT * FROM order WHERE customer_id=1
$orders = $customer->orders;
// no SQL executed
$orders2 = $customer->orders;
```

Lazy loading is very convenient to use. However, it may suffer from a performance issue in the following scenario:

```php
// SQL executed: SELECT * FROM customer LIMIT 100
$customers = Customer::find()->limit(100)->all();

foreach ($customers as $customer) {
    // SQL executed: SELECT * FROM order WHERE customer_id=...
    $orders = $customer->orders;
    // ...handle $orders...
}
```

How many SQL queries will be performed in the above code, assuming there are more than 100 customers in
the database? 101! The first SQL query brings back 100 customers. Then for each customer, a SQL query
is performed to bring back the orders of that customer.

To solve the above performance problem, you can use the so-called *eager loading* approach by calling [[yii\db\ActiveQuery::with()]]:

```php
// SQL executed: SELECT * FROM customer LIMIT 100;
//               SELECT * FROM orders WHERE customer_id IN (1,2,...)
$customers = Customer::find()->limit(100)
    ->with('orders')->all();

foreach ($customers as $customer) {
    // no SQL executed
    $orders = $customer->orders;
    // ...handle $orders...
}
```

As you can see, only two SQL queries are needed for the same task!

> Info: In general, if you are eager loading `N` relations among which `M` relations are defined with `via()` or `viaTable()`,
> a total number of `1+M+N` SQL queries will be performed: one query to bring back the rows for the primary table, one for
> each of the `M` pivot tables corresponding to the `via()` or `viaTable()` calls, and one for each of the `N` related tables.

> Note: When you are customizing `select()` with eager loading, make sure you include the columns that link
> the related models. Otherwise, the related models will not be loaded. For example,

```php
$orders = Order::find()->select(['id', 'amount'])->with('customer')->all();
// $orders[0]->customer is always null. To fix the problem, you should do the following:
$orders = Order::find()->select(['id', 'amount', 'customer_id'])->with('customer')->all();
```

Sometimes, you may want to customize the relational queries on the fly. This can be
done for both lazy loading and eager loading. For example,

```php
$customer = Customer::findOne(1);
// lazy loading: SELECT * FROM order WHERE customer_id=1 AND subtotal>100
$orders = $customer->getOrders()->where('subtotal>100')->all();

// eager loading: SELECT * FROM customer LIMIT 100
//                SELECT * FROM order WHERE customer_id IN (1,2,...) AND subtotal>100
$customers = Customer::find()->limit(100)->with([
    'orders' => function($query) {
        $query->andWhere('subtotal>100');
    },
])->all();
```


Inverse Relations
-----------------

Relations can often be defined in pairs. For example, `Customer` may have a relation named `orders` while `Order` may have a relation
named `customer`:

```php
class Customer extends ActiveRecord
{
    ....
    public function getOrders()
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id']);
    }
}

class Order extends ActiveRecord
{
    ....
    public function getCustomer()
    {
        return $this->hasOne(Customer::className(), ['id' => 'customer_id']);
    }
}
```

If we perform the following query, we would find that the `customer` of an order is not the same customer object
that finds those orders, and accessing `customer->orders` will trigger one SQL execution while accessing
the `customer` of an order will trigger another SQL execution:

```php
// SELECT * FROM customer WHERE id=1
$customer = Customer::findOne(1);
// echoes "not equal"
// SELECT * FROM order WHERE customer_id=1
// SELECT * FROM customer WHERE id=1
if ($customer->orders[0]->customer === $customer) {
    echo 'equal';
} else {
    echo 'not equal';
}
```

To avoid the redundant execution of the last SQL statement, we could declare the inverse relations for the `customer`
and the `orders` relations by calling the [[yii\db\ActiveQuery::inverseOf()|inverseOf()]] method, like the following:

```php
class Customer extends ActiveRecord
{
    ....
    public function getOrders()
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id'])->inverseOf('customer');
    }
}
```

Now if we execute the same query as shown above, we would get:

```php
// SELECT * FROM customer WHERE id=1
$customer = Customer::findOne(1);
// echoes "equal"
// SELECT * FROM order WHERE customer_id=1
if ($customer->orders[0]->customer === $customer) {
    echo 'equal';
} else {
    echo 'not equal';
}
```

In the above, we have shown how to use inverse relations in lazy loading. Inverse relations also apply in
eager loading:

```php
// SELECT * FROM customer
// SELECT * FROM order WHERE customer_id IN (1, 2, ...)
$customers = Customer::find()->with('orders')->all();
// echoes "equal"
if ($customers[0]->orders[0]->customer === $customers[0]) {
    echo 'equal';
} else {
    echo 'not equal';
}
```

> Note: Inverse relation cannot be defined with a relation that involves pivoting tables.
> That is, if your relation is defined with [[yii\db\ActiveQuery::via()|via()]] or [[yii\db\ActiveQuery::viaTable()|viaTable()]],
> you cannot call [[yii\db\ActiveQuery::inverseOf()]] further.


Joining with Relations
----------------------

When working with relational databases, a common task is to join multiple tables and apply various
query conditions and parameters to the JOIN SQL statement. Instead of calling [[yii\db\ActiveQuery::join()]]
explicitly to build up the JOIN query, you may reuse the existing relation definitions and call
[[yii\db\ActiveQuery::joinWith()]] to achieve this goal. For example,

```php
// find all orders and sort the orders by the customer id and the order id. also eager loading "customer"
$orders = Order::find()->joinWith('customer')->orderBy('customer.id, order.id')->all();
// find all orders that contain books, and eager loading "books"
$orders = Order::find()->innerJoinWith('books')->all();
```

In the above, the method [[yii\db\ActiveQuery::innerJoinWith()|innerJoinWith()]] is a shortcut to [[yii\db\ActiveQuery::joinWith()|joinWith()]]
with the join type set as `INNER JOIN`.

You may join with one or multiple relations; you may apply query conditions to the relations on-the-fly;
and you may also join with sub-relations. For example,

```php
// join with multiple relations
// find out the orders that contain books and are placed by customers who registered within the past 24 hours
$orders = Order::find()->innerJoinWith([
    'books',
    'customer' => function ($query) {
        $query->where('customer.created_at > ' . (time() - 24 * 3600));
    }
])->all();
// join with sub-relations: join with books and books' authors
$orders = Order::find()->joinWith('books.author')->all();
```

Behind the scene, Yii will first execute a JOIN SQL statement to bring back the primary models
satisfying the conditions applied to the JOIN SQL. It will then execute a query for each relation
and populate the corresponding related records.

The difference between [[yii\db\ActiveQuery::joinWith()|joinWith()]] and [[yii\db\ActiveQuery::with()|with()]] is that
the former joins the tables for the primary model class and the related model classes to retrieve
the primary models, while the latter just queries against the table for the primary model class to
retrieve the primary models.

Because of this difference, you may apply query conditions that are only available to a JOIN SQL statement.
For example, you may filter the primary models by the conditions on the related models, like the example
above. You may also sort the primary models using columns from the related tables.

When using [[yii\db\ActiveQuery::joinWith()|joinWith()]], you are responsible to disambiguate column names.
In the above examples, we use `item.id` and `order.id` to disambiguate the `id` column references
because both of the order table and the item table contain a column named `id`.

By default, when you join with a relation, the relation will also be eagerly loaded. You may change this behavior
by passing the `$eagerLoading` parameter which specifies whether to eager load the specified relations.

And also by default, [[yii\db\ActiveQuery::joinWith()|joinWith()]] uses `LEFT JOIN` to join the related tables.
You may pass it with the `$joinType` parameter to customize the join type. As a shortcut to the `INNER JOIN` type,
you may use [[yii\db\ActiveQuery::innerJoinWith()|innerJoinWith()]].

Below are some more examples,

```php
// find all orders that contain books, but do not eager loading "books".
$orders = Order::find()->innerJoinWith('books', false)->all();
// which is equivalent to the above
$orders = Order::find()->joinWith('books', false, 'INNER JOIN')->all();
```

Sometimes when joining two tables, you may need to specify some extra condition in the ON part of the JOIN query.
This can be done by calling the [[yii\db\ActiveQuery::onCondition()]] method like the following:

```php
class User extends ActiveRecord
{
    public function getBooks()
    {
        return $this->hasMany(Item::className(), ['owner_id' => 'id'])->onCondition(['category_id' => 1]);
    }
}
```

In the above, the [[yii\db\ActiveRecord::hasMany()|hasMany()]] method returns an [[yii\db\ActiveQuery]] instance,
upon which [[yii\db\ActiveQuery::onCondition()|onCondition()]] is called
to specify that only items whose `category_id` is 1 should be returned.

When you perform query using [[yii\db\ActiveQuery::joinWith()|joinWith()]], the on-condition will be put in the ON part
of the corresponding JOIN query. For example,

```php
// SELECT user.* FROM user LEFT JOIN item ON item.owner_id=user.id AND category_id=1
// SELECT * FROM item WHERE owner_id IN (...) AND category_id=1
$users = User::find()->joinWith('books')->all();
```

Note that if you use eager loading via [[yii\db\ActiveQuery::with()]] or lazy loading, the on-condition will be put
in the WHERE part of the corresponding SQL statement, because there is no JOIN query involved. For example,

```php
// SELECT * FROM user WHERE id=10
$user = User::findOne(10);
// SELECT * FROM item WHERE owner_id=10 AND category_id=1
$books = $user->books;
```


Working with Relationships
--------------------------

ActiveRecord provides the following two methods for establishing and breaking a
relationship between two ActiveRecord objects:

- [[yii\db\ActiveRecord::link()|link()]]
- [[yii\db\ActiveRecord::unlink()|unlink()]]

For example, given a customer and a new order, we can use the following code to make the
order owned by the customer:

```php
$customer = Customer::findOne(1);
$order = new Order();
$order->subtotal = 100;
$customer->link('orders', $order);
```

The [[yii\db\ActiveRecord::link()|link()]] call above will set the `customer_id` of the order to be the primary key
value of `$customer` and then call [[yii\db\ActiveRecord::save()|save()]] to save the order into database.


Scopes
------

When you call [[yii\db\ActiveRecord::find()|find()]] or [[yii\db\ActiveRecord::findBySql()|findBySql()]], it returns an
[[yii\db\ActiveQuery|ActiveQuery]] instance.
You may call additional query methods, such as [[yii\db\ActiveQuery::where()|where()]], [[yii\db\ActiveQuery::orderBy()|orderBy()]],
to further specify the query conditions.

It is possible that you may want to call the same set of query methods in different places. If this is the case,
you should consider defining the so-called *scopes*. A scope is essentially a method defined in a custom query class that
calls a set of query methods to modify the query object. You can then use a scope like calling a normal query method.

Two steps are required to define a scope. First create a custom query class for your model and define the needed scope
methods in this class. For example, create a `CommentQuery` class for the `Comment` model and define the `active()`
scope method like the following:

```php
namespace app\models;

use yii\db\ActiveQuery;

class CommentQuery extends ActiveQuery
{
    public function active($state = true)
    {
        $this->andWhere(['active' => $state]);
        return $this;
    }
}
```

Important points are:

1. Class should extend from `yii\db\ActiveQuery` (or another `ActiveQuery` such as `yii\mongodb\ActiveQuery`).
2. A method should be `public` and should return `$this` in order to allow method chaining. It may accept parameters.
3. Check [[yii\db\ActiveQuery]] methods that are very useful for modifying query conditions.

Second, override [[yii\db\ActiveRecord::find()]] to use the custom query class instead of the regular [[yii\db\ActiveQuery|ActiveQuery]].
For the example above, you need to write the following code:

```php
namespace app\models;

use yii\db\ActiveRecord;

class Comment extends ActiveRecord
{
    /**
     * @inheritdoc
     * @return CommentQuery
     */
    public static function find()
    {
        return new CommentQuery(get_called_class());
    }
}
```

That's it. Now you can use your custom scope methods:

```php
$comments = Comment::find()->active()->all();
$inactiveComments = Comment::find()->active(false)->all();
```

You can also use scopes when defining relations. For example,

```php
class Post extends \yii\db\ActiveRecord
{
    public function getActiveComments()
    {
        return $this->hasMany(Comment::className(), ['post_id' => 'id'])->active();

    }
}
```

Or use the scopes on-the-fly when performing relational query:

```php
$posts = Post::find()->with([
    'comments' => function($q) {
        $q->active();
    }
])->all();
```

### Default Scope

If you used Yii 1.1 before, you may know a concept called *default scope*. A default scope is a scope that
applies to ALL queries. You can define a default scope easily by overriding [[yii\db\ActiveRecord::find()]]. For example,

```php
public static function find()
{
    return parent::find()->where(['deleted' => false]);
}
```

Note that all your queries should then not use [[yii\db\ActiveQuery::where()|where()]] but
[[yii\db\ActiveQuery::andWhere()|andWhere()]] and [[yii\db\ActiveQuery::orWhere()|orWhere()]]
to not override the default condition.


Transactional operations
------------------------

When a few DB operations are related and are executed

TODO: FIXME: WIP, TBD, https://github.com/yiisoft/yii2/issues/226

,
[[yii\db\ActiveRecord::afterSave()|afterSave()]], [[yii\db\ActiveRecord::beforeDelete()|beforeDelete()]] and/or [[yii\db\ActiveRecord::afterDelete()|afterDelete()]] life cycle methods. Developer may come
to the solution of overriding ActiveRecord [[yii\db\ActiveRecord::save()|save()]] method with database transaction wrapping or
even using transaction in controller action, which is strictly speaking doesn't seem to be a good
practice (recall "skinny-controller / fat-model" fundamental rule).

Here these ways are (**DO NOT** use them unless you're sure what you are actually doing). Models:

```php
class Feature extends \yii\db\ActiveRecord
{
    // ...

    public function getProduct()
    {
        return $this->hasOne(Product::className(), ['product_id' => 'id']);
    }
}

class Product extends \yii\db\ActiveRecord
{
    // ...

    public function getFeatures()
    {
        return $this->hasMany(Feature::className(), ['id' => 'product_id']);
    }
}
```

Overriding [[yii\db\ActiveRecord::save()|save()]] method:

```php

class ProductController extends \yii\web\Controller
{
    public function actionCreate()
    {
        // FIXME: TODO: WIP, TBD
    }
}
```

Using transactions within controller layer:

```php
class ProductController extends \yii\web\Controller
{
    public function actionCreate()
    {
        // FIXME: TODO: WIP, TBD
    }
}
```

Instead of using these fragile methods you should consider using atomic scenarios and operations feature.

```php
class Feature extends \yii\db\ActiveRecord
{
    // ...

    public function getProduct()
    {
        return $this->hasOne(Product::className(), ['product_id' => 'id']);
    }

    public function scenarios()
    {
        return [
            'userCreates' => [
                'attributes' => ['name', 'value'],
                'atomic' => [self::OP_INSERT],
            ],
        ];
    }
}

class Product extends \yii\db\ActiveRecord
{
    // ...

    public function getFeatures()
    {
        return $this->hasMany(Feature::className(), ['id' => 'product_id']);
    }

    public function scenarios()
    {
        return [
            'userCreates' => [
                'attributes' => ['title', 'price'],
                'atomic' => [self::OP_INSERT],
            ],
        ];
    }

    public function afterValidate()
    {
        parent::afterValidate();
        // FIXME: TODO: WIP, TBD
    }

    public function afterSave($insert)
    {
        parent::afterSave($insert);
        if ($this->getScenario() === 'userCreates') {
            // FIXME: TODO: WIP, TBD
        }
    }
}
```

Controller is very thin and neat:

```php
class ProductController extends \yii\web\Controller
{
    public function actionCreate()
    {
        // FIXME: TODO: WIP, TBD
    }
}
```

Optimistic Locks
----------------

TODO

Dirty Attributes
----------------

TODO

See also
--------

- [Model](model.md)
- [[yii\db\ActiveRecord]]
