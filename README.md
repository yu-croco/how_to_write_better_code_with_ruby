# How to write better code with Ruby
- This repository is about how to write better (cleaner/ more receivable) code with Ruby.
- This is inspired by [Sandi Metz' Rules For Developers - Thoughtbot](https://thoughtbot.com/blog/sandi-metz-rules-for-developers) and [Practical Object-Oriented Design in Ruby](https://www.amazon.co.jp/Practical-Object-Oriented-Design-Ruby-Addison-Wesley/dp/0321721330/ref=sr_1_6?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2EGHAGFK7S380&keywords=object+oriented+programming&qid=1563603741&s=gateway&sprefix=object+ori%2Caps%2C295&sr=8-6). I recommend you to read them. :)

## Table of Contents
1. One of the most important things when you write code
2. Concrete simple example to write better code with Ruby
3. Other tips to write cleaner codes

## 1. One of the most important things  when you write code
Although there are many theories, DRY / SOLID / design patterns or so on, in the world, one of the most important things is `Separation by interest` called Single Responsibility Principle in my opinion.

There are a few reasons.
- We need to implement feature, considering specification changes.
  - If we implement a feature at one method and specification changes happen...
- Simple code helps you to understand easily what it really does.
  - easy to read, understand and modify.

In the real world, we always encounter specification changes, like or not.

So we need to have to care how to implement freatures with `flexible code`.

In order to implement features with flexible code, `Separation by interest` is the key.

## 2. Concrete simple example to write better code with Ruby
Let me explain the details of `Separation by interest` with a simple example below.

The PaymentReport class wants to calculate the payment fee during the specific period.

This code is bad, in my opinion, because total_payment_within_period handles many things, in spite of the method name.

```ruby
# Bad
class PaymentReport
  attr_reader :start_date, :end_date, :customers

  def initialize(start_date, end_date, customers)
    @start_date = start_date
    @end_date = end_date
    @customers = customers
  end

  def total_payment_within_period
    payed_customers = []
    customers.each do |customer|
     if customer.payed_at >= start_date && customer.payed_at <= end_date && customer.status == 'confirmed'
      payed_customers << customer
    end
    total = 0
    payed_customers.each do |customer|
      total += customer.payment_fee
    end
  end
end
```

Let's separate logics by following `Separation by interest`.

We do not have to do everyhting in one method.
```ruby
# A little Better
class PaymentReport
  attr_reader :start_date, :end_date, :customers

  def initialize(start_date, end_date, customers)
    @start_date = start_date
    @end_date = end_date
    @customers = customers
  end

  def total_payment_within_period
    payed_customers = filter_payed_customers # <-- transfer logic
    calculate_payment(payed_customers) # <-- transfer logic
  end

  # just separate logic to outside of total_payment_within_period
  def filter_payed_customers
    customers.each do |customer|
      if customer.payed_at >= start_date && customer.payed_at <= end_date && customer.status == 'confirmed'
        payed_customers << customer
      end
    end
  end

  # just separate logic to outside of total_payment_within_period
  def calculate_payment(payed_customers)
    total = 0
    payed_customers.each do |customer|
      total += customer.payment_fee
    end
  end
end
```

Wow!

We just moved logics to new methods, but now `total_payment_within_period` method has only 2 lines!
Now, the code inside the method is easier to understand.

But we still have a problem.

The `IF` logic at `filter_payed_customers` method is difficult to understand.　And it does not seem to define this concrete logic here because the if conditions depend on payment_statement object, not payment_report object.

So let's transfer the logics to where they should be.

```ruby
# Better
class PaymentReport
  attr_reader :start_date, :end_date, :customers

  def initialize(start_date, end_date, payment_statements)
    @start_date = start_date
    @end_date = end_date
    @payment_statements = payment_statements
  end

  def total_payment_within_period
    payed_statements = filter_payed_customers
    calculate_payment(payed_statements)
  end

  def filter_payed_customers
    payed_statements = []
    payment_statements.each do |statement|
      if payment.countable? # <-- transfer logic
        payed_statements << statement
      end
    end
  end

  def calculate_payment(payed_statements)
    total = 0
    payed_statements.each do |statement|
      total += statement.fee
    end
  end
end

# payment_statement object has to have specific logic on its own class
class PaymentStatement
  attr_reader :start_date, :end_date, :status, :payed_at

  def initialize(start_date, end_date, status, payed_at)
    @start_date = start_date
    @end_date = end_date
    @status = status
    @payed_at = payed_at
  end

# do one thing at one method and just call them!
  def countable?
    payed_at_between? && confirmed?
  end

  def payed_at_between?
    payed_at >= start_date && payed_at <= end_date
  end

  def confirmed?
    status == 'confirmed'
  end
end
```

So, now we just call `countable?` instance method at filter_payed_customers method, instead of defining the detail logic.

I'd like to improve a little bit more by using convenient methods of Ruby.

```ruby
# Good
class PaymentReport
  attr_reader :start_date, :end_date, :status, :payed_at

  def initialize(start_date, end_date, payment_statements)
    @start_date = start_date
    @end_date = end_date
    @payment_statements = payment_statements
  end

  def total_payment_within_period
    payed_statements = filter_payed_customers
    calculate_payment(payed_statements)
  end

  # If you assign temporal Array or Hash, you may use each_with_object
  def filter_payed_customers
    payment_statements.each_with_object([]) do |arr, statement|
      arr << statement if payment.countable?
    end
  end

  # sum method is convenient!
  def calculate_payment(payed_statements)
    payed_statements.map(&:fee).sum
  end
end

class PaymentStatement
  attr_reader :start_date, :end_date, :status, :payed_at

  def initialize(start_date, end_date, payment_statements)
    @start_date = start_date
    @end_date = end_date
    @payment_statements = payment_statements
  end

  def countable?
    payed_at_between? && confirmed?
  end

  def payed_at_between?
    payed_at >= start_date && payed_at <= end_date
  end

  def confirmed?
    status == 'confirmed'
  end
end
```

Now, the code is cleaner and easy to understand!
Each method concentrates on one thing, so it's easier to handle specification changes with smaller bad impacts to existing logics.

## 3. Other tips to write cleaner codes
Let me share you some tips to write cleaner code.

## Duck Typing
Objects know which class they belongs to and how to react against assined methods.

So, all you have to do is to give a method to a object, instead of separate by conditions.

The code below is really bad.
This is because `1. 'when' clause grows easily when new employee type is added` and `2. calc_salary has to have big logic in a logic (It violates SRP!)`.
```ruby
# NotGood
class Salary
  def calc_salary(employees)
    employees.each do |employee|
      case employee
      when FullTime
        # write salary calculation for fulltime employee
      when PartTimer
        # write salary calculation for part time
      when Outsourcing
        # write salary calculation for outsourcing
      end
    end
  end
end
```

Let's just give a method to objects.
Each object can react aganst calc_salary method.
```ruby
# Better
class Salary
  def calc_salary(employees)
    employees.each do |employee|
      # send samle method to all employee object because each object knows how to handle method
      employee.calc_salary(self)
  end
end

class FullTime
  def calc_salary(data)
    # write specific logic
  end
end

class PartTimer
  def calc_salary(data)
    # write specific logic
  end
end

class Outsourcing
  def calc_salary(data)
    # write specific logic
  end
end
```

### Lazy initialization
You can save memory until you really use the method by using lazy initialization.

Here is a simple example.

In this case, `cry` method just returns string, but what if the ,method has big calculation logic?

The logic at cry method will be evaluated only when it's really needed.

In addition, the value will be cached as instance variable, so when you call twice at same instance object, cry method just returns @cry instead of evaluating the logic again.

```ruby
# Get data from DB every time when you call this method
class Cat
  def cry
    'Meowwwwwww'
  end
end

# if '@sleep' is not nil, it does not get data from DB
class Cat
  def cry
    @cry ||= 'Meowwwwwww'
  end
end
```

If you want to write multi lines code, you can use `begin ~ end` like below.
```ruby
# usual
def configure_setting
  config = build_base_config
  if config.valid_values?
    @config = set_config(config)
  else
    @config = set_default_config(config)
  end
end

def configure_setting
  @config ||= begin
    config = build_base_config
    if config.valid_values?
      set_config(config)
    else
      set_default_config(config)
    end
  end
end
```

### Cleaner I/O
We usually use `if`, but it let us understand difficultly. I mean it give you some stress to understand what the `if` return at last.

The example below, if returns string, the contens of greeting. But it looks a little stressful to understand what `if` returns at last.

```ruby
# Not Good
# I/O is not clear
def greete(current_user)
  if current_user.admin
    greete = 'hello admin'
  elsif current_user.special_user
    greete = 'hello special user'
  else
    greete = 'hello user'
  end
  say_hello_to_user(greete)
end
```
In this case, you can write code like this.
Now it's easier to understand what `if` returns.
```ruby
# Better
# I/O is clear
def greete
  greete =
    if current_user.admin
      'hello admin'
    elsif current_user.special_user
      'hello special user'
    else
      'hello user'
    end
  say_hello_to_user(greete)
end
```

### Multiple declaration
We usually declare a few temporal variable in a method, like below.
```ruby
# This code is Rails example
# NotGood
class NotebooksController < ApplicationController
  def index
    per = params[:per]
    offset = params[:offset]
    query_key = params[:q]
    # more params...
  end
end
```
You can write like this.

```ruby
# Better
class NotebooksController < ApplicationController
  def index
    per, offset, query_key =  set_attribute
  end

  private

  def set_attribute
    [
      params[:per],
      params[:offset],
      params[:q],
      # more params...
    ]
  end
end
```

## License
The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
