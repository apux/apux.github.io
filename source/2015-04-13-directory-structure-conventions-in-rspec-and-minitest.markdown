---
title: Directory structure conventions in RSpec and Minitest
date: 2015-04-13
tags: [Ruby, RSpec, Minitest]
---

When we create a new project in Ruby, the basic structure is putting the code in a directory named `lib`. We can also add a directory named `test` or `spec` for our testing files. That is not mandatory, but it is a convention that we should follow, not only because our directory structure will be more intuitive, but also because the testing frameworks sometimes assume it.
If we want to use RSpec, the directory name should be `spec`. If we prefer Minitest, we are allowed to name it the way we want, but it is common to named it `test` or `spec` as well.

## RSpec

Let's try a basic example with RSpec:

Directory structure:

```
fizz_buzz_rspec
├── lib
│   └── fizz_buzz.rb
└── spec
    └── fizz_buzz_spec.rb
```

`#spec/fizz_buzz_spec.rb`

```ruby
require 'fizz_buzz'

describe FizzBuzz do
  it 'returns "1" when receives 1' do
    expect(FizzBuzz.new.convert(1).to eq '1'
  end
end
```

`#lib/fizz_buzz.rb`

```ruby
class FizzBuzz
  def convert(n)
    '1'
  end
end
```

It is just the first test for a fizz buzz project.

Now, we can just run `rspec`:

```
[fizz_buzz_rspec]$ rspec
.

Finished in 0.00151 seconds (files took 0.1615 seconds to load)
1 example, 0 failures
```

I'm starting with a passing test because the result is less verbose, but you know that you should start with a failing test, right?, right? :)

As we can see, we didn't need any extra configuration. In our `fizz_buzz_spec` file, we required the `fizz_buzz` file and nothing more. Actually, we didn't need to require the `rspec` library and we didn't specify where our test files were. We just run the `rspec` command from the command line in our project directory and RSpec did the rest.

What would have happened if our spec directory was named differently? Let's say

```
fizz_buzz_rspec
├── lib
│   └── fizz_buzz.rb
└── my_testing_directory
    └── fizz_buzz_spec.rb
```

Well, in that case, if we run the `rspec` command it won't find any test file to execute.

```
[fizz_buzz_rspec]$ rspec
No examples found.


Finished in 0.00031 seconds (files took 0.07711 seconds to load)
0 examples, 0 failures
```

We must run the command indicating the directory.

```
[fizz_buzz_rspec]$ rspec my_testing_directory
.

Finished in 0.00151 seconds (files took 0.1615 seconds to load)
1 example, 0 failures
```

Also, it is important to follow the spec file name convention. RSpec expects all the spec file names to end with `_spec`. In our case, `fizz_buzz_spec.rb`. If we rename it, RSpec won't execute it. Let's see.

```
fizz_buzz_rspec
├── lib
│   └── fizz_buzz.rb
└── spec
    └── fizz_buzz_testing_file.rb
```

Let's try to execute:

```
[fizz_buzz_rspec]$ rspec
No examples found.


Finished in 0.00031 seconds (files took 0.07711 seconds to load)
0 examples, 0 failures
```

RSpec didn't load our file because it does not follow the name convention. If we name it back to `fizz_buzz_spec.rb`, it will work again.

If we see the testing file, we can see we didn't need any special configuration to load the `fizz_buzz.rb` besides requiring it. RSpec assumes that our code is in a `lib` directory and that is why it can load our file correctly. Let's try to rename the `lib` directory and see what happen.

```
fizz_buzz_rspec
├── main
│   └── fizz_buzz.rb
└── spec
    └── fizz_buzz_spec.rb
```

If we try to run `rspec`.

```
[fizz_buzz_rspec]$ rspec
[...]core_ext/kernel_require.rb:54:in `require': cannot load such file -- fizz_buzz (LoadError)
...
[...]spec/fizz_buzz_spec.rb:1:in `<top (required)>'
...
```

It is failing because of the `require` (the first line of the spec file). It doesn't find a `fizz_buzz.rb` file, which is assumed to be in the `lib` directory that we just renamed. So, we rename it back to `lib` and everything works again.

So, as we can see, it is better to stick to the conventions when we use RSpec.

### `spec_helper`

It is common to use a `spec_helper` file. When our project grows, we need to include more testing libraries, or add more tasks to do before or after testing. For example, we would like to include `shoulda`, or to clean the database. This tasks affect many testing files, so, it is better to have them in one file and include it in the other files. Our example does not need anything else, but I just want to show how it would be if it needed it.

Let's say we want to use the `rspec-given` gem. After installing it, we need to add this line in the `fizz_buzz_spec.rb` file:

`#spec/fizz_buzz_spec.rb`

```ruby
require 'rspec/given'
```

In our project, we only have one file, but if we had more, we would have to add that line to every single spec file. In order to avoid that, we can use a `spec_helper`.

First, we need to create a `spec_helper.rb` file in our `spec` directory.

```
fizz_buzz_rspec
├── lib
│   └── fizz_buzz.rb
└── spec
    ├── fizz_buzz_spec.rb
    └── spec_helper.rb
```

Now, we need to change the spec file:

`#spec/fizz_buzz_spec.rb`

```ruby
require 'spec_helper'
require 'fizz_buzz'

describe FizzBuzz do
  Given(:fizz_buzz) { FizzBuzz.new }
  When(:result) { fizz_buzz.convert(1) }
  Then { result == '1' }
end
```

If we run it now, it executes correctly. It doesn't matter where our test file is (for example, it could be under `spec/models/inbox/fizz_buzz_spec.rb`) it will be able to require the `spec_helper` with `require 'spec_helper'`.

Let's see how a Rails's `spec_helper` file looks, just as an example:

`spec/spec_helper.rb`

```ruby
ENV["RAILS_ENV"] ||= 'test'
require File.expand_path("../../config/environment", __FILE__)
require 'rspec/rails'
require 'shoulda-matchers'
require 'rspec/autorun'

RSpec.configure do |config|
  config.use_transactional_fixtures = true
  config.order = "random"
end
```

It loads all the libraries it needs to work with, and configures RSpec to use transactional fixtures and to execute the tests in random order.

Note that this time, `spec_helper` is just another file, it can be named differently and it will still work as longs as it is required with the correct name.


## Minitest

Minitest allows us to name our directories the way we want, but we need to have it in mind because it is important when running our tests.

I like to follow the conventions, so I created the project with a structure very similar to the RSpec's one.

```
fizz_buzz_minitest
├── lib
│   └── fizz_buzz.rb
└── test
    └── fizz_buzz_test.rb
```

`#test/fizz_buzz_test.rb`

```ruby
require 'minitest/autorun'
require 'fizz_buzz'

describe FizzBuzz do
  it 'returns "1" when receives 1' do
    assert '1', FizzBuzz.new.convert(1)
  end
end
```

`#lib/fizz_buzz.rb`

```ruby
class FizzBuzz
  def convert(n)
    '1'
  end
end
```

The first thing I'd like to point out is that in Minitest it is mandatory to require the Minitest library, specifically `minitest/autorun`, which provides everything we need to execute the test (e.g. the assertions methods). Note that I'm using the _spec_ syntax here, but it is also possible to use the classic syntax for Minitest.

Now, we don't have a `minitest` command to run, as we had with RSpec. Minitest is a very basic (but very powerful) testing framework. One of its strengths is that it is not a DSL (like RSpec), but simple Ruby (although our syntax uses the DLS style). Anyway, in order to execute our test, we need to run just ruby.

```
[fizz_buzz_minitest]$ ruby test/fizz_buzz_test.rb
[...]core_ext/kernel_require.rb:54:in `require': cannot load such file -- fizz_buzz (LoadError)
...
        from test/fizz_buzz_test.rb:1:in `<main>'
```

Humm, it actually executed something, but it didn't go well. What happened here is that it was not possible to load the `fizz_buzz` file. Luckily, it is something easy to solve. Ruby includes `require_relative` to require a file specifying a relative path. This path must be built based on the current file path. In our case, our test file is inside a `test` directory, so the path has to move backward one directory (with `..`) and then add the path to the required file.

Our testing file now looks this way.

`#test/fizz_buzz_test.rb`

```ruby
require 'minitest/autorun'
require_relative '../lib/fizz_buzz'

describe FizzBuzz do
  it 'returns "1" when receives 1' do
    assert '1', FizzBuzz.new.convert(1)
  end
end
```

We can run our tests again.

```
[fizz_buzz_minitest]$ ruby test/fizz_buzz_test.rb
Run options: --seed 21470

# Running:

.

Finished in 0.000718s, 1393.3787 runs/s, 1393.3787 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

Now it works. Note that you must run the command from the project base directory, not from `lib` or `test`.

However, there is a little problem with this approach. If we re-arrange the directory structure, either test or lib, we need to change the relative path. For example.

```
fizz_buzz_minitest
├── lib
│   └── models
│       └── fizz_buzz.rb
└── test
    └── models
        └── fizz_buzz_test.rb
```

Then we need to change the relative path to:

```ruby
require_relative '../../lib/models/fizz_buzz'
```

In our example we have just one file, but in a real project it would be a big problem.

Fortunately, we have an alternative. We can execute our test file specifying in the command line where ruby must look for the required files. With our initial structure, we can execute the test this way.

```
[fizz_buzz_minitest]$ ruby -I lib test/fizz_buzz_test.rb
Run options: --seed 2633

# Running:

.

Finished in 0.000820s, 1220.1178 runs/s, 1220.1178 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

We used the `-I` option, indicating that ruby should load the directory `lib`.

Note that with minitest it doesn't matter how the files and directories are named, it will just work. Example:

```
fizz_buzz_minitest
├── my_own_directory
│   └── fizz_buzz.rb
└── my_testing_directory
    └── fizz_buzz_test.rb
```

Let's execute the test.

```
[fizz_buzz_minitest]$ ruby -I my_own_directory test/fizz_buzz_testing_file.rb
Run options: --seed 31542

# Running:

.

Finished in 0.000719s, 1389.9584 runs/s, 1389.9584 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

Everything is working. This is because we are not running the entire test suite, as we do with RSpec. In this case, we are only running one file and we specify what directory should be loaded.

### `test_helper`

In the same way we use a `spec_helper` file in RSpec, we can use a `test_helper` in minitest. Note that the file name does not matter, but `test_helper` is a widely used name when working with Minitest.

We can create a `test_helper.rb` file in our `test` directory.

```
fizz_buzz_minitest
├── lib
│   └── fizz_buzz.rb
└── test
    ├── fizz_buzz_test.rb
    └── test_helper.rb
```

The content for our `test_helper` file would be a little better than the `spec_helper`, because now we can add some code at least.

```ruby
require 'minitest/autorun'
require 'minitest/pride'
```
We extracted the require of Minitest's autorun to the `test_helper` file, and also required `pride`, a small library from Minitest that colorize the output.

Now we need to change the require of our testing file.

`#spec/fizz_buzz_test.rb`

```ruby
require 'test_helper'
require 'fizz_buzz'

describe FizzBuzz do
  it 'returns "1" when receives 1' do
    assert '1', FizzBuzz.new.convert(1)
  end
end
```

And try to run it.

```
[fizz_buzz_minitest]$ ruby -I lib test/fizz_buzz_test.rb

[...]core_ext/kernel_require.rb:54:in `require': cannot load such file -- test_helper (LoadError)
...
        from spec/fizz_buzz_test.rb:1:in `<main>'
```

Ups, something failed.

The reason is that the required file couldn't be loaded. We indicated ruby that it should load the `lib` directory, but the `test_helper` is not there. It is in the `test` directory. Well, for that case, we can use `require_relative`, but we could have the same potential problems: if we move the files into other directories, all the paths passed to `require_relative` should be updated as well.

Again, the best option is to specify that another directory should be loaded by ruby when executing the test file. We can do that by adding the `test` directory to the list (separated by a colon `:`)

```
[fizz_buzz_minitest]$ ruby -I lib:test test/fizz_buzz_test.rb
Run options: --seed 64180

# Running:

.

Fabulous run in 0.000797s, 1254.2865 runs/s, 1254.2865 assertions/s.

1 runs, 1 assertions, 0 failures, 0 errors, 0 skips
```

And... every thing is working again.

As we can see, Minitest does not assume a specific directory structure, because we can specify its dependencies as command line options. However, I recommend to use the same structure we use with RSpec because is a very widespread one. Any developer could figure out where to find which files, and it is easy to grow if we want to use rake (for example) to run our test in batches. We will talk about that in a future post.
