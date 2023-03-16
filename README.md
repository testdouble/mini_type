[![Ruby Style Guide](https://img.shields.io/badge/code_style-standard-brightgreen.svg)](https://github.com/testdouble/standard)
[![build](https://github.com/testdouble/mini_type/actions/workflows/main.yml/badge.svg)](https://github.com/testdouble/mini_type/actions/workflows/main.yml)
![license](https://img.shields.io/github/license/testdouble/mini_type)

# MiniType

MiniType is a small runtime type checking system for Ruby. MiniType does not require any setup other than installing the gem, and adding `accepts` definitions to your methods.

## Quick Start

```ruby
require "mini_type"

class AmazingClass
  # include MiniType in your class
  include MiniType

  def initialize(param1, param2 = nil)
    accepts {{ param1: String, param2: [Integer, NilClass] }}
    # param1 should only be a String
    # param2 can be a String or `nil`
  end

  def print_name(name:)
    accepts {{ name: String }}
    # works with positional arguments and with keyword arguments
  end

  def self.output_array(array)
    accepts {{ array: array_of(String, NilClass) }}
    # allow an array filled with defined types of objects
  end

  def self.output_hash(hash)
    accepts {{ hash: hash_with(:key1, :some_other_key) }}
    # define that a argument is a hash that should have certain keys
  end

  def self.render(thing)
    accepts {{ thing: with_interface(:render, :foo) }}
    # define that an argument must respond to a given interface
  end
end
```

## Installation

Install the gem and add to the application's Gemfile by executing:

    $ bundle add mini_type

If bundler is not being used to manage dependencies, install the gem by executing:

    $ gem install mini_type

## Why?

Having worked in both typed and untyped langauges I find myself loving the flexibility and clarity of Ruby, but missing the guidance that typing gives you when trying to understand a large or complex codebase. MiniType is designed to help document how methods are expected to be used, as well as providing an easy way to provide runtime guarding against unexpected input. For example, given this method:

```ruby
def add_one(input)
  input += 1
end
```

it is easy to imagine giving this method unexpected input like a `String` or other objects. Additionally in more complex examples it might be hard to work out what the expected input actually is, for example:

```ruby
def render(input)
  puts input.render_to_string
end
```

MiniType makes it easy to document and enforce expectations in your code:

```ruby
def render(input)
  accepts {{ input: RenderableObject }}
  puts input.render_to_string
end
```

now anyone working with this code can see at a glance what type of object it's expecting to be given, and if the `render` method is given anything other than a `RenderableObject` it will raise an exception with a clear error message:

```
Expected argument ':input' to be of type 'RenderableObject', but got 'String'
```

## Type declarations:

```ruby
# accept an object of a given class
accepts {{ arg1: String }}

# accept an object that is one of a list of classes
accepts {{ arg1: [String, OtherClass, ThirdClass] }}

# accept an array filled with a specific class
accepts {{ arg1: array_of(String) }}

# accept an array filled with any of a list of classes
accepts {{ arg1: array_of(String, OtherClass, ThirdClass) }}

# accept a hash that must have the specified keys
accepts {{ arg1: hash_with(:foo, :bar) }}

# accept an object that must respond to the specified methods
accepts {{ arg1: with_interface(:method1, :method2) }}
```

## How it works

The `accepts` method takes a block as the only argument, and the block should return a hash containing the local variables you want to check. This can be expressed in two ways:

```ruby
accepts do
  { foo: String }
end

# or the preferred style:
accepts {{ foo: String }}

```

In Ruby blocks capture information about the context in which they were defined. This allows us to inspect variables that were local to the block when it was defined like so:

```ruby

def accepts(&block)
  context = block.binding
  context.local_variables.each do |name|
    puts "Name: #{name}"
    puts "Value: #{context.local_variable_get(name).inspect}"
  end
end

foo = 1
bar = "abc"

accepts {{ }} # block that returns empty hash

# Output:
# => Name: bar
# => Value: "abc"
# => Name: foo
# => Value: 1
```

The nice thing about this is it allows `MiniType` to be implemented in a really simple way with no 'magic'! 🎉


## Development

After checking out the repo, run `bundle install` to install dependencies. Then, run `bundle exec rspec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and the created tag, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/testdouble/mini_type. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [code of conduct](https://github.com/testdouble/mini_type/blob/main/CODE_OF_CONDUCT.md).

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the MiniType project's codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/testdouble/mini_type/blob/main/CODE_OF_CONDUCT.md).
