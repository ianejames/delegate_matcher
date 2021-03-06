# Delegate Matcher

An RSpec matcher for validating delegation. This matcher works with delegation based on the [Forwardable](http://ruby-doc.org/stdlib-2.0.0/libdoc/forwardable/rdoc/Forwardable.html) module,
the [delegate](http://api.rubyonrails.org/classes/Module.html#method-i-delegate) method in the Active Support gem or with
simple custom delegation.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'delegate_matcher'
```

And then execute:

```bash
$ bundle
```

Or install it yourself as:

```bash
$ gem install delegate_matcher
```

Then add the following to your ```spec_helper.rb``` file:


```ruby
require 'delegate_matcher'
```

## Usage

This matcher allows you to validate delegation to:
* instance methods
* class methods
* instance variables
* class variables
* constants
* arbitrary objects

```ruby
describe Post do
  it { should delegate(:name).to(:author)   } # name  => author().name    instance method
  it { should delegate(:name).to(:class))   } # name  => self.class.name  class method
  it { should delegate(:name).to(:@author)  } # name  => @author.name     instance variable
  it { should delegate(:name).to(:@@author) } # name  => @@author.name    class variable
  it { should delegate(:first).to(:GENRES)  } # first => GENRES.first     constant
  it { should delegate(:name).to(author)    } # name  => author.name      object
end
```

### Delegate Method Name

If the name of the method being invoked on the delegate is different from the method being called you
can check this using the ```with_prefix``` method (based on Active Support ```delegate``` method) or the
```as``` method.

```ruby
describe Post do
  it { should delegate(:name).to(author).with_prefix }          # author_name  => author.name
  it { should delegate(:name).to(author).with_prefix(:writer) } # writer_name  => author.name
  it { should delegate(:writer).to(author).as(:name) }          # writer       => author.name
end
```

### Handling Nil Delegates

If you expect the delegate to return ```nil``` when the delegate is ```nil``` rather than raising an error
then you can check this using the ```allow_nil``` method.

```ruby
describe Post do
  it { should delegate(:name).to(:author).allow_nil        } # name => author && author.name
  it { should delegate(:name).to(:author).allow_nil(true)  } # name => author && author.name
  it { should delegate(:name).to(:author).allow_nil(false) } # name => author.name
end
```

Nil handling is only checked if ```allow_nil``` is specified.

Note that matcher will raise an error if you use this when checking delegation to a constant or an
object since the matcher cannot validate ```nil``` handling in these cases.

### Arguments

If the method being delegated takes arguments then you can supply them with the ```with``` method. The matcher
will ensure that the provided arguments are in turn passed to the delegate.

```ruby
describe Post do
  it { should delegate(:name).with('Ms.')to(:author) }  # name('Ms.')  => author.name('Ms.')
end
```

Also, in some cases the delegator might make minor changes to the arguments. While this is arguably no
longer true delegation you can still check that arguments are correctly passed by using a second ```with```
method to specify the arguments expected by the delegate.

```ruby
describe Post do
  it { should delegate(:name).with('Ms.')to(author).with('Miss') }  # name('Ms.')  => author.name('Miss')
end
```

### Blocks

You can check that a block passed is in turn passed to the delegate via the ```with_block``` method.

```ruby
describe Post do
  it { should delegate(:name).to(author).with_a_block }    # name(&block) => author.name(&block)
  it { should delegate(:name).to(author).with_block }      # name(&block) => author.name(&block) alias for with_a_block

  it { should delegate(:name).to(author).without_a_block } # name(&block) => author.name
  it { should delegate(:name).to(author).without_block }   # name(&block) => author.name         alias for without_a_block
end
```

By default, block delegated is only check if ```with_a_block``` or ```without_a_block``` is specified.

### Active Support

You can test delegation based on the [delegate](http://api.rubyonrails.org/classes/Module.html#method-i-delegate) method in the Active Support gem.

```ruby
class Post
  attr_accessor :author

  class_variable_set(:@@authors, ['Ann Rand', 'Catherine Asaro'])
  GENRES ||= ['Fiction', 'Science Fiction']

  delegate :name,                to: :author
  delegate :name,                to: :author, prefix: true
  delegate :name,                to: :author, prefix: :writer
  delegate :name_with_nil_check, to: :author, allow_nil: true
  delegate :name_with_arg,       to: :author
  delegate :name_with_block,     to: :author
  delegate :count,               to: :@@authors
  delegate :first,               to: :GENRES
  delegate :name,                to: :class, prefix: true
end

class Author
  def name
    'Catherine Asaro'
  end

  def name_with_nil_check
    name
  end

  def name_with_arg(arg)
    "#{arg} #{name}"
  end

  def name_with_block(&block)
    "#{block.call} #{name}"
  end
end

describe Post do
  it { should delegate(:name).to(:author) }
  it { should delegate(:name).to(:@author) }
  it { should delegate(:name_with_nil_check).to(:author).allow_nil }
  it { should delegate(:name).to(:author).with_prefix }
  it { should delegate(:name).to(:author).with_prefix(:writer) }

  it { should delegate(:name_with_arg).to(:author).with('Ms.') }
  it { should delegate(:name_with_block).to(:author).with_block }
  it { should delegate(:count).to(:@@authors)   }
  it { should delegate(:first).to(:GENRES)   }
  it { should delegate(:name).to(:class).with_prefix   }
end
```
However, don't use the following features as they are not supported by the delegate method:
* delegation to objects
* different arguments passed to delegate

### Forwardable Module

You can test delegation based on the [Forwardable](http://ruby-doc.org/stdlib-2.0.0/libdoc/forwardable/rdoc/Forwardable.html) module.

```ruby
class Post
  extend Forwardable

  attr_accessor :author

  def_delegator :author, :name
  def_delegator :author, :name, :writer
  def_delegator :author, :name_with_arg
  def_delegator :author, :name_with_block
end

class Author
  def name
    'Catherine Asaro'
  end

  def name_with_arg(arg)
    "#{arg} #{name}"
  end

  def name_with_block(&block)
    "#{block.call} #{name}"
  end
end

describe Post do
  it { should delegate(:name).to(:author) }
  it { should delegate(:name).to(:@author) }
  it { should delegate(:writer).to(:author).as(:name) }
  it { should delegate(:name_with_arg).to(:author).with('Ms.') }
  it { should delegate(:name_with_block).to(:author).with_block }
end
```
However, don't use the following features as they are not supported by the Forwardable module:
* allow_nil
* delegation to class variables
* delegation to constants
* delegation to objects
* different arguments passed to delegate

## Contributing

1. Fork it ( https://github.com/dwhelan/delegate_matcher/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request

## Notes

This matcher was inspired by [Alan Winograd](https://github.com/awinograd) via the gist https://gist.github.com/awinograd/6158961
