---
title: Minitest v4 and v5
date: 2014-07-14
tags: [Minitest]
---

The current version of Minitest is 5, this is what we get if you do `gem install minitest`. However, chances are that our version is 4, because it is the version included in Ruby. I mean, if we didn't install Minitest explicitly, then we are using the one packet with the interpreter.

For us, the main difference is the class we need to inherit.

In version 4, we inherit from `MiniTest::Unit::TestCase`. Example:

```ruby
require 'minitest/autorun'

class FizzBuzzTest < MiniTest::Unit::TestCase
  def test_1_is_1
    assert_equal "1", FizzBuzz.new.convert(1)
  end
end
```

Note that the Minitest module is `MiniTest` with a capital T.

In version 5, we inherit from `Minitest::Test`. Example:

```ruby
require 'minitest/autorun'

class FizzBuzzTest < Minitest::Test
  def test_1_is_1
    assert_equal "1", FizzBuzz.new.convert(1)
  end
end
```

Now, the module is `Minitest` and the hierarchy changed.

If we prefer the spec syntax, we don't have to change anything.

```ruby
require 'minitest/autorun'

describe FizzBuzz do
  it "is '1' when receives 1" do
    assert_equal "1", FizzBuzz.new.convert(1)
  end
end
```

This code works well in both versions.
