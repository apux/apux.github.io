---
title: Minitest v4 y  v5
date: 2014-09-01
tags: [Minitest]
---

La versión actual de Minitset es 5, ésta es la que obtenemos al ejecutar `gem install minitest`. Sin embargo, es muy probable que nuestra versión sea la 4, porque es la versión que se incluye con Ruby. Es decir, si no hemos instalado Minitest de manera explícita, entonces estamos usando la versión empaquetada con el intérprete.

Para nosotros, la diferencia es la clase de la que heredamos.

En la versión 4, heredamos de `MiniTest::Unit::TestCase`. Ejemplo:

```ruby
require 'minitest/autorun'

class FizzBuzzTest < MiniTest::Unit::TestCase
  def test_1_is_1
    assert_equal "1", FizzBuzz.new.convert(1)
  end
end
```

Nota que el módulo Minitest es `MiniTest` con T mayúsculas.

En la versión 5, heredamos de `Minitest::Test`. Ejemplo:

```ruby
require 'minitest/autorun'

class FizzBuzzTest < Minitest::Test
  def test_1_is_1
    assert_equal "1", FizzBuzz.new.convert(1)
  end
end
```

Ahora, el módulo es `Minitest` y la herarquía cambió.

Si preferimos la sintaxis _spec_, no tenemos que cambiar nada.

```ruby
require 'minitest/autorun'

describe FizzBuzz do
  it "is '1' when receives 1" do
    assert_equal "1", FizzBuzz.new.convert(1)
  end
end
```

Este código funciona bien en ambas versiones.
