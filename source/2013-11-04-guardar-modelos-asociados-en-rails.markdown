---
title: "Guardar modelos asociados en Rails"
date: 2013-11-04
tags: [Rails, ActiveRecord, Asociaciones]
---

Como desarrolladores Rails, solemos olvidarnos de la capa de persistencia gracias a que ActiveRecord se encarga de _mapear_ objetos Ruby a tablas de la base de datos y viceversa (a esto se le conoce como *Mapeo objeto_relacional*). Nosotros trabajamos con objetos Ruby, con la lógica de negocios que contienen, y delegamos por completo al _framework_ la forma en la que esos objetos se guardan (salvo en ocasiones especiales, que tenemos que ensuciarnos las manos con SQL). Sin embargo, como toda herramienta, es necesario conocer cómo trabaja para poder sacarle el mayor provecho, sobre todo en ocasiones especiales, donde el comportamiento no es siempre el esperado. La intensión del post es conocer cómo se comportan los modelos asociados cuando se realizan asignaciones, ya que en ciertas ocasiones la asociación se guarda automáticamente y en otras no. También veremos cómo 'forzar' el comportamiento que deseamos.

Aunque el tema principal del post es verificar el comportamiento de las asociaciones para casos específicos, en los ejemplos también se cubrirán aspectos más básicos para que una persona con escaso conocimiento de asociaciones en Rails pueda seguirlos sin problema. De cualquier forma, si se requiere documentación básica más detallada, la [guía de asociaciones en Rails](http://guides.rubyonrails.org/association_basics.html) es un buen lugar para comenzar.

# Las asociaciones

Imaginemos que tenemos los siguientes modelos asociados:

```ruby
#app/models/empleado.rb
class Empleado < ActiveRecord::Base
  has_one :puesto
  has_many :computadoras
  has_and_belongs_to_many :proyectos
end

#app/models/puesto.rb
class Puesto < ActiveRecord::Base
  belongs_to :empleado
end

#app/models/computadora.rb
class Computadora < ActiveRecord::Base
  belongs_to :empleado
end

#app/models/proyecto.rb
class Proyecto < ActiveRecord::Base
  has_and_belongs_to_many :empleados
end
```

Como podemos ver, tenemos cuatro modelos con diversas asociaciones: uno a uno, uno a muchos y muchos a muchos. Por comodidad, vamos a pasar por alto las validaciones de estos modelos y centrarnos únicamente en las relaciones.

# Creación de modelos

Para empezar, veamos cómo crear un `Empleado`. Esto lo podemos hacer en dos pasos:

```ruby
empleado = Empleado.new nombre_completo: 'Juan Pérez'
empleado.save
```

O bien, en uno solo:

```ruby
empleado = Empleado.create nombre_completo: 'Juan Pérez'
```

En el primer ejemplo, se construye el modelo `Empleado` y se salva después. En el segundo ejemplo se construye el modelo `Empleado` y se guarda automáticamente en la base de datos.

También podemos crear por separado un puesto, un proyecto y una computadora.

```ruby
puesto = Puesto.create nombre: 'Programador'
proyecto = Proyecto.create nombre: 'Proyecto Importante'
computadora = Computadora.create modelo: 'laptop 15"'
```

Hasta aquí, hemos creado custro modelos por separado y todo ha funcionado correctamente. Es momento de asociarlos.

# Asociación uno a muchos

Empezaremos analizando cómo se comporta la relación uno a muchos al momento de realizar una asignación. Rails nos permite asociar nuestros modelos por medio de los ids (*foreign_keys*), pero una forma más natural es asociar los modelos directamente. Como ya tenemos creados nuestros modelos, basta con asignar uno al otro. Ahora bien, las asignaciones de uno a muchos pueden hacerse de dos maneras: asignar el empleado a la computadora, o bien, agregar la computadora a la lista de computadoras que tiene el empleados. Empecemos por hacer lo primero:

```ruby
computadora.empleado = empleado
```

Una vez hecha la asignación, verificamos que efectivamente estén asociados.

```ruby
computadora.empleado
# => #<Empleado id: 1, nombre_completo: "Juan Pérez">
empleado.computadoras
# => #<ActiveRecord::Associations::CollectionProxy []>
```

La asignación funcionó correctamente, pero al tratar de acceder a la asociación contraria, nos regresa un arreglo vacío.

## Guardado manual

La razón por la que la lista de computadoras regresó vacía es sencilla: no hemos salvado nuestra relación, por lo que los cambios todavía no se han reflejado en la base de datos y por tanto, el empleado no se ha enterado que tiene una computadora asignada. Salvemos el modelo y veamos cómo se comporta.

```ruby
computadora.save
Computadora.find(1).empleado
# => #<Empleado id: 1, nombre_completo: "Juan Pérez">
Empleado.find(1).computadoras
# => [#<Computadora id: 1, modelo: 'laptop 15"', empleado_id: 1>]
```

Hecho esto, las asociaciones funcionan correctamente. Para comprobarlo, hemos consultado directamente la información de la base de datos en lugar de los objetos que tenemos en memoria, y la asociación se mantiene ya que se ha guardado en la tabla correspondiente. Eso también lo podemos ver en el campo `empleado_id` de la computadora, que ahora tiene el id del empleado (en este caso, 1).

## Guardado automático

Veamos ahora cómo se comporta la asignación si la hacemos al revés, es decir, agregando una computadora a la lista de computadoras de un empleado.

```ruby
computadora_2 = Computadora.create modelo: 'desktop 24"'
empleado.computadoras << computadora_2
Computadora.find(2).empleado
# => #<Empleado id: 1, nombre_completo: "Juan Pérez">
Empleado.find(1).computadoras
# => [#<Computadora id: 1, modelo: 'laptop 15"', empleado_id: 1>,
#     #<Computadora id: 2, modelo: 'desktop 24"', empleado_id: 1>]
```

Lo primero que hicimos fue crear una nueva computadora y luego agregarla a la lista de computadoras del empleado, después comprobamos el estado de las asociaciones y todo funciona correctamente. Lo interesante es que no necesitamos guardar ninguno de los dos modelos después de la asociación, Rails lo hizo automáticamente por nosotros.

## Asignación de un modelo que no ha sido guardado

Veamos un tercer caso: en los dos casos anteriores, el modelo computadora ya existía en la base de datos, ahora veremos qué sucede con un modelo que aún no ha sido guardado:

```ruby
empleado.computadoras << Computadora.new(modelo: 'netbook')
Computadora.find(3).empleado
# => #<Empleado id: 1, nombre_completo: "Juan Pérez">
Empleado.find(1).computadoras
# => [#<Computadora id: 1, modelo: 'laptop 15"', empleado_id: 1>,
#     #<Computadora id: 2, modelo: 'desktop 24"', empleado_id: 1>,
#     #<Computadora id: 3, modelo: 'netbook', empleado_id: 1>]
```

La nueva computadora aparece en la lista de computadoras asociadas, con el campo `empleado_id: 1`, lo que significa que está asociado al empleado que creamos previamente, y con el `id: 3`, lo que significa que la computadora no sólo fue asignada, sino que al momento de realizar la asignación, Rails la guardó automáticamente.

Este comportamiento puede ser el esperado en muchas de las ocasiones, pero en otras no. A veces preferimos asociar un modelo nuevo a uno existente sin que el nuevo modelo se guarde automáticamente, por ejemplo, cuando hay validaciones que todavía no se satisfacen, o cuando tenemos que hacer un procesamiento posterior sobre el modelo a agregar antes de que éste sea guardado. Si ese es nuestro caso, ¿qué podemos hacer?

## `create`, `build` y `new` a través de la clase proxy

Nuestra relación `computadoras` dentro de un modelo `empleado` es una clase proxy que incluye métodos que nos permiten algunas operaciones. Una pequeña aclaración sobre la clase de la asociación: en Rails 3, si ejecutábamos `empleado.computadoras.class`, obteníamos como resultado `Array`, pero era en realidad información falsa, porque `empleado.computadoras` es en realidad una clase _proxy_. En Rails 4, la información es más adecuada: `ActiveRecord::Associations::CollectionProxy::ActiveRecord_Associations_CollectionProxy_Computadora`.

Esta clase nos provee de un par de métodos más para asociar dos modelos en una relación uno a muchos: los métodos `build` y `create`. El método `build` construye y asigna el modelo pero no lo guarda (debemos llamar a `save` manualmente si queremos guardarlo). El método `create`, en cambio, guarda automáticamente el modelo asociado.

```ruby
computadora_sin_guardar = empleado.computadoras.build modelo: 'server'
computadora_sin_guardar.new_record?
# => true
computadora_sin_guardar.empleado_id
# => 1
empleado.computadoras
# => [#<Computadora id: 1, modelo: 'laptop 15"', empleado_id: 1>,
#     #<Computadora id: 2, modelo: 'desktop 24', empleado_id: 1>,
#     #<Computadora id: 3, modelo: "netbook", empleado_id: 1>,
#     #<Computadora id: nil, modelo: "server", empleado_id: 1>]
empleado.save
Empleado.find(1).computadoras
# => [#<Computadora id: 1, modelo: 'laptop 15"', empleado_id: 1>,
#     #<Computadora id: 2, modelo: 'desktop 24', empleado_id: 1>,
#     #<Computadora id: 3, modelo: "netbook", empleado_id: 1>,
#     #<Computadora id: 4, modelo: "server", empleado_id: 1>]
```

Como la asociación no se guarda automáticamente, hay que hacerlo de manera manual llamando al método `save`. Esto se puede hacer ya sea desde el objecto `empleado` (`empleado.save`) o desde el objeto `computadora_sin_guardar` (`computadora_sin_guardar.save`). En ambos casos, se guarda también la relación.

Existe también el método `new`, que es simplemente un alias para el `build`.

Como regla general, para las asociaciones uno a muchos, podemos decir que si la asignación se hace del modelo con `belongs_to` al modelo con `has_many`, la asociación se guarda automáticamente, mientras que si se hace a la inversa, la asociación no será guardada mientras no se guarde alguno de los dos modelos.

# Asociación muchos a muchos

Las asociaciones muchos a muchos se guardan automáticamente, sin importar la dirección de la asignación. Si el modelo que se asigna no está guardado y el receptor sí, el nuevo modelo se guarda automáticamente. Este comportamiento lo podemos ver en el siguiente código.

```ruby
empleado.proyectos << proyecto
# => [#<Proyecto id: 1, nombre: "Proyecto Importante">]
proyecto.empleados << Empleado.new(nombre_completo: 'Guadalupe Martínez')
Proyecto.find(1).empleados
# => [#<Empleado id: 1, nombre_completo: "Juan Pérez">,
#     #<Empleado id: 2, nombre_completo: "Guadalupe Martínez">]
```

De nuevo, si queremos asignar un proyecto a un empleado (o viceversa) sin que la asignación provoque que el modelo se guarde automáticamente, podemos usar los métodos `build` o `new` de la clase _proxy_ de la asociación, y salvar el modelo posteriormente. Por ejemplo:

```ruby
empleado_sin_guardar = proyecto.empleados.build nombre_completo: 'Carlos López'
proyecto.empleados
# => [#<Empleado id: 1, nombre_completo: "Juan Pérez">,
#     #<Empleado id: 2, nombre_completo: "Guadalupe Martínez">,
#     #<Empleado id: nil, nombre_completo: "Carlos López">]
proyecto.save
Proyecto.find(1).empleados
# => [#<Empleado id: 1, nombre_completo: "Juan Pérez">,
#     #<Empleado id: 2, nombre_completo: "Guadalupe Martínez">,
#     #<Empleado id: 3, nombre_completo: "Carlos López">]
```

Como la asociación no guarda automáticamente, hay que hacerlo de manera manual llamando al método `save`, similar a como se hace en las asociaciones uno a muchos. La diferencia es que en este caso se tiene que llamar al método `save` desde el objeto que recibe la asignación, en este caso, `proyecto`. Si se llama a `save` desde el modelo nuevo (`empleado_sin_guardar.save`) sólo se guarda este objeto y no la relación.

También se puede usar el método `build` desde la otra asociación, por ejemplo: `empleado.proyectos.build(nombre: 'secreto')`.

Existe, como cabe suponer, un método `create` en ambas relaciones que construye y guarda al mismo tiempo el modelo asociado.

# Asociación uno a uno

Conociendo el comportamiento de la asociación uno a muchos y muchos a muchos, la asociación uno a uno parece más sencilla. Reproduciremos los mismos escenarios de los casos anteriores. En este ocasión trabajaremos con los modelos `Empleado` y `Puesto`. Como ya tenemos creados nuestros modelos, simplemente los asociamos.

```ruby
puesto.empleado = empleado
puesto.empleado
# => #<Empleado id: 1, nombre_completo: "Juan Pérez"> 
empleado.puesto
# => nil
```

Cuando asignamos el modelo que tiene el `has_one` al modelo que tiene el `belongs_to`, la relación no se salva automáticamente. Veamos la asignación contraria:

```ruby
empleado.puesto = puesto
Puesto.find(1).empleado
# => #<Empleado id: 1, nombre_completo: "Juan Pérez">
Empleado.find(1).puesto
# => #<Puesto id: 1, empleado_id: 1, nombre: "Programador">
```

El comportamiento es el esperado, al hacer la asignación contraria, la asociación se guarda automáticamente. Probemos ahora cómo se comporta cuando se asocia un modelo que no ha sido salvado:

```ruby
empleado.puesto = Puesto.new(nombre: 'Programador Sr')
Empleado.find(1).puesto
# => #<Puesto id: 2, empleado_id: 1, nombre: "Programador Sr">
```

De nuevo, se mantiene el comportamiento visto en la relación uno a muchos, es decir, el nuevo modelo se guarda automáticamente al asignarse a un modelo existente.

## `build_`association y `create_`association

Si queremos asociar un nuevo modelo `Puesto` al modelo de `Empleado`, pero sin salvarlo, buscaríamos hacer algo como esto: `empleado.puesto.build(nombre: 'jefe')`, desafortunadamente eso no funciona, nos arroja un error que dice `NoMethodError: undefined method 'build' for nil:NilClass`. Esto es porque la asociación `puesto` no corresponde a una clase _proxy_ como en la relación uno a muchos, sino que es ya propiamente el modelo `puesto` que en este caso es `nil` (o puede ser un objeto `Puesto` en caso de que ya tenga un puesto asociado).

Afortunadamente, gracias a un poco de metaprogramación, Rails nos ofrece un método que hace lo que necesitamos. Por cada asociación uno a uno que tengamos, se genera un método. El nombre del método varía según el nombre de la asociación. En nuestro caso, el método se llama `build_puesto`. Usaremos este método para construir nuestro modelo.

```ruby
puesto_sin_gurdar = empleado.build_puesto nombre: 'Jefe'
empleado.puesto
# => #<Puesto id: nil, empleado_id: 1, nombre: "Jefe">
```

Aquí, al igual que en las relaciones uno a muchos, podemos salvar cualquiera de los dos modelos (`empleado` o `puesto_sin_guardar`) y en ambos casos, se guardará correctamente la asociación.

Así, podemos construir y asociar un modelo sin que sea guardado automáticamente.

De manera similar, existe un método que empieza por `create_` (`create_puesto` en nuestro caso) que realiza la asociación y la guarda automáticamente, como la asignación directa. Podemos usar este método para crear y asociar automáticamente un empleado a un puesto, por ejemplo:

```ruby
puesto_nuevo = Puesto.create 'nuevo'
puesto_nuevo.create_empleado(nombre_completo: 'Mario Guerrero')
```

# Conclusiones

Las asociaciones en Rails son una parte fundamental en el _framework_ y conocer su funcionamiento resulta muy útil en varios escenarios, particularmente en aquellos en los que el comportamiento, aunque consistente, no siempre resulta intuitivo.
Trataré de resumir los aspectos importantes que vimos:
* Las asociaciones uno a uno y uno a muchos tienen tienen un modelo que especifica el `belongs_to` y otro con `has_one` o `has_many` respectivamente. Cuando la asignación se hace del modelo con `belongs_to` al modelo con `has_*`, la asignación guarda automáticamente la relación y el modelo asignado en caso de que no exista en la base de datos.
* Si la asignación se hace en el otro sentido, la relación no se guarda automáticamente.
* Las asociaciones muchos a muchos salvan automáticamente la relación y el modelo, sin importar en qué sentido se haga la asignación.
* Para las asociaciones uno a muchos y muchos a muchos, existen los métodos `build` y `new` en la asociación (por medio de una clase _proxy_ ), que nos permiten construir modelos asociados sin salvarlos.
* Para las asociaciones uno a uno, el método existe directamente en el modelo (cualquiera de los dos) y se llama `build_#{nombre_de_la_asociacion}`.
