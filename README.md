# How to write better code with Ruby
- This repository is about how to write better (cleaner/ more receivable) code with Ruby.
- This is inspired by [Sandi Metz' Rules For Developers - Thoughtbot](https://thoughtbot.com/blog/sandi-metz-rules-for-developers) and [Practical Object-Oriented Design in Ruby](https://www.amazon.co.jp/Practical-Object-Oriented-Design-Ruby-Addison-Wesley/dp/0321721330/ref=sr_1_6?__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&crid=2EGHAGFK7S380&keywords=object+oriented+programming&qid=1563603741&s=gateway&sprefix=object+ori%2Caps%2C295&sr=8-6). I recommend you to read them. :)

## Table of Contents
1. The most important things when you write code
2. Concrete simple example to write better code with Ruby
3. Other tips to write cleaner codes (coming soon...)

## 1. The most important things  when you write code
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
This method says "I calculate the payment during the specific period.".
```ruby
# Bad
class PaymentReport
  def initialize(start_date, end_date, customers)
    @start_date = start_date
    @end_date = end_date
    @customers = customers
  end

  def total_payment_within_period(customers)
    payed_customers = []
    customers.each do |customer|
     if customer.payed_at >= @start_date && customer.payed_at <= @end_date && customer.status == 'confirmed'
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
  def initialize(start_date, end_date, customers)
    @start_date = start_date
    @end_date = end_date
    @customers = customers
  end

  def total_payment_within_period(customers)
    payed_customers = filter_payed_customers(customers) # <-- transfer logic
    calculate_payment(payed_customers) # <-- transfer logic
  end

  # just separate logic to outside of total_payment_within_period
  def filter_payed_customers(customers)
    customers.each do |customer|
      if customer.payed_at >= @start_date && customer.payed_at <= @end_date && customer.status == 'confirmed'
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
The `IF` logic at `filter_payed_customers` method is difficult to understand.
And it does not seem to define this concrete logic here because the if conditions depend on payment_statement object, not payment_report object.
So let's transfer the logics to where they should be.

```ruby
# Better
class PaymentReport
  def initialize(start_date, end_date, payment_statements)
    @start_date = start_date
    @end_date = end_date
    @payment_statements = payment_statements
  end

  def total_payment_within_period(payment_statements)
    payed_statements = filter_payed_customers(payment_statements)
    calculate_payment(payed_statements)
  end

  def filter_payed_customers(payment_statements)
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

  # do one thing at one method and just call them!
  def countable?(start_date, end_date)
    payed_at_between?(start_date, end_date) && confirmed?
  end

  def payed_at_between?(start_date, end_date)
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
  def initialize(start_date, end_date, payment_statements)
    @start_date = start_date
    @end_date = end_date
    @payment_statements = payment_statements
  end

  def total_payment_within_period(payment_statements)
    payed_statements = filter_payed_customers(payment_statements)
    calculate_payment(payed_statements)
  end

  # If you assign temporal Array or Hash, you may use each_with_object
  def filter_payed_customers(payment_statements)
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

  def countable?(start_date, end_date)
    payed_at_between?(start_date, end_date) && confirmed?
  end

  def payed_at_between?(start_date, end_date)
    payed_at >= start_date && payed_at <= end_date
  end

  def confirmed?
    status == 'confirmed'
  end
end
```

Now, the code is cleaner and easy to understand!
Each method concentrates on one thing, so it's easier to handle specification changes with smaller bad impacts to existing logics.

to be continued...
