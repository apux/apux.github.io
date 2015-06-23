---
title: Formularios anidados
date: 2014-03-11
tags: [Rails, nested_attributes, formularios]
---

En ocasiones, tenemos modelos asociados que necesitamos manipular en un único
formulario en lugar de tener un formulario por cada uno de ellos. La gema
`nested_form` nos permite crear formularios complejos cuando trabajamos con
modelos anidados, su uso es realmente sencillo y su
[documentación](https://github.com/ryanb/nested_form) cubre los aspectos más
comunes. Sin embargo, existen algunos escenarios en los que debemos de tener
algunas consideraciones adicionales para que todo funcione correctamente,
especialmente con Rails 4.

En este tutorial, veremos cómo utilizar esta gema cuando incluimos relaciones
_uno a muchos_ y _muchos a muchos_ en nuestros modelos.

## Creación del proyecto

Para este tutorial, trabajaremos con un proyecto en Rails 4.1. Al momento de 
escribir este tutorial, es una versión que todavía está en RC, por lo que se se 
instala de la siguiente manera:

    gem install rails --pre

Para empezar, agregamos la gema `nested_form` a nuestro proyecto.

    echo "gem 'nested_form'" >> Gemfile

Y ejecutamos

    bundle install

Ahora debemos agregar el _javascript_ de la gema al _asset pipeline_. En el
archivo `application.js`, ingresamos la siguiente línea:

    //= require jquery_nested_form

Nota: La versión de `nested_form` utilizada para este post es `0.3.2`.

## Creación de modelos

Trabajaremos con tres modelos relacionados entre sí. El modelo `Proyecto` tiene
muchas `Tareas` (relación uno a muchos), y las `Tareas` están asociadas a muchos
`Empleados` (relación muchos a muchos).

Por comodidad, usaremos `scaffold` para generar los recursos de `Proyecto` y
`Empleado`, mientras que `Tarea` no tendrá ni vista ni controlador propio, sino 
que se creará a través del formulario de `Proyecto`, que es donde utilizaremos 
la gema `nested_form` que acabamos de instalar.

    rails generate scaffold proyecto nombre fecha_entrega:date
    rails generate scaffold empleado nombre_completo
    rails generate model tarea nombre prioridad:integer proyecto:references

Para la relación muchos a muchos entre `tareas` y `empleados` necesitamos
una migración adicional para crear la tabla intermedia entre estos dos
modelos. En Rails 4, podemos hacerlo de la siguiente manera:

    rails generate migration create_join_table_empleados_tareas empleado tarea

Este comando nos genera una migración que crea la tabla intermedia para nuestra
relación muchos a muchos:

```ruby
class CreateJoinTableEmpleadosTareas < ActiveRecord::Migration
  def change
    create_join_table :empleados, :tareas do |t|
      # t.index [:empleado_id, :tarea_id]
      # t.index [:tarea_id, :empleado_id]
    end
  end
end
```

Ahora creamos la base de datos y generamos las tablas por medio de las
migraciones. En este ejemplo, utilizaremos SQLite como manejador de base de 
datos, por lo que no es necesario configurar nada más. Si prefieres utilizar 
MySql, Postgres o algún otro manejador, asegúrate de incluir su gema 
correspondiente en el archivo `Gemfile` y adaptar el archivo 
`config/database.yml`.

    rake db:create
    rake db:migrate

Modificamos los modelos correspondientes para que incluyan las relaciones que
hemos creado en la base de datos:

```ruby
# app/models/proyecto.rb
class Proyecto < ActiveRecord::Base
  has_many :tareas
end

# app/models/tarea.rb
class Tarea < ActiveRecord::Base
  belongs_to :proyecto
  has_and_belongs_to_many :empleados
end

# app/models/empleado.rb
class Empleado < ActiveRecord::Base
  has_and_belongs_to_many :tareas
end
```

## Datos de inicio

Agreamos empleados a la base de datos por medio del archivo `db/seeds.rb`.

```ruby
# db/seeds.rb
Empleado.create! [
  {nombre_completo: 'Juan Pérez'},
  {nombre_completo: 'Pedro López'},
  {nombre_completo: 'María Hernández'},
  {nombre_completo: 'Carlos Sánchez'},
]
```

Para almacenar esa información en la base de datos:

    rake db:seed

Por supuesto, también podemos agregar empleados manualmente desde nuestra
aplicación, ejecutando `rails server`, y accediendo a la ruta `empleados/new`.

## Formulario de proyectos

Ya con nuestros modelos creados, debemos trabajar con el formulario de
proyectos, donde podemos crear un proyecto que contenga muchas tareas que a su
vez estén asociadas con muchos empleados.

El formulario que nos creó el _scaffold_ es el siguiente:

`app/views/proyectos/_form.html.erb`

```erb
<%= form_for(@proyecto) do |f| %>
  <% if @proyecto.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@proyecto.errors.count, "error") %> prohibited this proyecto from being saved:</h2>

      <ul>
      <% @proyecto.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :nombre %><br>
    <%= f.text_field :nombre %>
  </div>
  <div class="field">
    <%= f.label :fecha_entrega %><br>
    <%= f.date_select :fecha_entrega %>
  </div>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

Para utilizar la gema, debemos utilizar el _helper_ `nested_form_for` en lugar de
`form_for` y agregar los campos de las tareas. Rails nos provee de un _helper_
para agregar campos a objetos asociados: `fields_for`. El formulario quedaría de
la siguiente manera:

`app/views/proyectos/_form.html.erb`

```erb
<%= nested_form_for(@proyecto) do |f| %>
  <% if @proyecto.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@proyecto.errors.count, "error") %> prohibited this proyecto from being saved:</h2>

      <ul>
      <% @proyecto.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :nombre, 'Nombre del proyecto' %><br>
    <%= f.text_field :nombre %>
  </div>
  <div class="field">
    <%= f.label :fecha_entrega %><br>
    <%= f.date_select :fecha_entrega %>
  </div>
  <fieldset id="tareas">
    <%= f.fields_for :tareas do |tareas_form| %>
      <div class="field">
        <%= tareas_form.label :nombre, 'Nombre de la tarea' %><br>
        <%= tareas_form.text_field :nombre %>
      </div>
      <div class="field">      
        <%= tareas_form.label :prioridad %><br>
        <%= tareas_form.text_field :prioridad %>
      </div>
      <%= tareas_form.link_to_remove "Eliminar esta tarea" %>
    <% end %>
    <p><%= f.link_to_add "Agregar una tarea", :tareas %></p>
  </fieldset>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

### Uso de `accepts_nested_attributes_for`

Si en estos momentos entramos a nuestra aplicación e intentamos crear un
proyecto, obtendremos el siguiente error:

    Invalid association. Make sure that accepts_nested_attributes_for is used for :tareas association.

Esto es porque nuestro formulario de `Proyecto` incluye ya los atributos
anidados de `Tareas`, pero no hemos configurado nuestros modelos para que
permita recibir estos campos.

En nuestro modelo `Proyecto` agregamos el `accepts_nested_attributes_for`:

```ruby
# app/models/proyecto.rb
class Proyecto < ActiveRecord::Base
  has_many :tareas
  accepts_nested_attributes_for :tareas, allow_destroy: true
end
```

De esta manera, `Proyecto` puede recibir y procesar los atributos de tareas
mediante la llave `tareas_attributes`. Ejemplo:

```ruby
Proyecto.create nombre: 'Mi proyecto',
             fecha_entrega:  1.month.from_now,
             tareas_attributes: [
                                 {nombre: 'Tarea 1', prioridad: 5},
                                 {nombre: 'Tarea 2', prioridad: 3}
             ]
```

Con esto, le indicamos a ActiveRecord que cree un proyecto con dos tareas.

La opción `allow_destroy` nos permitirá eliminar alguna tarea de la lista de
tareas agregando la bandera `:_destroy`. Ejemplo:

```ruby
Proyecto.create nombre: 'Mi proyecto',
             fecha_entrega:  1.month.from_now,
             tareas_attributes: [
                                 {nombre: 'Tarea 1', prioridad: 5},
                                 {nombre: 'Tarea 2', prioridad: 3, _destroy: true }
             ]
```

Con esto, le indicamos a ActiveRecord que cree un proyecto con una sola tarea
(`Tarea 1`), ya que `Tarea 2` está marcada para ser eliminada y por lo tanto no
se crea. Si la tarea ya existe previamente, al incluir la opción `_destroy:
true` se borrará también de la base de datos, pero para eso habrá que
especificar el `id`, como veremos más adelante.

Si tratamos de entrar a nuestro formulario de proyectos (/proyectos/new) veremos
que el formulario se muestra correctamente y podemos agregar o eliminar tareas
visualmente de manera dinámica.

## Agregar tareas por omisión

Si queremos que nuestro proyecto se muestre inicialmente con algunas tareas
asociadas, podemos agregar algunas desde el controlador. Ejemplo:

```ruby
#app/controllers/proyectos_controller.rb
def new
  @proyecto = Proyecto.new
  2.times { @proyecto.tareas.build }
end
```

## Parciales

En nuestro ejemplo, el modelo `Proyecto` tiene dos campos, y el modelo `Tarea`
otros dos, pero en un caso real, ambos tendrían muchos más campos y mostrarlos
todos dejaría un tanto sucio el código de nuestro formulario. En esos casos, es
mejor utilizar parciales.

`nested_form` permite el uso de parciales de una manera elegante. Necesitamos
crear una parcial con el nombre en singular de nuestro modelo más el sufijo
`_fields`. En la nueva parcial, la variable del formulario anidado
(`tareas_form` en nuestro caso), se pasa simplemente como `f`.  Los archivos
quedarían de la siguiente manera:

El formulario principal de proyectos de proyectos.

`app/views/proyectos/_form.html.erb`

```erb
<%= nested_form_for(@proyecto) do |f| %>
  <% if @proyecto.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@proyecto.errors.count, "error") %> prohibited this proyecto from being saved:</h2>

      <ul>
      <% @proyecto.errors.full_messages.each do |msg| %>
        <li><%= msg %></li>
      <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :nombre, 'Nombre del proyecto' %><br>
    <%= f.text_field :nombre %>
  </div>
  <div class="field">
    <%= f.label :fecha_entrega %><br>
    <%= f.date_select :fecha_entrega %>
  </div>
  <fieldset id="tareas">
    <%= f.fields_for :tareas %>
    <p><%= f.link_to_add "Agregar una tarea", :tareas %></p>
  </fieldset>
  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

La nueva parcial:

`app/views/proyectos/_tarea_fields.html.erb`

```erb
<div class="field">
  <%= f.label :nombre, 'Nombre de la tarea' %><br>
  <%= f.text_field :nombre %>
</div>
<div class="field">
  <%= f.label :prioridad %><br>
  <%= f.text_field :prioridad %>
</div>
<%= f.link_to_remove "Eliminar esta tarea" %>
```

Como vemos, queda más limpio y mejor organizado.

## Show

Antes de crear un proyecto por medio del formulario, modificaremos el `show`
para mostrar no sólo los datos del proyecto sino también los de las tareas
asociadas. Así podremos saber si las tareas se crearon correctamente.

`app/views/proyectos/show.html.erb`

```erb
<p id="notice"><%= notice %></p>

<p>
  <strong>Nombre:</strong>
  <%= @proyecto.nombre %>
</p>

<p>
  <strong>Fecha entrega:</strong>
  <%= @proyecto.fecha_entrega %>
</p>

<p>
  <strong>Tareas:</strong>
  <p>
    <% @proyecto.tareas.each do |tarea| %>
      Nombre: <%= tarea.nombre %><br/>
      Prioridad: <%= tarea.prioridad %>
      <hr/>
    <% end %>
  </p>
</p>

<%= link_to 'Edit', edit_proyecto_path(@proyecto) %> |
<%= link_to 'Back', proyectos_path %>
```

## Strong parameters

Ahora sí, es momento de crear nuestro primer proyecto por medio del formulario.
Ingresamos los datos tanto del proyecto como de las tareas, y damos clic en el
botón de _submit_.

La página de `show` nos muestra el nombre del proyecto y la fecha de entrega
pero la lista de tareas está vacía, lo que significa que algo falló en el
proceso.

El problema se encuentra en el controlador. Rails 4 incluye `strong_parameters`,
que valida que se reciban sólo atributos autorizados para un formulario. Por
omisión, al ejecutar el _scaffold_, sólo se autorizan los campos que se incluyen
en el modelo `Proyecto`, por lo que tenemos que agregar a mano los campos del
modelo `Tarea`, incluyendo el atributo `_destroy`, para que una tarea pueda ser
eliminada de la lista. Estos atributos se agregan a la lista que recibe el
método `permit` como un _hash_, donde la llave es `tareas_attributes` y el valor
es un arreglo con los nombres de los campos.

```ruby
# app/controllers/proyectos_controller.rb

# [...]

# Never trust parameters from the scary internet, only allow the white list through.
def proyecto_params
  params.require(:proyecto).permit(
    :nombre, :fecha_entrega, tareas_attributes: [
      :nombre, :prioridad, :_destroy
    ]
  )
end
```

Y ahora sí, al crear un proyecto con tareas, éstas se crearán correctamente.

## Update

Ahora bien, si queremos modificar un proyecto que ya hayamos ingresado, podemos
hacerlo en el _path_ correspondiente, por ejemplo, /proyectos/1/edit.

Sin embargo, aquí notamos un comportamiento extraño. En lugar de actualizar las
tareas, crea siempre tareas nuevas. Es decir, mantiene las tareas creadas
anteriormente y trata las tareas que estamos modificando como si las estuvieras
agregando.  Vemos también otro error: si queremos eliminar una tarea, ésta no se
elimina, sino que sigue ahí.

Este comportamiento sucede porque para realizar la modificación, ActiveRecord
espera recibir como parámetro, el id del modelo a modificar. Si el id
corresponde a un registro de la base de datos, entonces lo modifica. Ejemplo:

Imaginemos un proyecto con dos tareas, las tareas tienen los ids 1 y 2
respectivamente.

```ruby
proyecto = Proyecto.first
proyecto.update nombre: 'Mi nuevo proyecto',
             fecha_entrega:  1.week.from_now,
             tareas_attributes: [
                                 {nombre: 'Tarea nueva', prioridad: 1 },
                                 {id: 1, nombre: 'Tarea modificada', prioridad: 4 },
                                 {id: 2, nombre: 'Tarea 2', prioridad: 2, _destroy: true }
             ]
```

Este fragmento de código actualiza los datos del proyecto, incluyendo sus
tareas. La primera tarea no lleva id, por lo que simplemente se crea y
se asocia al proyecto. Las siguientes dos tareas sí llevan id, por lo que se
entiende que ya existen previamente y están asociadas a ese proyecto; esas
tareas se actualizan. En este caso, la tarea con id 1 sólo actualiza sus datos,
mientras que la tarea con id 2 se elimina porque incluye la bandera `_destroy`
como `true`.

### Strong parameters

Conociendo la teoría, la solución es sencilla. Basta con agregar a la lista de
parámetros aceptados, el id de las tareas. Para esto, modificamos de nuevo la
lista de nuestros _strong parameters_.

```ruby
# app/controllers/proyectos_controller.rb

# [...]

# Never trust parameters from the scary internet, only allow the white list through.
def proyecto_params
  params.require(:proyecto).permit(
    :nombre, :fecha_entrega, tareas_attributes: [
      :id, :nombre, :prioridad, :_destroy
    ]
  )
end
```

Y listo. Con eso podemos modificar y eliminar tareas correctamente desde nuestro
formulario de proyectos.

## Muchos a muchos

Ya hemos asociado correctamente proyectos y tareas desde un solo formulario,
pero aún nos falta asociar tareas y empleados, que es una relación muchos a
muchos.

Lo primero que modificaremos será nuestro `show` para incluir información sobre
los empleados.

`app/views/proyectos/show.html.erb`

```erb
<p id="notice"><%= notice %></p>

<p>
  <strong>Nombre:</strong>
  <%= @proyecto.nombre %>
</p>

<p>
  <strong>Fecha entrega:</strong>
  <%= @proyecto.fecha_entrega %>
</p>

<p>
  <strong>Tareas:</strong>
  <p>
    <% @proyecto.tareas.each do |tarea| %>
      Nombre: <%= tarea.nombre %><br/>
      Prioridad: <%= tarea.prioridad %><br/>
      Empleados: <%= tarea.empleados.map(&:nombre_completo).to_sentence %>
      <hr/>
    <% end %>
  </p>
</p>

<%= link_to 'Edit', edit_proyecto_path(@proyecto) %> |
<%= link_to 'Back', proyectos_path %>
```
Ahora procedemos a modifcar nuestro formulario para incluir empleados.

Como primer recurso, recurro a este
[railscast](http://railscasts.com/episodes/17-habtm-checkboxes), donde se
muestra una forma de hacerlo utilizando `chec_kbox_tag`.

En un formulario normal, la asociación podría hacerce de la siguiente manera:

```erb
<% Empleado.all.each do |empleado| %>
  <%= check_box_tag 'tarea[empleado_ids][]',
                    empleado.id,
                    @tarea.empleado_ids.include?(empleado.id) %>
  <%= empleado.nombre_completo %><br/>
<% end %>
```

Sin embargo, eso da por sentado que la tarea es el formulario del primer nivel,
lo que es falso para nuestro caso. Nosotros necesitaríamos de una estructura más
compleja que incluyera el proyecto y la tarea (mediante `tareas_attributes`).

Si analizamos el _request_ que se hace cuando envía nuestro formulario, veremos
que los parámetros llevan la siguiente estructura:

```ruby
Started POST "/proyectos" for 127.0.0.1 at 2013-08-12 12:41:01 -0500
Processing by ProyectosController#create as HTML
  Parameters: {"utf8"=>"✓",
               "authenticity_token"=>"RtyGum+m6wIvaoOAvxcQnIlvMEPGStBmFvaTL+t+paQ=",
               "proyecto"=>{
                             "nombre"=>"Mi proyecto",
                             "fecha_entrega(1i)"=>"2013",
                             "fecha_entrega(2i)"=>"9",
                             "fecha_entrega(3i)"=>"22",
                             "tareas_attributes"=>{
                                                    "0"=>{
                                                           "nombre"=>"Tarea 1",
                                                           "prioridad"=>"1",
                                                           "_destroy"=>"false"},
                                                    "1"=>{
                                                           "nombre"=>"Tarea 2",
                                                           "prioridad"=>"",
                                                           "_destroy"=>"false"}
                                                  }
                           },
               "commit"=>"Create Proyecto"}
```

Como vemos, el formulario envía `tareas_attributes` no como un arreglo sino como
un _hash_, donde la llave es un índice para cada tarea. Si quisiéramos incluir los
_checkboxes_ de empleados, tendríamos que indicar el número de cada tarea. Otro
aspecto a tomar en cuenta es que no tenemos `@tarea`, sino `@proyecto`, por lo
que tendríamos que tomarlo del objeto del formulario. Algo como esto:

```erb
<%= check_box_tag "proyecto[tareas_attributes][#{id_tarea}][empleado_ids][]",
                  empleado.id,
                  f.object.empleado_ids.include?(empleado.id) %>
```

Para el caso de nuestro ejemplo, es posible llevar el control del id de la tarea
por medio de alguna variable generada en `_form` y pasarla a la parcial
`_tareas_fields`, pero pensando en varios niveles de formularios anidados, sería
muy complicado llevar el registro de cada uno de los ids de los modelos e irlos
pasando entre parciales. Por ejemplo, si la tarea no estuviera relacionada
directamente con empleados, sino con asignaciones, y las asignaciones con
empleados, tendríamos un _tag_ parecido a esto:

```erb
<%= check_box_tag(
    "proyecto[tareas_attributes][#{tarea_index}][asignaciones_attributes][#{asignacion_index}][empleado_ids][]",
    empleado.id,
    f.object.empleado_ids.include?(empleado.id)) %>
```

Lo que poco a poco se vuelve insostenible.

Afortunadamente, Rails 4 incluye un _helper_ para asociaciones muchos a muchos a
través de _checkboxes_, y eso nos facilita bastante las cosas. El _helper_ es
`collection_check_boxes`. Nuestra parcial de tareas quedaría de la siguiente
manera:

`app/views/proyectos/_tarea_fields.html.erb`

```erb
<div class="field">
  <%= f.label :nombre, 'Nombre de la tarea' %><br>
  <%= f.text_field :nombre %>
</div>
<div class="field">
  <%= f.label :prioridad %><br>
  <%= f.text_field :prioridad %>
</div>
<div class="field">
  <%= f.collection_check_boxes :empleado_ids, Empleado.all, :id, :nombre_completo %>
</div>
<%= f.link_to_remove "Eliminar esta tarea" %>
```

Haciendo uso de este _helper_, no tenemos que preocuparnos por nada más, Rails
se encarga de llevar los índices por nosotros.

### Strong parameters

Si probamos nuestro formulario ahora, funciona correctamente, los parámetros que
genera son los correctos, pero aún no guarda las asociaciones con empleados. De
nuevo, esto tiene que ver con _strong parameters_. Debemos indicar que acepte
los campos correspondientes a empleado_ids.

Aquí, hay que observar un pequeño _truco_ para hacer que `strong parameters`
acepte la lista de ids que le mandamos desde nuestro formulario. Si agregamos
simplemente el campo `:empleado_ids`, _strong parameters_ lo filtrará como un
campo cuando en realidad necesitamos que lo considere un arreglo. En este caso
debemos indicarlo como un _hash_, donde la llave sea `:empleado_ids` y el valor
un arreglo vacío.

```ruby
# app/controllers/proyectos_controller.rb

# [...]

# Never trust parameters from the scary internet, only allow the white list through.
def proyecto_params
  params.require(:proyecto).permit(
    :nombre, :fecha_entrega, tareas_attributes: [
      :id, :nombre, :prioridad, :_destroy, empleado_ids: []
    ]
  )
end
```

Y ahora sí, nuestro formulario funciona correctamente asociando proyectos,
tareas y empleados.

# Conclusiones

Rails permite trabajar con formularios anidados de manera relativamente fácil.
La gema `nested_form` hace que este trabajo sea todavía más sencillo, sin
embargo, hay que conocer el funcionamiento de otros módulos, como _strong
parameters_ y *accepts_nested_attributes_for* para que todo funcione
correctamente.

Este tutorial cubrió los aspectos básicos para el trabajo con formularios
anidados, pero todavía quedan algunos detalles que cubrir. En el siguiente post
explicaré cómo utilizar la gema cuando tenemos una estructura de modelos más
compleja. En particular, cuando tenemos modelos dentro de un _namespace_ y
recursos anidados ( _nested resources_ ) en nuestras rutas.

# Recursos

Para quien esté interesado en conocer el funcionamiento interno de la gema,
puede revisar este
[post](http://blog.madebydna.com/all/code/2010/10/07/dynamic-nested-froms-with-the-nested-form-gem.html).
Además, estos _railscasts_ son de mucha utilidad: [Nested Model Form Part
1](http://railscasts.com/episodes/196-nested-model-form-part-1) y [Nested Model
Form Part 2](http://railscasts.com/episodes/197-nested-model-form-part-2).
También hay una versión actualizada (de paga): [Nested Model Form
(revised)](http://railscasts.com/episodes/196-nested-model-form-revised).

Un [post](http://blog.sensible.io/2013/08/17/strong-parameters-by-example.html)
muy completo sobre `Strong Parameters`.

El railscast de [habtm con
checkboxes](http://railscasts.com/episodes/17-habtm-checkboxes) y su [versión de
pago](http://railscasts.com/episodes/17-habtm-checkboxes-revised).

La documentación de [collection check
boxes](http://edgeapi.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_check_boxes).

También creé un [proyecto de ejemplo en github](https://github.com/apux/formularios_anidados)
para que sea fácil revisar el código.
