## The object model
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
