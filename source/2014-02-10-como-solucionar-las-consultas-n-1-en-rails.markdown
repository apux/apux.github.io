---
title: Cómo solucionar consultas N+1 en Rails
date: 2014-02-10
tags: [Rails, ActiveRecord]
---

Uno de los problemas de desempeño más comunes en Rails está relacionado con consultas N+1. Afortunadamente, nuestro framework incluye una solución simple para este problema que debemos conocer para evitarnos algunos dolores de cabeza.

Para aquellos que no lo sepan, el problema de consultas N+1 generamente aparece cuando tenemos dos o más modelos asociados, por ejemplo Empleado (`Employee`) que tiene una Computadora (`Computer`), y se quiere mostrar información sobre los empleados así como de sus computadoras. Esto provoca que rails ejecute una consulta para cargar a los empleados, y una consulta por cada computadora. Si la compañia tiene 10 empleados, se ejecutan 11 consultas.

Veamos un ejemplo:

```ruby
#app/models/employee.rb
class Employee < ActiveRecord::Base
  has_one :computer
end

#app/models/computer.rb
class Computer < ActiveRecord::Base
  belongs_to :employee
end
```

Queremos listar todos los empleados con sus computadoras.

```erb
<p>
  <% employees = Employee.all %>
  <ul>
    <% employees.each do |employee| %>
      <li>
        <strong>Name:</strong>
        <%= employee.name %>
        <strong>Computer</strong>
        <%= employee.computer.brand %>
      </li>
    <% end %>
  </ul>
</p>
```

Si revisamos los logs, nos encontramos este tipo de salidas:

```
Employee Load (0.3ms)  SELECT "employees".* FROM "employees"
Computer Load (0.2ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 1]]
Computer Load (0.2ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 2]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 3]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 4]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 5]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 6]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 7]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 8]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 9]]
Computer Load (0.1ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" = ? LIMIT 1  [["employee_id", 10]]
```

Esto indica que tenemos un problema N+1. Podemos solucionar este problema usando _eager loading_. En rails, esto puede hacerse usando `includes` para indicar que la consulta debe cargar no sólo los datos del empleado sino también los datos de sus computadoras.

```ruby
<p>
  <% employees = Employee.includes(:computer).all %>
  <ul>
    <% employees.each do |employee| %>
      <li>
        <strong>Name:</strong>
        <%= employee.name %>
        <strong>Computer</strong>
        <%= employee.computer.brand %>
      </li>
    <% end %>
  </ul>
</p>
```

Si revisamos los logs ahora, podemos ver que Rails realizó sólo una consulta:

```
Computer Load (0.6ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

Se redujo el número de consultas a 1. Esto puede no parecer una gran mejora aquí, pero si la compañía tiene 1,000 empleados, se evita golpear 1,000 veces la base de datos cuando alguien visita esta página, y en ese caso, la diferencia sería importante.

Y sólo para aclarar, he usado este código como ejemplo por facilidad, pero no estoy recomendando realizar las consultas desde la vista. Hay que mover parte de este código a donde pertenece (controladores o modelos).
