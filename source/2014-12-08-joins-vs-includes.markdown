---
title: Joins vs Includes
date: 2014-12-08
tags: [Rails, ActiveRecord]
---

Sometimes, `joins` and `includes` can be confusing because there are scenarios where they seem to be interchangeable, but there are specific reasons to use one or the other.

## `joins`

`joins` allow us to specify a relation to be included in the query. The resulting query will include a `JOIN` clause (SQL).

Let's say we have these two models: `Employee` that has one `Computer`.

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

If we want to know all the employees whose name is 'Fernando' and has a Dell computer, we can use `joins`.

```ruby
Employee.
  joins(:computer).
  where(name: 'Fernando', computers: { brand: 'Dell' })
```

This query works well and returns all the employees who fulfill the conditions.

The SQL will be something like this:

```sql
SELECT "employees".*
 FROM "employees"
 INNER JOIN "computers" ON "computers"."employee_id" = "employees"."id"
 WHERE "employees"."name" = 'Fernando' AND "computers"."brand" = 'Dell'

```

## `includes`

As we explained in [a previous post](/2014/02/03/how-to-fix-n-1-queries-in-rails.html), if we want to load all the employees and all their computers in a single query, we can use `includes`, 

```ruby
Employee.includes(:computer)
```

This way, Rails hits the database just once, and load all the data from employees and computers. The confusion might start here, because using `includes` we can also perform some conditionals in the where clause. For, example:

```ruby
Employee.
  includes(:computer).
  where(name: 'Fernando', computers: { brand: 'Dell' })
```

As we can see, it is possible to include conditionals that affect both relations, just like we did with `joins`.

However, the resulting SQL is different. Let's see:

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

As we can see, `includes` adds all the fields from computer, as expected, while `joins` doesn't. And this little difference is what we must take into account.

When we just want to filter the result of a query based on a field that belongs to a secondary relation, but we aren't going to use data of this relation later, we must use `joins`. Otherwise, we will load a lot of data that we are not going to use.

Example:

```ruby
employees = Employee.
  joins(:computer).
  where(name: 'Fernando', computer: { brand: 'Dell' })
puts employees.map(&:name)
```

On the other hand, if we need to use data from the secondary relation, then `includes` is mandatory, otherwise, we will have N+1 query issues.

Example:

```ruby
employees = Employee.
  joins(:computer).
  where(name: 'Fernando', computer: { brand: 'Dell' })
puts employees.map{|e| "#{e.full_name} has a Dell #{e.computer.model}" }
```

It is not so confusing after all, right?
