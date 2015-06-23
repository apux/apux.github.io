---
title: Joins vs Includes (Español)
date: 2015-01-12
tags: [Rails, ActiveRecord]
---

A veces, `includes` y `joins` pueden resultar confusos porque hay escenarios donde parecen ser intercambiables, pero hay razones específicas para usar uno o el otro.

## `joins`

`joins` nos permite especificar una relación a incluirse en la consulta. La consulta resultante incluirá una cláusula `JOIN` (SQL).

Digamos que tenemos estos dos modelos, Empleado (`Employee`) que tiene una Computadora (`Computer`).

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

Si queremos conocer todos los empleados cuyo nombre sea Fernando y tengan una computadora Dell, podemos usar `joins`.

```ruby
Employee.
  joins(:computer).
  where(name: 'Fernando', computers: { brand: 'Dell' })
```

Esta consulta funciona bien y regresa todos los empleados que cunmplan con las condiciones.

El SQL será algo como esto:

```sql
SELECT "employees".*
 FROM "employees"
 INNER JOIN "computers" ON "computers"."employee_id" = "employees"."id"
 WHERE "employees"."name" = 'Fernando' AND "computers"."brand" = 'Dell'

```

## `includes`

Como explicamos en [un post previo](/2014/02/10/como-solucionar-las-consultas-n-1-en-rails.html), si queremos cargar todos los empleados y todas sus computadoras en una única consulta, podemos usar `includes`, 

```ruby
Employee.includes(:computer)
```

De esta manera, Rails golpea la base de datos sólo una vez, y carga todos los datos de empleados y computadoras. Puede que la confusión empiece aquí, porque al usar `includes` podemos también realizar algunas condicionales en la cláusula `where`. Por ejemplo:

```ruby
Employee.
  includes(:computer).
  where(name: 'Fernando', computers: { brand: 'Dell' })
```

Como podemos ver, es posible incluir condicionales que afecten ambas asociaciones, tal como lo hicimos con `joins`.

Sin embargo, el SQL resultante es diferente. Veamos:

```sql
SELECT "employees"."id" AS t0_r0,
 "employees"."name" AS t0_r1,
 "employees"."created_at" AS t0_r2,
 "employees"."updated_at" AS t0_r3,
 "computers"."id" AS t1_r0,
 "computers"."brand" AS t1_r1,
 "computers"."model" AS t1_r2,
 "computers"."employee_id" AS t1_r3,
 "computers"."created_at" AS t1_r4,
 "computers"."updated_at" AS t1_r5
 FROM "employees"
 LEFT OUTER JOIN "computers" ON "computers"."employee_id" = "employees"."id"
 WHERE "employees"."name" = 'Fernando' AND "computers"."brand" = 'Dell'
```

Como vemos, `includes` agrega todos los campos de la computadora, como se espera, mientras que `joins` no lo hace. Y esa pequeña diferencia es la que debemos tenee en cuenta.

Cuando sólo queremos filtrar los resultado de una consulta basados en un campo que pertenece a una relación secundaria, pero no vamos a usar datos de esa relación más adelante, debemos usar `joins`. De otra forma, cargaremos muchos datos que no vamos a usar.

Ejemplo:

```ruby
employees = Employee.
  joins(:computer).
  where(name: 'Fernando', computer: { brand: 'Dell' })
puts employees.map(&:name)
```

En la otra mano, si necesitamos usar datos de la relación secundaria, entonces `includes` es obligatorio, de otra manera, tendremos poblemas de consultas N+1.

Ejemplo:

```ruby
employees = Employee.
  joins(:computer).
  where(name: 'Fernando', computer: { brand: 'Dell' })
puts employees.map{|e| "#{e.full_name} has a Dell #{e.computer.model}" }
```

No es tan confuso después de todo, ¿cierto?
