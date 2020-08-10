## OKR_Q3 devise

Công nghệ càng ngày các phát triển, mọi thứ dần dần được số hóa. Khi đó việc sử dụng email(username) và password để thực hiện login theo truyền thống sẽ không còn đủ tính bảo mật nữa.

Vậy làm thế nào để thông tin trên internet của chúng ta được bảo mật hơn.

Two factor authentication (Xác thực 2 bước) hay còn gọi là 2FA ra đời để giải quyết vấn đề đó.

Xác thực 2 bước cung cấp 1 bước bổ sung vào thủ tục đăng nhập thông thường. Việc đăng nhập không còn đơn thuần là sử dụng email và password nữa. Nó sẽ yêu cầu thêm 1 mã bảo mật xác thực để hoàn thành việc đăng nhập đó.

Xác thực 2 bước được áp dụng rộng rãi và cung cấp cho người dùng nhiều hình thức xác thực.
- Rsa securid
- App điện thoại (google authentication, ...)
- SMS
- Email<br>
...

Hình thức phổ biến nhất là mã xác thực được gửi về SMS điện thoại.

Trong VD này thì mình sẽ xác thực 2 bước bằng cách gửi SMS về điện thoại, sử dụng dịch vụ của [Nexmo](https://github.com/nexmo-community/nexmo-rails-devise-2fa-demo)

Để có thể thực hiện được xác thực 2 bước với nexmo thì chúng ta cần đăng ký 1 tài khoản test trên [Nexmo](https://www.nexmo.com/products/verify/)

Với nexmo tài khoản free thì chúng ta chỉ có thể gửi SMS đến số điện thoại đã đăng ký với Nexmo thôi. Và khi nâng cấp lên tài khoản trả phí thì chúng ta mới có thể gửi tới các số điện thoại khác nhau. Những dịch vụ cung cấp xác thực 2 bước bằng SMS này nó đều như thế.
