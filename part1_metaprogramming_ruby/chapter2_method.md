# Dynamic methods và Ghost methods

- Dynamic methods là những methods được định nghĩa động thông qua `define_method`. Chúng là những method tồn tại trong Class
- Trong khi Ghost methods thực chất không tồn tại. Nó override lại `method_missing`. Hoạt động theo nguyên tắc, khi không tìm thấy tôi hãy thực thi nó
