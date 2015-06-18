---
title: "Merge (Español)"
date: 2013-09-30
tags: [Ruby, Rails, ActiveRecord]
---

Hay una funcionalidad de Rails que yo utilizo bastante pero tengo la impresión
de que no es tan difundida como debería. Estoy hablando del metodo `merge` de
`ActiveRecord`. Posiblemente la razón por la que no es muy utilizado es que
existe un metodo con el mismo nombre en la clase `Hash` y eso provoca cierta
confusion.

## El `merge` de `Hash`

El metodo `merge` de `Hash` mezcla dos _hashes_ y devuelve el resultado en un
tercer _hash_. Ejemplo:

```ruby
h1 = {a: 'cat', b: 'dog'}
h2 = {c: 'bird'}
h3 = h1.merge(h2)
# => {:a=>"cat", :b=>"dog", :c=>"bird"}
```

Este método está disponible en el
[Core de Ruby](http://www.ruby-doc.org/core-2.0.0/Hash.html#method-i-merge).

## El `merge` de `ActiveRecord`

El metodo `merge` de `ActiveRecord`, en cambio, sirve para construir consultas
que involucren dos o más modelos que necesiten de trabajar con _scopes_ ya
definidos.

Por ejemplo: imaginemos los modelos `Client` y `Product`, con _scopes_ definidos
la siguiente manera:

```ruby
# app/models/client.rb
class Client < ActiveRecord::Base
  has_many :products

  scope :recently_activated, -> { where 'activated_at > ?', 1.month.ago }
end

# app/models/product.rb
class Product < ActiveRecord::Base
  belongs_to :client

  scope :expensive,  -> { where 'products.price > 1000' }
  scope :small,      -> { where 'products.size <= 99' }
  scope :medium,     -> { where 'products.size > 99 AND products.size < 999' }
  scope :big,        -> { where 'products.size >= 999' }
  scope :available,  -> { where available: true }
end
```

Ahora, si quieremos conocer todos los clientes recientemente activados que hayan
comprado productos dispoibles, de talla mediana, y caros, se podría construir la
consulta de esta manera.

```ruby
Client.recently_activated.joins(:products).
  where('products.price > 1000 AND products.size > 99 AND products.size < 999 AND products.available = ?', true)
```

O, usando el método `merge`:

```ruby
Client.recently_activated.joins(:products).merge(Product.expensive.medium.available)
```

El último fragmento de código no es sólo más fácil de construir y más fácil de
leer, además sigue mejor el concepto DRY. Si, por alguna razón, la definición de
'medium' cambia, sólo se tiene que cambiar en un único lugar, todas las
consultas se mantienen intactas.

Además, es posible usar `merge` dentro de un _scope_. Ejemplo:


```ruby
# app/models/client.rb
class Client < ActiveRecord::Base
  has_many :products

  scope :recently_activated, -> { where 'activated_at > ?', 1.month.ago }
  scope :with_expensive_products, -> { joins(:products).merge(Product.expensive) }
end
```

Bastante útil, ¿cierto?
