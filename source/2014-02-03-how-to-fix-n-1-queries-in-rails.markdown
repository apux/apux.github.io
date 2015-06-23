---
title: How to fix N+1 queries in Rails
date: 2014-02-03
tags: [Rails, ActiveRecord]
---

One of the most commons performance issues in Rails is related to N+1 queries. Fortunately, our framework includes a simple solution for this problem that we must know in order to avoid some headaches.

For those who don't know, the issue of N+1 queries usually appears when we have two or more associated models, e.g. `Employee` which has one `Computer`, and we want to show information about all the employees as well as their computers. This provokes rails to execute one query to load the employees, and then, one query for each computer. If the company has 10 employees, 11 queries are executed.

Let's see an example:

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

We want to list all the employees with their computers.

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

If we check the logs, we can see these kinds of output:

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

That indicates we are having a N+1 issue. We can solve this problem using eager loading. In Rails, that can be done using `includes` to indicate the query must load not only the data from the employee but also the data from its computer.

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

If we check the logs now, we can see that Rails just performed one query:

```
Computer Load (0.6ms)  SELECT "computers".* FROM "computers" WHERE "computers"."employee_id" IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
```

It reduced the queries to only 1. It might not seem like a big improvement here, but if the company has 1,000 employees, you avoid hitting the database 1,000 times when a person visit this page, and in that case, the difference would be important.

And, just to clarify, I'm using this code as an example for ease, but I'm not recommending making the query from the view. We have to move part of this code where it belongs (models or controllers).
