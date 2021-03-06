---
layout   : post
title    : Late static binding en PHP
summary  : Resolver referencias estáticas en tiempo de ejecución
tags     : PHP late-static-binding
category : software
permalink: /blog/Late-static-binding-php
disqus   : true
---

Si has programado un poco en PHP te habrás dado cuenta de que su lado
orientado a objetos difiere un poco de lo que cabría esperar en cuanto
al uso de variables y métodos estáticos. Afortunadamente, los chicos
de PHP arreglaron muchas de esas inconsistencias en el lanzamiento de
la [versión 5.3] (hace ya más de 4 años).

Una de ellas era que las referencias estáticas dentro de una clase,
como `self::` o `__CLASS__`, se resuelven empleando
el nombre de la clase, en tiempo de compilación. Usaré el patrón
Singleton para poner de manifiesto el problema y su solución.

<em>
   Los ejemplos siguientes son solo ilustrativos, en general el uso
   del patrón Singleton [está desaconsejado], entre otras cosas porque
   incrementa el acoplamiento entre clases y hace difícil hacer
   tests de unidad (o al menos eso he leído, tampoco estoy del
   todo convencido).
</em>

<div class="subhead">Singleton</div>

Es considerado un patrón *creacional* ya que interviene
en la creación de objetos. Su utilidad consiste básicamente en garantizar
que una clase solo tenga una instancia. El ejemplo típico es utilizar un
Singleton como el gestor de conexiones a una base de datos:

{% highlight php %}
<?php
   class DBManager {
      private static $instance;
      protected static $klass = __CLASS__;
      private static $count = 0;

      private function __construct() {}

      public static function getInstance() {
         if (self::$instance == null) {
            self::$instance = new self::$klass;
            self::$count += 1;
         }
         return self::$instance;
      }

      public static function count() {
         return self::$count;
      }
   }
?>
{% endhighlight %}

Algunos detalles: usamos la variable `$instance` para guardar la única
instancia de la clase `DBManager`. Es una *variable de clase* porque
está precedida del modificador `static`.
El constructor de la clase es privado, luego nadie puede usar el constructor
para crear instancias de la clase (de otra manera no tendríamos un Singleton).

Es con el método `getInstance()` con el que obtenemos la única instancia
de la clase.
El método `count()` lo he incluido solamente para demostrar que el
Singleton funciona:

{% highlight php %}
<?php
   $db = DBManager::getInstance();
   assert (DBManager::count() == 1);
   $db2 = DBManager::getInstance();
   assert (DBManager::count() == 1); // Siempre es 1
?>
{% endhighlight %}

#### Late Static Binding

Pero esto empieza a complicarse si decides especializar el Singleton para dar soporte
a múltiples bases de datos:

{% highlight php %}
<?php
   class MySQLManager extends DBManager {
      protected static $klass = __CLASS__;
   }

   $mysqldb = MySQLManager::getInstance();
   assert (get_class($mysqldb) == 'MySQLManager'); // Nop
?>
{% endhighlight %}

La clase `DBManager` declara la variable
`$klass` con la ilusión de que las clases que hereden de ella puedan sobreescribirla.
Esto es lo que hace `MySQLManager`: provee su propia definición de `$klass`.
El funcionamiento ideal sería que una vez instanciada `MySQLManager`, la variable
`$klass` tomase el valor "MySQLManager". Pero esto no
pasa, la última aserción falla porque `get_class($mysqldb)` devuelve "DBManager".

El problema es la palabra reservada `self`. `self` se resuelve en **tiempo de compilación**
y toma el valor de la clase en la que se define. De modo que da igual si alguna
subclase sobreescribe `$klass` porque `DBManager` siempre usa `self::$klass`, es decir,
`DBManager::$klass`.

La solución es reemplazar `self` por `static` cuando creamos
la instancia (disponible a partir de la versión 5.3):
{% highlight php %}
<?php
   class DBManager {
      private static $instance;
      protected static $klass = __CLASS__;
      private function __construct() {}

      public static function getInstance() {
         if (self::$instance == null) {
            self::$instance = new static::$klass;
         }
         return self::$instance;
      }
   }

   class MySQLManager extends DBManager {
      protected static $klass = __CLASS__;
   }

   $mysqldb = MySQLManager::getInstance();
   assert (get_class($mysqldb) == 'MySQLManager'); // Ok
?>
{% endhighlight %}

La palabra reservada `static` se resuelve en **tiempo de ejecución**.
De esta manera cuando instanciemos `MySQLManager` el valor de `static::$klass`
se resuelve lo más tarde posible (tomando el valor definido en la subclase).


[versión 5.3]: http://php.net/releases/5_3_0.php
[está desaconsejado]: http://www.ibm.com/developerworks/webservices/library/co-single/index.html