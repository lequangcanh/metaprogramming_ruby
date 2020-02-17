# Điều gì xảy ra khi bạn gọi một method?

Khi bạn gọi một method, Ruby thực hiện 2 việc:
1. Tìm kiếm method được gọi (method lookup)
2. Thực thi method đó

## Method lookup

Trước khi bước vào tìm hiểu Ruby tìm kiếm methods như thế nào, bạn cần phải biết 2 định nghĩa: `receiver` và `ancestors chain`
* `receiver` chính là object mà bạn muốn gọi method. Ví dụ: `my_object.call` thì `my_object` chính là `receiver`.
* Để hiểu khái niệm `ancestors chain`, hãy nhìn vào một class Ruby bất kỳ, sau đó hãy di chuyển từ class đó đến class cha mà nó kế thừa, rồi từ class cha đó lại tiếp tục di chuyển đến class cha tiếp theo, cho đến khi gặp class tổ tiên cuối cùng là `BasicObject`. Con đường mà chúng ta vừa đi qua từng class đó chính là `ancestors chain`. Trên con đường này chúng ta cũng sẽ gặp một số module được include vào, tuy nhiên chúng ta sẽ đề cập về module sau.

Bây giờ bạn đã hiểu về khái niệm `receiver` và `ancestors chain`, chúng ta có thể tổng hợp về quy trình của method lookup trong 1 câu như sau: Để tìm kiếm một method được gọi, Ruby đi vào class của `receiver`, leo qua từng class trong `ancestors chain` cho đến khi tìm được method.

Ví dụ:

```ruby
class MyClass
  def my_method
    print "Hello"
  end
end

class MySubclass < MyClass
end

obj = MySubclass.new
obj.my_method
```

Chúng ta có thể tưởng tượng sơ đồ tìm kiếm `my_method` như sau:

![find_my_object](/images/find_my_method.jpg)

Khi bạn gọi `obj.my_method`, Ruby sẽ đi vào class của receiver (là `obj`) chính là `MySubclass`và không tìm thấy `my_method` ở đây, nó tiếp tục đi đến MyClass (trong ancestors chain) và tìm thấy `my_method`, việc tìm kiếm kết thúc.

MyClass không kế thừa từ class nào khác nên mặc định nó sẽ kế thừa từ class `Object`, và superclass của `Object` là `BasicObject`. Như vậy, trong trường hợp không tìm thấy `my_method` trong `MyClass`, Ruby sẽ tiếp tục đi đến `Object`, và cuối cùng là `BasicObject`, nếu không tìm thấy sẽ trả về `method missing` và kết thúc việc tìm kiếm.

Trong `irb`, bạn có thể gọi hàm `.ancestors` để xem thứ tự của `ancestors chain`

```ruby
MySubclass.ancestors # => [MySubclass, MyClass, Object, Kernel, BasicObject]
```

Khoan đã, Kernel là gì? Sao nó lại xuất hiện ở `ancestors chain`.
Kernel ở đây là một module. Như mình đã nói ở trên "Trên con đường này chúng ta cũng sẽ gặp một số module được include vào"

### Module

Chúng ta đã biết được rằng, ancestors chain sẽ đi từ class đến superclass. Tuy nhiên thực tế, ancestors chain cũng chứa cả module. Khi bạn include một module vào class, Ruby cũng sẽ chèn module này vào ancestors chain, bên phải của class đó.

Như vậy, với ancestors chain đã nêu trên, ta có thể biết rằng module `Kernel` được include vào class `Object`.

Từ Ruby 2.0, ta có 1 cách include module vào class khác là sử dụng `prepend`. Nó hoạt động tương tự `include`, tuy nhiên module sẽ được chèn vào bên trái của class trong ancestors chain. Ta có sơ đồ đơn giản như sau:

![method_lookup_with_module](/images/method_lookup_with_module.jpg)

### Multiple inclusions

Vậy điều gì xảy ra khi bạn cố gắng include một module trong một `ancestors chain` nhiều lần?

Ví dụ:

```ruby
module M1
end

module M2
  include M1
end

module M3
  prepend M1
  include M2
end

M3.ancestors # => [M1, M3, M2]
```
Rất đơn giản, nếu module đã tồn tại trong `ancestors chain`, Ruby sẽ ignore nó, mỗi module chỉ tồn tại trong `ancestors chain` 1 lần duy nhất

## Thực thi method

Sau khi tìm kiếm được method, Ruby sẽ thực thi nó như thế nào?

Giả sử, ta có một method như sau:

```ruby
def my_method
  temp = @x + 1
  my_other_method(temp)
end
```

Để thực thi method này, ta cần làm rõ 2 thứ: biến `@x` `my_other_method` thuộc về đâu?

Câu trả lời chắc khá là dễ dàng với chúng ta, cả 2 giá trị này đều thuộc về object mà chúng ta gọi nó - hay theo định nghĩa ở phía trên thì đó là `receiver`. Vậy Ruby nhận biết receiver này như thế nào?

### self keyword

Mỗi dòng code của Ruby được thực thi bên trong một object. Object đang thực thi đó được gọi là current object hoặc một tên gọi khác là `self`, bạn có thể access current object bằng `self` keyword.

Tại một thời điểm, chỉ có 1 object được đóng vai trò là self. Khi bạn gọi một method, `receiver` sẽ trở thành `self`, tất cả những instance variables và instance variables của self, tất cả những methods không có chỉ định rõ ràng receiver thì là methods của self. Ngay khi bạn gọi một method của một object khác được chỉ định rõ ràng, object đó sẽ trở thành self.

### private method

Private method được quản lý bởi một luật đơn giản: Bạn không thể gọi 1 private method với một receiver rõ ràng, private chỉ được gọi bởi receiver ngầm định - self

```ruby
class A
  def public_method
    self.private_method
  end

  private
  def private_method
  end
end

A.new.public_method
# => NoMethodError: private method 'private method' called ...
```

Remove từ khóa self, đoạn code trên sẽ hoạt động. Bởi vì private method nó chỉ được gọi bằng receiver ngầm định chứ không phải bằng receiver mà bạn chỉ định, dù đó là self.

### The top level

Như bạn đã biết, mỗi lần bạn gọi một method trong 1 object, object đó sẽ trở thành self. Vậy khi bạn không gọi object nào, self sẽ đi đâu về đâu? Mở irb chúng ta có thể dễ dàng có câu trả lời:

```ruby
self       # => main
self.class # => Object
```

Ngay khi bạn start 1 chương trình Ruby, bạn sẽ được đứng trong một object tên là `main` mà Ruby tạo ra cho bạn; hoặc sau khi tất cả method mà bạn gọi đã returned, main sẽ trở thành self.

## Tổng hợp

* Một object chứa các instance variables và một link đến class của object đó.
* Những methods của một object sống ở class của object đó.
* Một class thực chất là một object của class `Class`. Tên của class thực chất chỉ là một hằng số.
* `Class` thì lại là class con của `Module`. Một module là một package các methods cơ bản. Một class cũng có thể được khởi tạo (với `new`) hoặc được sắp xếp trong một hệ thống cấp bậc.
* Các hằng số được sắp xếp trong một cây tương tự cây thư mục của hệ thống file. Tên của module hoặc class tương tự như thư mục, còn hằng số thì tương tự như file trong thư mục đó.
* Mỗi class có một ancestors chain, bắt đầu với class đó và kết thúc là `BasicObject`.
* Khi gọi một method, Ruby sẽ đi qua từng bậc trong ancestors chain của class của `receiver` cho đến khi tìm được method.
* Khi include một module vào một class, module sẽ được insert vào ancestors chain của class đó, ngay phía sau class đó. Khi prepend thì ngược lại module sẽ được insert vào phía trước class đó trong ancestors chain.
* Khi gọi một method, receiver sẽ đóng vai trò là self.
* Khi đang define một module (hoặc class), module (hoặc class) đó sẽ đóng vai trò là self.
* Các instance variables luôn được giả định là instance variables của self.
* Bất kỳ một method nào được gọi mà không được chỉ định receiver rõ ràng thì sẽ được giả định là method của self.
* `Refinements` giống như một đoạn code được chắp vá vào một class, và chúng ghi đè những phương thức tra cứu method thông thường. Nói cách khác, một `Refinement` làm việc trong một phạm vi nhất định của chương trình: những dòng code nằm giữa đoạn bắt đầu với `using` và kết thúc của file, hoặc kết thúc của module definition.
