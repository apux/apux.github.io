---
title: Using Vim Regexp to Update to RSpec 3 Syntax
date: 2015-07-20 17:23 UTC
tags: [vim, regexp, rspec]
---

Recently, I had to update a project that uses RSpec. Part of the migration consisted in updating the RSpec syntax from version 2 to version 3. Since I use Vim, I ended up creating a bunch of regular expressions to make those updates. Here I will show you how I created them (and what I learned doing it).

The main change is RSpec 3 is not monkey-patching objects any more, and because of that it is not possible to use sentences like this:

```ruby
FizzBuzz.new(1).value.should == "1"
```

The new way to do it in RSpec 3 is:

```ruby
expect(FizzBuzz.new(1).value).to eq "1"
```

And that is the change we need to do with our regular expressions.

Vim's regular expressions are a little different from Ruby's one because we will need to escape more characters in order to have it working.

In Ruby, a regexp that finds a `.should` could be:

```
/\.should/
```

For example:

```ruby
"value.should == 1".match /\.should/
# => #<MatchData ".should">
```

By escaping the dot `\.` we ensure that it matches an actual dot `.`, otherwise, it will match any character because the dot actually means that in regular expressions: any character.

Example with unescaped dot:

```ruby
"value.should == 1".match /.should/
# => #<MatchData ".should">

"value_should == 1".match /.should/
# => #<MatchData "_should">
```

Example with escaped dot:

```ruby
"value.should == 1".match /\.should/
# => #<MatchData ".should">

"value_should == 1".match /\.should/
# => nil
```

This is the behaviour we are looking for.

It also works the same way in Vim. If we search this pattern in Vim, it will find a line that contains `.should`.

We can try it in Vim's search mode.

```
/\.should
```

It works as expected. Great!

## Capture group

Now, we want to capture the string (the group of characters) that precedes `.should`. We can do this in Ruby:

```
/(.+)\.should/
```

Te plus sign `+` indicates one or more repetitions of the previous character. In this case the previous character is *any character*, since we did not escape de dot `.` this time because it is the behaviour we are looking for. It will match any string that contains `.should` and it will capture all the characters behind it.

Let's see how it works.

```ruby
"value.should == 1".match /.+\.should/
# => #<MatchData "value.should" 1:"value">
```

You can see it matched and, additionally, it contains the string we captured (`1:"value"`).

But if we try to use it in Vim it will not work. We need to escape both parenthesis `(` and `)`, and the plus sign `+`.

```
/\(.\+\)\.should/
```

Now it will match in Vim.

In Ruby, it is possible to use the captured string with the special global variable `$1`. Example:

```ruby
"value.should == 1".match /(.+)\.should/
$1
# => "value"
```

In Vim, this value is stored in `\1` and we can use it while replacing. Example:

```
:%s/\(.\+\)\.should/expect(\1).to/g
```

And it worked! It changed all the lines containing `.should` .

There was only one problem, if the line is indented (and chances are it is), our replace command will include the blank spaces inside the parenthesis.

```ruby
expect(      value).to eq("1")
```

To avoid it, we need to improve our regexp. Let's do that by capturing all the blank spaces at the beginning of the line, and adding them to the new string. We can use `^` to indicate the beginning of a line in both Ruby and Vim, and it is not necessary to escape it.

```
:%s/^\( \+\)\(.\+\)\.should/\1expect(\2).to/g
```

Since a new captured pattern was introduced before the one we previously had, the numbering were also altered and now the blank spaces are stored in `\1` and the following string in `\2`.

## Variants

The regexp we just build is too generic for our requirements. Specifically, it ignores that the matchers have to change too.

RSpec 3 does not support some matchers that version 2 did. For example, `==`, and `=~`, `be_true` needs to be changed to `be_truthy`, etcetera.

So, I prefer to run specific replacements for each one. These are what I used.

```
:%s/^\( \+\)\(.\+\)\.should be_true/\1expect(\2).to be_truthy/g
:%s/^\( \+\)\(.\+\)\.should be_false/\1expect(\2).to be_falsey/g
:%s/^\( \+\)\(.\+\)\.should == nil/\1expect(\2).to be_nil/g
:%s/^\( \+\)\(.\+\)\.should ==/\1expect(\2).to eq/g
:%s/^\( \+\)\(.\+\)\.should =\~/\1expect(\2).to match/g
:%s/^\( \+\)\(.\+\)\.should_not ==/\1expect(\2).not_to eq/g
:%s/^\( \+\)\(.\+\)\.should/\1expect(\2).to/g
```

Executing these commands in this order does most of the work.

The first 3 commands change the expectations about booleans and nil. The forth one changes the `==` matcher to `eq`. Then we change the `=~` (for regexp) to `match` (note that we had to escape the `~`). The following command replaces the `should_not` with `==` to `not_to eq`, and, finally, we use the generic replace to change everything else (this will match remaining expectations like `render.should redirect_to(path)`).

## stub

Stubbing syntax was also modified in this version.

In RSpec 2, this was the syntax.

```ruby
my_object.stub(:my_method).and_return(1)
```

In RSpec 3, this is the new syntax.

```ruby
allow(my_object).to receive(:my_method).and_return(1)
```

So, we need to capture the indentation spaces and the object. It could be done with this:

```
:%s/^\( \+\)\(.\+\)\.stub/\1allow(\2).to receive/g
```

But again, it is too generic, because we can have `any_instance.stub` and `.stub_chain` and these lines will match too, but, obviously, it will not be changed correctly. In that case, it is preferable to replace them first and execute our replacements in this order.

```
:%s/^\( \+\)\(.\+\)\.any_instance.stub/\1allow_any_instance_of(\2).to receive/g
:%s/^\( \+\)\(.\+\)\.stub_chain/\1allow(\2).to receive_message_chain/g
:%s/^\( \+\)\(.\+\)\.stub/\1allow(\2).to receive/g
```

Now we have it. In my case, I had no more special cases to attend, but if you have, I hope this explanation could help you to build your own commands for replacing.

Any corrections or suggestions are welcome.
