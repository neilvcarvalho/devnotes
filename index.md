# Neil's development notes

## Code Smells

Code smells are signs that a refactoring is necessary. It's very important
to get rid of code smell while they are small, because as they increase in your
codebase, they can lead to resistance to change.

## 1 - Bloaters

Bloaters are classes and methods that have increased in size and complexity so
much that they are really hard to work with. They are those classes that no one
wants to touch and, in general, when someone has to make changes on them to fix
a bug, many other bugs appear.

## 1.1 - Long Method
A method that contains too many lines of code. It's easier to write code than to
read it, as one line of code is written only once and read many times by
different people. A long method that has more than 10 or even 5 lines of code
requires a greater effort to understand and many fragmentss of this method
represent smaller concepts that could have their own methods.

The treatment to long methods is:
- [Exctract Method](#1-extract-method)

If extracting a method is hard, those refactoring techniques could be useful:
- [Replace Temp with Query](#2-replace-temp-with-query)
- [Introduce Parameter Object](#3-introduce-parameter-object)
- [Preserve Whole Object](#4-preserve-whole-object)
- [Replace Method with Method Object](#5-replace-method-with-method-object)

For conditionals and loops:
- [Decompose Conditional](#6-decompose-conditional)

## 1.2 Large Class
A class that has too many fields, methods and/or lines of code. Large classes
tend to have too many responsibilities, violating the Single Responsibility
Principle. They usually start small, but it's less mentally taxing to place new
features in existing classes than to create new classes.

The treatments to Large Classes are:
- [Extract Class](#8-extract-class)
- [Extract Subclass](#9-extract-subclass)
- [Extract Interface](#10-extract-interface)
- [Duplicate Observed Data](#11-duplicate-observed-data)


## 1.3 Primitive Obsession

Primitive Obsession is the tendency to use primitive types instead of creating
small classes, such as date ranges and currencies. Classes that have such fields
have to implement all this shared behavior and become large classes over time.

Primitives also tend to spread to other parts of the code as constants.

The treatments to Primitive Obsession are:
- [Replace Data Value with Object](#12-replace-data-value-with-object)
- [Introduce Parameter Object](#3-introduce-parameter-object)
- [Preserve Whole Object](#4-preserve-whole-object)
- [Replace Type Code with Class](#13-replace-type-code-with-class)
- [Replace Type Code with Subclasses]
- [Replace Type Code with State/Strategy]
## Refactoring catalog

### 1. Extract Method

Problem: You have a code fragment that can be grouped together:

```ruby
def backup_data
  ObjectStorage.new(generate_user_data_file).save

  # Notify by e-mail
  BackupMailer.notify_user(user).deliver_later
  BackupMailer.notify_sysadmin(user).deliver_later
end
```

Solution: Move this fragment to a new method and replace the old code with
a call to this new method.

```ruby
def backup_data
  ObjectStorage.new(generate_user_data_file).save

  notify_by_email
end

def notify_by_email
  BackupMailer.notify_user(user).deliver_later
  BackupMailer.notify_sysadmin(user).deliver_later
end
```

### 2. Replace Temp with Query

Problem: You place the result of an expression in a local variable for later
use in your code.

```ruby
def discounted_price
  base_price = quantity * item_price

  if base_price > 1000
    base_price * 0.95
  else
    base_price * 0.98
  end
end
```

Solution: Move the entire expression to a separate method and return its result.
Query the method in the places you were previously using the local variable.

```ruby
def discounted_price
  if base_price > 1000
    base_price * 0.95
  else
    base_price * 0.98
  end
end

def base_price
  quantity * item_price
end
```

### 3. Introduce Parameter Object
Problem: Your method contains a repeating group of parameters

```ruby
class SalesReport
  def amout_invoiced_in(start_date:, end_date:)
    # ...
  end

  def amout_received_in(start_date:, end_date:)
    # ...
  end

  def amout_overdue_in(start_date:, end_date:)
    # ...
  end
end
```

Solution: Replace these paramteres with an object
```ruby
class DateRange
  def initialize(start_date, end_date)
    # ...
  end

  # ...
end

class SalesReport
  def amout_invoiced_in(date_range)
    # ...
  end

  def amout_received_in(date_range)
    # ...
  end

  def amout_overdue_in(date_range)
    # ...
  end
end
```

### 4. Preserve Whole Object
Problem: You get several values from an object and pass them as arguments to a
method.

```ruby
class TemperatureCheck
  # ...

  def within_plan?
    low = days_temp_range.lowest_temperature
    high = days_temp_range.highest_temperature

    plan.temperature_within_range?(low, high)
  end
```

Solution: Try passing the whole object as an argument.

```ruby
class TemperatureCheck
  # ...

  def within_plan?
    plan.temperature_within_range?(days_temp_range)
  end
```

### 5. Replace Method with Method Object
Problem: You have a long method in which the local variables are so dependent
on one another that you can't extract a method

```ruby
class Order
  # ...

  def price
    primary_base_price = # Information from order
    secondary_base_price = # Information from order
    tertiary_base_price = # Information from order

    # Perform long computation
  end
end
```

Solution: Extract the method to its own class so that you can use the local
variables as class fields, then extract methods within the new class.

```ruby
class Order
  # ...

  def price
    PriceCalculator.new(order).call
  end
end

class PriceCalculator
  def initialize(order)
    # Copy information from order
  end

  def call
    # Perform long computation
  end

  private

  attr_accessor :primary_base_price, :secondary_base_price, :tertiary_base_price
end
```

### 6. Decompose Conditional

Problem: You have a complex conditional.

```ruby
def charge
  if (date.before?(SUMMER_START) || date.after?(SUMMER_END))
    quantity * winter_rate + winter_service_charge
  else
    quantity * summer_rate
  end
end
```

Solution:
Decompose the complicated parts of the conditional into separate methods.

```ruby
def charge
  if is_summer?(date)
    summer_charge(quantity()
  else
    winter_charge(quantity)
  end
end

def is_summer?(date)
  date.before?(SUMMER_START) || date.after?(SUMMER_END)
end

def summer_charge(quantity)
  quantity * summer_rate
end

def winter_charge(quantity)
  quantity * winter_rate + winter_service_charge
end
```

### 7. Extract Method (loop example)
Problem: You have a code fragment that can be grouped together.

```ruby
def print_properties(users)
  users.each do |user|
    result = ""
    result += user.name
    result += " "
    result += user.age

    puts result
  end
end
```

Solution: Move this code to a separate new method and replace the old
code with a call to the new method.

```ruby
def print_properties(users)
  users.each do |user|
    puts user_properties(user)
  end
end

def user_properties(user)
  "#{user.name} #{user.age}"
end
```

### 8. Extract Class
Problem: You have a class with too many responsibilities.

```ruby
class Person
  attr_accessor :name, :office_area_code, :office_number

  def formatted_office_number
    "(#{office_area_code}) #{office_number}"
  end
end
```

Solution: Create a new class and move the relevant code to the new class.

```ruby
class Person
  attr_accessor :name, :office_phone

  def office_number
    office_phone.number
  end
end

class TelephoneNumber
  attr_accessor :area_code, :number

  def formatted_number
    "(#{area_code}) #{number}"
  end
end
```

### 9. Extract Subclass
Problem: A class has features that are used in only a few cases.

```ruby
class Item
  def total_price
    # Used frequently
  end

  def fuel_usage
    # Used only when the item is a vehicle
  end

  def license_plate
    # Used only when the item is a vehicle
  end
end
```

Solution: Extract the rarely used features to their own subclasses.

```ruby
class Item
  def total_price
    # Used frequently
  end
end

class Vehicle < Item
  def fuel_usage
    # Used only when the item is a vehicle
  end

  def license_plate
    # Used only when the item is a vehicle
  end
end
```

### 10. Extract Interface
Extracting an interface can help if it's necessary to have a list of the
operations and behaviors that a client can use.

Problem: Multiple clients are using
the same part of a class interface, or multiple classes share the same role.

Note: Ruby doesn't have `Interface` as Java does, but it's possible to use
modules and tests to guarantee that a role is shared between classes.

```ruby
class Car
  def accelerate
  end

  def break
  end

  def add_fuel
  end

  def add_to_trunk(item)
  end
end

class Motocycle
  def accelerate
  end

  def break
  end

  def add_fuel
  end

  def do_a_wheelie
  end
end
```

Solution:
Move this identical portion to its own interface.

```ruby
module MotorVehicle
  def accelerate
  end

  def break
  end

  def add_fuel
  end
end

class Car
  include MotorVehicle

  def add_to_trunk(item)
  end
end

class Motocycle
  include MotorVehicle

  def do_a_wheelie
  end
end
```

### 11. Duplicate Observed Data
If a large class is responsible to hold data for a graphical interface, you
can move some of this data to a separate domain object. It may be necessary to
store copies of the same data in two places and keep the data consistent.

Problem: Domain data is stored in classes responsible for the GUI.

```ruby
class IntervalWindow
  attr_accessor :start_field, :end_field, :length_field

  def start_field_focus_lost
  end

  def end_field_focus_lost
  end

  def length_field_focus_lost
  end

  def calculate_length
  end

  def calculate_end
  end
end
```

Solution: Separate the data into separate classes, ensuring those classes are
linked and synchronized between the domain class and the GUI.

```ruby
class IntervalWindow
  attr_accessor :start_field, :end_field, :length_field, :interval

  def start_field_focus_lost
  end

  def end_field_focus_lost
  end

  def length_field_focus_lost
  end
end

class Interval
  attr_accessor :start, :end, :length

  def calculate_length
  end

  def calculate_end
  end
end
```

### 12. Replace Data Value with Object

It may be possible to logically group some of the primitive fields you have in
your code to their own class - and move the behavior associated with this data
into this class.

Problem:
A class (or group of classes) contain a data field. The field has its own
behavior and associated data.

```ruby
class Order
  attr_accessor :customer_name

  def valid_customer_name?
    # ...
  end
end
```

Solution: Create a new class and move the data and behavior associated with that
field to the new class. Store the object of the class in the original class.

```ruby
class Order
  attr_accessor :customer
end

class Customer
  attr_accessor :name

  def valid_name?
    # ...
  end
end
```

### 13. Replace Type Code with Class

Problem: A class has a field that contains a type code. The values of this type
aren't used in operator conditions and don't affect the behavior of the program.

```ruby
class Person
  TYPE_O = 1
  TYPE_A = 2
  TYPE_B = 3
  TYPE_AB = 4

  attr_accessor :bloodgroup
end
```

Solution: Create a new class and use its objects instead of the type code values.

```ruby
class Person
  attr_accessor :bloodgroup
end

class BloodGroup
  TYPE_O = BloodGroup.new(significant_data_for_this_type)
  TYPE_A = BloodGroup.new(significant_data_for_this_type)
  TYPE_B = BloodGroup.new(significant_data_for_this_type)
  TYPE_AB = BloodGroup.new(significant_data_for_this_type)

  # ...
end
```