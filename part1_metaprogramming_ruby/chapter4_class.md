# Định nghĩa class
 - Trong class bạn có thể viết bất cứ thứ gì, và nó cũng sẽ return dòng code cuối cùng của class

```ruby
class MyClass
  puts "Hello"
end

# => Hello
```

- Bởi vì bản thân class nó cũng là một object

## Current class

- Bất cứ nơi đâu trong Ruby đều có 1 current object gọi là `self`
- Tương tự như vậy, chúng ta cũng có current class hoặc current module
- Tuy nhiên, chúng ta không có một keyword nào để lấy current class.
- Nhưng chúng ta có thể xác định được current class thông qua code:
  - Ở Top-Level current class chính là `Object`.
  - Trong một method, current class chính là class của current object
  - Khi bạn open một class với keyword class thì current class chính là class mà bạn định nghĩa

### class_eval()

- Bạn có thể mở 1 class mà không cần biết tới tên của class đó

```ruby
def add_method_to(a_class)
  a_class.class_eval do
    def hi
      puts "hello"
    end
  end
end

add_method_to String
"abc".hi # => Hello
```

- `Module#class_eval` rất khác với `Object#instance_eval`, instance eval chỉ thay đổi self, trong khi class_eval thay đổi cả self lẫn current class

## Class instance variables

- Trình thông dịch của Ruby giả định rằng tất cả instance variables đều thuộc về self. Điều này cũng đúng trong class
- Trong định nghĩa class, self sẽ chính là class đó, chính vì vậy biến instance được định nghĩa trong class sẽ thuộc về class đó, nó khác với biến instance của object của class đó

```ruby
class MyClass
  @my_var = 1

  def self.read
    @my_var
  end

  def write
    @my_var = 2
  end

  def read
    @my_var
  end
end

obj = MyClass.new
obj.read # => nil
obj.write
obj.read # => 2
MyClass.read # => 1
```

- Bởi vì class thực chất cũng chỉ là 1 object của `Class`
- `@my_var` được định nghĩa thuộc về object của class `obj` thì gọi là instance variable của object `obj`. Trong khi `@my_var` thuộc về class MyClass thì nó là một instance variables của object `MyClass`, có thể gọi là class instance variables
- Nếu bạn muốn lưu trữ 1 biến trong class thì ngoài Class instance variable có thể dùng Class Variables - được định nghĩa bằng `@@`. Điểm khác biệt là Class Variable có thể được truy cập ở class con, từ các method chứ không đơn giản là truy cập từ class object như Class Instance Variable.

## Singleton methods

- Là method định nghĩa riêng cho 1 object
- Có thể định nghĩa trực tiếp từ object đó hoặc thông qua `Object#define_singleton_method`

```ruby
str = "Hello"

def str.title?
  self.upcase == self
end

str.title? # => false
str.singleton_methods # => [:title?]
```

- Định nghĩa class method giống như Singleton Method

## Singleton Class

- method bình thường sẽ được lưu trong class
- Singleton method sẽ được lưu trong singleton class, đây là 1 hidden class
- Chỉ có 1 loại object - Đó là object bình thường hoặc là 1 module
- Chỉ có 1 loại module - Đó là module bình thường, class, hoặc singleton class
- Chỉ có 1 loại method, và nó sống trong module, class
- Mỗi object hoặc class sẽ có một "real class" của nó - Đó có thể là class bình thường hoặc singleton class
- Mỗi class đều sẽ có ancestors chain, trừ BasicObject
- Superclass của singleton class của 1 object chính là class của object đó. Superclass của 1 singleton class chính là singleton class của superclass của class đó.
- Mỗi khi gọi method, Ruby sẽ tìm đến "real class" của receiver và lần theo ancesters chain của class đó cho đến khi tìm thấy method

## Class Method

- Class method thực chất chỉ là 1 singleton method được lưu trong singleton class của class đó. Chính vì vậy nó có 3 cách khai báo như sau

```ruby
def MyClass.a_class_method
end

class MyClass
  def self.a_class_method
  end
end

class MyClass
  class << self
    def a_class_method
    end
  end
end
```

## Class method and include

```ruby
class A
  class << self
    include M
  end
end

# same

class A
  extend M
end
```
