## Stripe là gì? https://stripe.com/
Stripe là một cổng thanh toán của M, cho phép các trang thương mại điện tử nhận thanh toán trên website của mình. Nó là nền tảng phần mềm tốt nhất cho hoạt động kinh doanh trên internet. Phần mềm này đang thực hiện xử lý hàng tỷ đô la mỗi năm cho các doanh nghiệp trên khắp thế giới. Stripe cung cấp SDK để có thể tích hợp trên các thiết bị chạy hệ điều hành Android và IOS. Ngoài ra, nó còn cung cấp Stripe API để có thể được sử dụng bởi rất nhiều các ngôn ngữ như: Ruby, Python, Java, GO… (Stripe API).

### Tài khoản stripe
Tài khoản Stripe rất giống tài khoản ngân hàng. Bạn có thể tạo nhiều tài khoản Stripe cho sản phẩm hoặc các trang bán hàng của mình. Mỗi tài khoản có các khoản thanh toán, bảng điều khiển và người dùng được ủy quyền riêng mà bạn lựa chọn.

Tài khoản có loại tiền mặc định. Khi bạn nhận thanh toán từ các loại tiền tệ khác nó sẽ tự quy đổi về đơn vị tiền tệ mặc định. Bạn có thể chuyển tiền từ tài khoản Stripe về tài khoản ngân hàng của bạn


### Cơ chế hoạt động
1: Web gửi thông tin thẻ lên Stripe <br>
2: Stripe xử lý và trả về token<br>
3: Server giao tiếp với Stripe thông qua Stripe API <br>
4: Khi giao dịch xong thì Stripe trả kết quả về cho Server<br>
5: Server hiển thị kết quả giao dịch

## Tạo tài khoản Stripe
### [1: Đăng ký tài khoản Stripe](https://dashboard.stripe.com/register)
![Đăng ký tài khoản Stripe](/images/stripe/register.png "Đăng ký tài khoản Stripe")

Sau khi đăng ký xong tài khoản Stripe bạn cần confirm tài khoản thông qua email

### 2: Làm quen giao diện dashboard Stripe
![Dashboard](/images/stripe/dashboard.png "Dash board")

Ở thanh menu bên tay trái, đó là các tính năng mà Stripe cung cấp để chúng ta có thể quản lý được các giao dịch, người dùng, và tài khoản của chúng ta. Payments : Bao gồm các danh sách giao dịch của người dùng qua tài khoản Stripe của chúng ta. API: Bao gồm các thông tin như version, Secret key và Publishable key...

![API KEY](/images/stripe/api_key.png "API Key")

### 3: Publishable key và Secret key là gì?
#### Publishable key
Đây là key mà Stripe cung cấp khi sử dụng Stripe SDK. Nó được sử dụng để tạo ra được EphemeralKey dùng cho việc thêm card.

#### Secret key
Như cái tên của nó, nó là key mà chúng ta cẩn phải giữ kín. Với key này, chúng ta có thể thực hiện các giao dịch như thanh toán, xoá lịch sử giao dịch, refund...


### 4: Tạo customer và thanh toán
Trong môi trường test thì chúng ta phải tạo ra customer và thông tin của customer dành cho việc test chức năng thanh toán

![Customer](/images/stripe/customer.png "Customer")

Nhập thông tin để tạo customer
![Tạo customer](/images/stripe/customer_1.png "Create customer")

Sau đó vào chi tiết customer để thiết lập các thông tin để có thể thanh toán được

#### Thêm phương thức thanh toán
Tìm đến Payment Method --> Add Card.
Đây là tạo ra 1 thẻ master card ảo dùng để test thanh toán
![Card](/images/stripe/c.png "Card")

#### Customer balance
Tạo cho customer 1 số tiền dùng để test thanh toán.<br>
Tìm đến Customer Balance --> Add balance adjustment
![Balance](/images/stripe/balance.png "Customer balance")

##### Đến đây chúng ta đã tạo customer trên màn hình dashboard stripe dùng để thanh toán
