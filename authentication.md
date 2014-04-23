Autenticación
============

Autenticación es el acto de verificar que un usuario es quien dice ser, y es la base del proceso de inicio de sesión. 
Normalmente, la autenticación utiliza la combinación de un identificador --un usuario o dirección de email -- y una clave.
El usuario envia estos valores a través de un formulario y la aplicación compara esta información con la previamente
almacenada (p. ej., al registrarse).

En Yii, todo este proceso se realiza de forma semiautomática, dejando al desarrollador meramente implementar 
[[yii\web\IdentityInterface]], la clase más importante en el sistema de autenticación.
Normalmente, la implementación de `IdentityInterface` es acometida utilizando el modelo `User`.

Se puede encontrar un ejemplo con todas las funciones de autenticación en la
[Plantilla avanzada de la aplicación](installation.md).
A continuación se muestra un listado con los métodos del interface:

```php
class User extends ActiveRecord implements IdentityInterface
{
    // ...

    /**
     * Encontrar una identificacion por el ID dado.     
     *
     * @param string|integer $id the ID to be looked for
     * @return IdentityInterface|null the identity object that matches the given ID.
     */
    public static function findIdentity($id)
    {
        return static::findOne($id);
    }

    /**
     * Encontrar una identificacion por el token dado.     
     *
     * @param string $token the token to be looked for
     * @return IdentityInterface|null the identity object that matches the given token.
     */
    public static function findIdentityByAccessToken($token)
    {
        return static::findOne(['access_token' => $token]);
    }

    /**
     * @return int|string current user ID
     */
    public function getId()
    {
        return $this->id;
    }

    /**
     * @return string current user auth key
     */
    public function getAuthKey()
    {
        return $this->auth_key;
    }

    /**
     * @param string $authKey
     * @return boolean if auth key is valid for current user
     */
    public function validateAuthKey($authKey)
    {
        return $this->getAuthKey() === $authKey;
    }
}
```

Dos de los métodos descritos son simples: `findIdentity` al que se proporciona un valor ID y devuelve una instancia del modelo
asociado con ese ID. El método `getId` devuelve el ID.
Los otros dos métodos -- `getAuthKey` y `validateAuthKey` -- son utilizados para proporcionar mayor seguridad a la cookie "remember me".
El método `getAuthKey` debe devolver una cadena que es única para cada usuario. Se puede crear de forma fiable una cadena única usando
`Security::generateRandomKey()`.
Es una buena idea grabar también esta parte del registro de usuario:

```php
public function beforeSave($insert)
{
    if (parent::beforeSave($insert)) {
        if ($this->isNewRecord) {
            $this->auth_key = Security::generateRandomKey();
        }
        return true;
    }
    return false;
}
```

El método `validateAuthKey` solo necesita comparar la variable `$authKey` pasada como parámetro (recuperada de una cookie), con el valor
obtenido de la base de datos.
