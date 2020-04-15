## Cơ bản về block
- Block được định nghĩa trong cặp dấu `{}` hoặc `do...end`. Một convention cơ bản thì sử dụng `{}` khi viết block trên 1 hàng và `do...end` khi viết block trên nhiều hàng.
- Block chỉ được định nghĩa khi gọi một method và method gọi block thông qua từ khóa `yield`
- Bên trong method, bạn có thể kiểm tra lời gọi method hiện tại có tồn tại block hay không bằng hàm `Kenel#block_given?`

```ruby
def a_method
  return yield if block_given?
  "no block"
end

a_method                     # => "no block"
a_method {"this is a block"} # => "this is a block"
```

## Closures
Khi run một đoạn code, nó cần một môi trường chứa: local variables, instance variables, self, .... Toàn bộ những thành phần này được bao bọc trong object, nên chúng ta có thể gọi chúng là `bindings`.

Một đoạn code thực thi thực sự sẽ cần 2 thứ: đoạn code đó và bộ `bindings`

![code_runs](/images/code_runs.jpg)

Block sẽ bao gồm cả 2 thành phần trên và sẵn sàng để thực thi. Vì block không phải là object nên bạn sẽ tự hỏi rằng block sẽ lấy `bindings` ở đâu. Khi bạn định nghĩa block, nó sẽ lấy `bindings` tại thời điểm hiện tại, và mang chúng theo khi bạn truyền một block vào method

```ruby
def my_method
  x = "Good bye"
  yield("Tom")
end

x = "Hello"
my_method {|y| "#{x}, #{y}!"}
# => "Hello, Tom!"
```

Khi bạn define một block, nó sẽ lấy `bindings` hiện tại, như ví dụ trên là biến local `x`. Sau đó, bạn truyền block này vào một method với bộ `bindings` của method đó. Trong ví dụ trên, bộ `bindings` của `my_method` có biến local cũng tên là `x`. Nhưng block chỉ có thể nhìn thấy biến `x` xung quanh nơi nó định nghĩa, không thể nhìn thấy biến `x` của method mà nó được truyền vào.

Mặt khác, bạn cũng có thể định nghĩa thêm `bindings` bên trong block, nhưng những biến này sẽ biến mất sau khi kết thúc block

```ruby
def just_yield
  yield
end

top_var = 1

just_yield do
  top_var += 1
  local_var = 1
end

top_var   # => 2
local_var # => Error
```

Bởi vì những tính chất trên nên có thể gọi block là một bao đóng (Closures). Để hiểu thêm về closures, chúng ta cùng tìm hiểu về phạm vi của `bindings`, đó là `scope`.

## Scope
Hãy tưởng tượng bạn đặt một debugger trong chương trình Ruby. Bạn được chương trình thực thi đưa tới điểm breakpoint , khung cảnh xung quanh mà bạn nhìn thấy chính là scope.

Bạn có thể nhìn thấy `bindings` trong scope. Đó là các local variables, là các instance variables, methods của current object mà bạn đang đứng. Xa hơn bạn có thể nhìn thấy các constants, các global variables.

### Changing scope

Ví dụ sau sẽ cho thấy scope thay đổi như thế nào khi chương trình thực thi, tracking thông qua `Kenel#local_variables`

```ruby
v1 = 1

class MyClass
  v2 = 2
  local_variables # => [:v2]

  def my_method
    v3 = 3
    local_variables
  end

  local_variables # => [:v2]
end

obj = MyClass.new
obj.my_method   # => [:v3]
obj.my_method   # => [:v3]
local_variables # => [:v1, :obj]
```

Chương trình bắt đầu trong scope Top-Level và định nghĩa biến local `v1`. Sau đó chương trình tiếp tục vào scope `MyClass`. Ở đây, chương trình định nghĩa `v1` và `my_method`, nhưng đoạn code trong `my_method` thực sự chưa được thực thi. Kết thúc class, chương trình trở lại với scope Top-Level. Lúc này, chương trình định nghĩa một object của `MyClass` trong biến `obj`. Khi `my_method` được gọi, chương trình sẽ mở ra một scope mới và định nghĩa biến local `v3`, sau khi method kết thúc, biến v3 sẽ biến mất. Điều gì xảy ra khi `my_method` được gọi thêm 1 lần nữa? Lúc đó một scope mới sẽ được mở ra và một biến `v3` mới được định nghĩa và biến mất sau khi kết thúc method. Cuối cùng, chương trình trở về với scope Top-level, nơi đang có 2 biến `v1` và `obj`.

Có thể thấy rằng, khi chương trình thay đổi scope, một số `bindings` sẽ được thay thế bằng một bộ `bindings` mới. Nó không phải thay thế tất cả. Ví dụ instance variables sẽ vẫn tồn tại qua các method được gọi bởi một object. Nói một cách dễ hiểu, local variables sẽ thay đổi khi scope thay đổi.

### Scope Gates

Có 3 nơi mà chương trình sẽ thay đổi scope:
- Định nghĩa Class `class`
- Định nghĩa Module `module`
- Methods `def`

Scope sẽ thay đổi scope khi chương trình đi vào hoặc kết thúc những thành phần trên. Tại mỗi điểm bắt đầu và kết thúc như vậy gọi là một `Scope Gate`.

Có một chút khác biệt là `class` và `module` sẽ thực thi và thay đổi scope ngay lập tức, trong khi `def` chỉ thực thi và thay đổi scope khi bạn gọi đến chúng.

Nhưng bạn sẽ phải làm gì khi muốn pass 1 biến qua những điểm này? Hãy trở lại với Block

## Scope Flat và Shared Scope

```ruby
my_var = "My var"

class MyClass
  # how to print my_var here

  def my_method
    # and here
  end
end
```

Như đã đề cập ở phần Scope, biến local sẽ không thể đi qua Scope Gates. Vậy muốn thực hiện yêu cầu trên thì chúng ta phải định nghĩa class, và method mà không cần Scope Gates.

Nếu nghĩ lại, chúng ta có thể biết rằng MyClass thực chất là một object của class `Class`. Vì vậy chúng ta hoàn toàn có thể định nghĩa MyClass thông qua Class mà không cần Scope Gates. Với method thì chúng ta đã biết đến `Module#define_method`

```ruby
my_var = "My var"

MyClass = Class.new do
  puts "#{my_var} is in class"

  define_method :my_method do
    puts "#{my_var} is in method"
  end
end
# => "My var is in class"

MyClass.new.my_method
# => "My var is in method"
```

Tương tự với class, bạn có thể định nghĩa một module bằng `Module.new`

Nếu bạn replace Scope Gates và cho phép một scope có thể nhìn thấy variables từ những scope khác, kỹ thuật này gọi là "Flattening the scope", hay nói ngắn gọn là **Flat Scope**

Tuy nhiên, đôi lúc bạn chỉ muốn share variables cho một số methods nhất định. Lúc này, hãy định nghĩa những method đó trong một Flat Scope, kỹ thuật này gọi là **Shared Scope**

```ruby
def my_method
  shared = 0

  define_method :counter do
    shared
  end

  define_method :inc do |x|
    shared += x
  end
end

my_method
counter # => 0
inc(2)
counter # => 2
```

## instance_eval và instance_exec

```ruby
class MyClass
  def initialize
    @v = 1
  end
end

obj = MyClass.new
obj.instance_eval do
  self # => #<MyClass:0x0000000000d46700 @v=1>
  @v # => 1
end
```

Block được đánh giá với `receiver` như là `self`. Vì vậy, nó có thể truy cập vào những private methods cũng như instance variables. Thậm chí nếu `instance_eval` thay đổi self, block bạn truyền vào cũng có thể xem những `bindings` từ nơi mà nó được định nghĩa.

Người anh em họ hàng với `instance_eval` là `instance_exec`. Điểm khác nhau là ở chỗ `instance_exec` cho phép truyền params vào block

```ruby
class A
  def initialize
    @x = 1
  end
end

class B
  def my_method
    @y = 2
    A.new.instance_eval {"x = #{@x}, y = #{@y}"}
  end
end

B.new.my_method
# => "x = 1, y = "

# Block truyền vào instance_eval đang trong scope bindings của object A.new nên không thể nhìn thấy @y
# Thay bằng instance_exec, truyền @y vào block
class B
  def my_method
    @y = 2
    A.new.instance_exec(@y) {|y| "x = #{@x}, y = #{y}"}
  end
end

B.new.my_method
# => "x = 1, y = 2"
```

## Callable objects

Sử dụng một block là quy trình gồm 2 bước: Một là viết code vào đó, hai là thực thi code đó. Định nghĩa "Đóng gói code trước, thực thi nó sau" không phải là duy nhất với block. Trong Ruby có ít nhất 3 nơi bạn có thể đóng gói code:
- `proc`, là một block object
- `lambda`, tương tự như proc
- `method`

### Proc objects

Hầu hết mọi thứ trong Ruby đều là object, nhưng block thì không phải. Nhưng tại sao bạn phải quan tâm về điều đó? Hãy tưởng tượng bạn muốn lưu trữ một block và thực thi chúng sau, để làm được điều này bạn phải cần 1 object.

Ruby cung cấp một class là `Proc`, cho phép một block được trở thành một object.
Khai báo một proc với `Proc.new` và thực thi nó với `Proc#call`

```ruby
inc = Proc.new {|x| x + 1}
inc.class # => Proc
inc.call(2) # => 3
```

Ruby cung cấp 2 method để convert block thành Proc là `proc` và `lambda`

```ruby
# use proc
inc = proc {|x| x + 1}

# use lambda
inc = lambda {|x| x + 1}
# hoặc 1 cách viết khác của lambda
inc = ->(x) {x + 1}
```

**Toán tử &**

Một block giống như là một argument vô danh của method. Bạn có thể thực thi block trong method bằng cách sử dụng `yield`. Tuy nhiên, trong 2 trường hợp sau thì `yield` không thể:
- Bạn muốn pass một block của method này vào một method khác (thậm chí là vào block khác)
- Bạn muốn convert block thành một object Proc

Để làm được điều này, đầu tiên bạn cần 1 cái tên cho block. Để đính kèm một `bindings` với block, bạn có thể thêm một argument đặc biệt vào method, nó nằm cuối cùng của list arguments và có thêm tiền tố `&`

```ruby
def math(a, b)
  yield(a, b)
end

def do_math(a, b, &op)
  math(a, b, &op)
end

do_math(2, 3) {|x, y| x * y} # => 6
```

Ở ví dụ trên, toán tử `&` cũng đã đồng thời convert block thành một object Proc.

```ruby
def my_method &pr
  pr
end

a = my_method {"Hello"}
a.class # => Proc
a.call # => "Hello"
```
Bạn có thể convert block thành Proc bằng toán tử `&`, vậy nếu bạn muốn làm ngược lại convert Proc thành block thì làm thế nào? Vẫn là toán tử `&`:

```ruby
def my_method
  "Hello #{yield}"
end

p = Proc.new {"Canh"}
my_method(&p)
# => "Hello Canh"
```

### procs vs lambdas

Chúng ta có nhiều cách để tạo ra một Proc: Proc.new, lambda, toán tử `&`. Tất cả object tạo ra đều là Proc. Tuy nhiên, có một chút khác biệt giữa lambda với phần còn lại - gọi chung là `procs`. Khác biệt tuy ít nhưng khá quan trọng để chúng ta định nghĩa thành 2 loại của Proc là `procs` và `lambdas` (bạn cũng có thể dùng method `Proc#lambda?` để kiểm tra xem Proc object đó có phải là lambda hay không).
Hai điểm khác biệt giữa procs và lambdas là từ khóa `return` và việc check arguments

**procs, lambdas with return keyword**

Trong 1 lambda, `return` sẽ returns từ lambda đó

```ruby
def double(callable_obj)
  callable_obj.call * 2
end

l = lambda {return 10}
double(l) # => 20
```

Trong 1 proc, `return` sẽ returns từ scope mà proc đó được định nghĩa:

```ruby
def another_double
  p = Proc.new {return 10}
  result = p.call
  # Method sẽ được return tại đây, những dòng code sau không thực thi
  puts "Hello"
  return result * 2
end

another_double # => 10
```

Trở lại với hàm `double` ở trên, nếu thay lambda bằng proc

```ruby
p = Proc.new {return 10}
double(p) # => LocalJumpError
```

Chương trình sẽ return từ scope mà proc được định nghĩa, chính là Top-Level. Nhưng bạn không thể return từ scope Top-Level. Để fix lỗi trên bạn có thể remove từ khóa `return` trong proc

**procs, lambdas with arguments**

Điểm khác nhau thứ 2 giữa procs và lambdas chính là cách nhận arguments truyền vào.

procs sẽ không quan tâm tới việc block muốn nhận bao nhiêu arguments

```ruby
p = Proc.new {|a, b| [a, b]}
p.call(1, 2, 3) # => [1, 2]
p.call(1)       # => [1, nil]
```

Trong khi lambdas sẽ raise error khi bạn truyền vào không đúng tham số mà block mong muốn

```ruby
l = lambda {|a, b| [a, b]}
l.call(1, 2, 3) # => ArgumentError
l.call(1)       # => ArgumentError
l.call(1, 2)    # => [1, 2]
```

Một cách thông thường chúng ta có thể thấy lambdas khá giống với method, nghiêm ngặt với tham số truyền vào, exists khi return. Chính vì vậy nếu không có lí do đặc biệt, chúng ta sẽ sử dụng lambdas thay vì procs.

### Method objects

Thành viên cuối cùng trong gia đình callable objects là method objects.
```ruby
class MyClass
  def initialize x
    @x = x
  end

  def my_method
    @x
  end
end

obj = MyClass.new(1)
m = obj.method :my_method
m.call # => 1
````

Bằng cách gọi hàm `Kenel#method` bạn có thể biến method đó thành một object của class Method, và thực thi sau với `Method#call`

Một Method object tương tự như một block hoặc một lambda. Bạn có thể convert một Method thành một proc bằng `Method#to_proc`. Hoặc bạn cũng có thể convert 1 block thành 1 method bằng `define_method`.

Tuy nhiên, có một sự khác biệt quan trọng giữa lambdas và methods đó là: lambdas thì được đánh giá bên trong scope nó định nghĩa (nó là một bao đóng), còn methods thì được đánh giá bên trong object gọi nó.

### Unbound Method

## Domain-Specific Language
