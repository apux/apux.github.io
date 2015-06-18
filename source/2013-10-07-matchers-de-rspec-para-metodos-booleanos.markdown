---
title: Matchers de RSpec para métodos booleanos
date: 2013-10-07
tags: [RSpec]
---

RSpec incluye una funcionalidad muy práctica al momento de probar métodos que
regresan falso o verdadero. Por convención, en Ruby estos métodos terminan con
un signo de interrogación que cierra: `?`.

Ejemplo:

```ruby
class Product
  def initialize(price)
    @price = price
  end

  def expensive?
    @price > 1000
  end
end
```

En la clase vemos que el método `expensive?` nos regresa verdadero si el precio
del producto es mayor a 1000 y falso en caso contrario. La prueba en RSpec
sería como la que sigue:

```ruby
describe Product do
  describe "#expensive?" do
    it 'is false when price is less than 1000' do
      Product.new(999).expensive?.should == false
    end

    it 'is false when price is equal to 1000' do
      Product.new(1000).expensive?.should == false
    end

    it 'is true when price is greater than 1000' do
      Product.new(1001).expensive?.should == true
    end
  end
end
```

Lo que se prueba son los casos límite para nuestro escenario: que los precios
de 999 y 1000 hagan que nuestro producto no sea _expensive_ y el precio de 1001
haga que sí lo sea.

La prueba es correcta y considero que los casos probados son los adecuados, sin
embargo, la sintaxis puede ser mejorada. RSpec integra algunos _matchers_ que
permiten una sintaxis más concisa: todos los métodos que terminan con `?`
pueden ser verificados con el matcher `be_*`, donde `*` es el nombre del método
sin el signo de interrogación. En nuestro caso: `be_expensive`. La prueba
completa puede quedar de la siguiente manera:

```ruby
describe Product do
  it 'is not expensive when price is less than 1000' do
    Product.new(999).should_not be_expensive
  end

  it 'is not expensive when price is equal to 1000' do
    Product.new(1000).should_not be_expensive
  end

  it 'is expensive when price is greater than 1000' do
    Product.new(1001).should be_expensive
  end
end
```

El cambio es menor, pero permite una mejor lectura y eso siempre se agradece.
