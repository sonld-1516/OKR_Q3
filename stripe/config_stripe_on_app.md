## Config stripe trong ứng dụng để có thể thanh toán được

- Trong bài viết này sẽ thực hiện các bước config trong ứng dụng Ruby on Rails.
- Ở những ngôn ngữ khác, các bước config cũng tiến hành tương tự

> Add gem stripe vào gem file</br>
gem "stripe"

> Thêm link stripe checkout js vào header script layout hoặc tải về bỏ vào thư mục js </br>
[stripe checkout js](https://checkout.stripe.com/v3/checkout.js)

> Lấy stripe public key và secret key từ dashboard stripe bỏ vào env để sử dụng<br>
![api key](/images/stripe/api_key.png)


### Tạo một view list các sản phẩm để thực hiện thanh toán thử bằng stripe
![Product](/images/stripe/product.png)

Khi click payment sẽ hiển thị lên popup nhập thông tin credit card. Giờ ta sẽ config popup payment. Stripe có hỗ trợ sẵn giao diện popup payment cho mình rồi. Sử dụng file checkout.js ở phía trên

![Config button payment](/images/stripe/config_p.png)

- Function stripe_checkout config các thông tin để gửi lên server sau khi người dùng nhấn submit ở
- Trong event click button payment gọi lại hàm config stripe và dùng nó để mở popup nhập thông tin thanh toán <br>
  - *name*: sẽ hiển thị tên sản phẩm cần thanh toán
  - *description*: hiển thị mô tả của sản phầm cần thanh toán
  - *amount*: hiển thị giá của sản phẩm cần thanh toán. <br>
  Vì sao amount lại phải * 100. Chỗ này mình ko biết stripe nó xử lý thế nào, nhưng mình không * 100 thì khi lên stripe số tiền cần thanh toán sẽ /100

  VD: số tiền cần thanh toán là 150$
  - Trong field amount nhập 150 thì trên popup thanh toán sẽ là 1.5$

Sau khi config event cho button payment thì khi click vào sẽ hiển thị lên popup thanh toán như thế này
![Popup payment](/images/stripe/popup_payment.png)

Người dùng sẽ nhập email, và thông tin thẻ master card, visa (thẻ quốc tế) vào để tiến hành thanh toán<br>
**Đến đây thì xong bước config dưới client. Tiếp theo sẽ xử lý trên server sau khi người dùng click pay trên popup payment**


### Các bước xử lý trên server
#### Gửi thông tin thanh toán lên server stripe và nhận về kết quả

Ở controller ta sẽ gửi thông thẻ lên stripe để tiến hành thanh toán

Trước khi gửi lên stripe thì ta sẽ tạo invoice để phục vụ cho việc quản lý thanh toán và thực hiện thanh toán lại sau này nếu quá trình thanh toán trên stripe thất bại

```
class PaymentController < ApplicationController
  before_action :load_product, :load_invoice, only: :create

  def create
    Stripe.api_key = ENV["STRIPE_SECRET_KEY"]
    token = params[:token]

    begin
      if @invoice
        @invoice.no_invoice_due!
      else
        @invoice = @product.invoices.create(
          price: (@product.price * 100),
          status: :no_invoice_due,
          name: @product.name
        )
      end

      charge = Stripe::Charge.create(
        :amount => @product.price.to_i * 100,
        :currency => "usd",
        :source => token,
        :description => "Payment for #{@product.name}",
        :metadata => {
          invoice_id: @invoice.id
        }
      )

      redirect_to payments_success_path
    rescue Stripe::CardError => e
      @product.invoices.create(
        price: (@product.price * 100),
        status: :cancel
      )
      redirect_to payments_cancel_path
    end
  end

  private
  def load_product
    @product = Product.find_by id: params[:product_id]
  end

  def load_invoice
    @invoice = Invoice.find_by id: params[:invoice_id]
  end
end
```
Sau khi gửi thành công lên stripe ta sẽ chuyển hướng sang trang thông báo thanh toán thành công

![Payment success](/images/stripe/payment_succes.png)

Hoặc quá trình gửi bị lỗi thì chuyển hướng sang trang thất bại

![Payment fail](/images/stripe/payment_cancel.png)

Đến đây thì cơ bản là đã hoàn thành. Nhưng để xác nhận thanh toán trên stripe đã thực sự thành công hay chưa thì ta phải config 1 webhook để lắng nghe sự kiện thanh toán thành công trên stripe để tiến hành cập nhập invoice ở server của mình

#### Tạo webhook để xác nhận thanh toán thành công
##### Tạo contronller để lắng nghe khi thanh toán thực sự thành công trên stripe

```
class StripeIpnController < ApplicationController
  protect_from_forgery except: :create
  skip_before_action :verify_authenticity_token, only: :create
  before_action :authenticate_user!, except: :create


  def create
    event = Stripe::Event.retrieve params[:id]

    case
    when event.type == "charge.refunded"
      ipn_refund event
    when event.type == "charge.succeeded"
      ipn_charge event
    when event.type == "invoice.payment_succeeded"
      ipn_subscription event
    end

    render nothing: true
  end

  private
  def ipn_charge event
    unless event.data.object.metadata.to_h.blank?
      invoice = Invoice.find_by id: event.data.object.metadata.invoice_id
      if event.data.object.paid
        # Xác nhận thanh toán thành công
        # create invoice
        invoice.update_attributes charge_id: event.data.object.id, status: :paid
      else
        # Xác nhận thanh thoán không thành công
        # Ta gửi email thông báo cho user
        invoice.update_attributes charge_id: event.data.object.id, status: :unpaid
      end
    end
  end

  def ipn_refund event
    unless event.data.object.metadata.to_h.blank?
      invoice = Invoice.find_by id: event.data.object.metadata.invoice_id
      if event.data.object.paid
        invoice.update_attributes status: :refuned, reason: "Requested by Customer"
      else
        invoice.update_attributes reason: "Refund fails. Please try again", status: :paid
      end
    end
  end

  def ipn_subscription event
    invoice = Invoice.find_by id: event.data.object.lines.data.first.metadata.invoice_id
    if event.data.object.paid
      invoice.update_attributes status: :paid, charge_id: event.data.object.id
    else
      invoice.update_attributes status: :unpaid, charge_id: event.data.object.id
    end
  end
end
```
Ở đây mình có tạo sẵn các hàm để lắng nghe sự kiện khi
- Stripe thực hiện thanh toán thực sự thành công
- Stripe thực hiện `refund` 1 khoản thanh toán thành công
- Stripe thực hiện thanh toán định kỳ thành công

<br>
Ở những hàm này thì ta sẽ không trả về gì cả, chỉ tiến hành update invoice trên hệ thống thành `paid` để xác nhận thanh toán thành công, hoặc `unpaid` để xác nhận thanh toán thất bại

##### Config webhook trên stripe dashboard
![Webhook](/images/stripe/webhook.png)
- Nhập vào url enpoit lắng nghe sự kiện của server
- Chọn các sự kiện sẽ lắng nghe là
  - invoice.payment_succeeded (sự kiện thanh toán định kỳ thành công - subscription)
  - charge.refunded ( sự kiện hoàn trả thanh toán thành công)
  - charge.succeeded ( sự kiện thanh toán thành công )

**Note**<br>
Stripe không thể gửi callback về link localhost được. Muốn test được callback webhook ở dưới localhost thì
- Sử dụng ngrok để tạo link ảo gán cho webhook stripe
- Sử dụng thư viện `stripe-cli` cài đặt dưới local để test function trong controller callback
([Stripe CLI](https://stripe.com/docs/stripe-cli))



**Đến đây thì thanh toán đã thực sự thành công**

#### Thanh toán định kỳ với stripe (Subscription)

Cũng tương tự như thanh toán riêng lẻ từng sản phẩm phía trên.<br>
Subscription có các dạng thanh toán
- day (theo ngày)
- week ( theo tuần)
- month (theo tháng)
- quarter ( 3 tháng)
- semiannual ( 6 tháng)
- year ( theo năm)

Ở trong ví dụ này thì mình sẽ tiến hành thanh toán theo `month` và `year`

##### Tạo product và plan trên stripe
Ngoài ra trước khi cho phép thanh toán định kỳ thì ta cần phải tại ra các plan trên stripe để lựa chọn thanh toán. Có 2 cách để tạo ra được plan

- Cách 1: Tạo bằng API
Ta sẽ tạo seed data hoặc rake task để gửi request tạo lên stripe

**Bước 1: Tạo product**
```
product = Stripe::Product.create(
  name: "Basic Plan",
  type: "service"
)
```

**Bước 2: Tạo plan**
```
plan = Stripe::Plan.create(
  amount: 30000,
  interval: "month",
  product: product.id,
  currency: "usd",
  id: "basic_plan_3"
)
```

**Bước 3: Lưu thông tin plan vào database**
```
Plan.create(name: product.name, plan_stripe_id: plan.id, price: plan.amount.to_f / 100, interval: "month")
```


- Cách 2: Tạo trên màn hình dashboard

![Plan](/images/stripe/plan.png)

Nhập các thông tin tên plan, giá sẽ thanh toán, loại thanh toán định kỳ là month, year ...

Tạo xong trên dashboard lấy thông tin plan_id lưu về dưới database của mình để sử dụng khi tiến hành thanh toán
![Plan ID](/images/stripe/plan_id.png)

##### Tiến hành thanh toán định kỳ

Tạo view hiển thị plan cho user chọn để thanh toán subscription
![Plan view](/images/stripe/plan_view.png)
Các bước config popup payment ở client tương tự như ở đây
[Config checkout stripe client](config_stripe_on_app.md#tạo-một-view-list-các-sản-phẩm-để-thực-hiện-thanh-toán-thử-bằng-stripe)

**Tạo controller để gửi thông tin thanh toán định kỳ**
**Bước 1: Tạo customer cho mỗi thanh toán subscription**
```
Stripe.api_key = ENV["STRIPE_SECRET_KEY"]
token = params[:token]
info = Stripe::Token.retrieve(token)

customer = Stripe::Customer.create(
  source: token,
  email: info.email
)
```
Có thông tin customer trên stripe thì stripe mới có thể định kỳ thanh toán cho customer đó được

**Bước 2: Tạo invoice cho thanh toán**
```
@invoice = @plan.invoices.create price: @plan.price, status: :no_invoice_due, name: @plan.name
```
**Bước 3: Tiến hành gửi yêu cầu thanh toán định kỳ lên stripe**
```
Stripe::Subscription.create(
  :customer => customer.id,
  :metadata => {
    invoice_id: @invoice.id
  },
  :items => [
    { plan: @plan.plan_stripe_id }
  ]
)
```
- customer ==> là tạo subsription cho customer nào trên stripe
- metadata ==> Thông tin thêm sẽ được webhook gửi về kèm khi xác nhận thanh toán
- items ==> Chọn plan muốn tạo thanh toán định kỳ

Sau khi gửi thông tin lên stripe thành công thì ta chuyển về trang thông báo subscription thành công, hoặc trang thông báo thất bại

Còn xác nhận cụ thể thanh toán subscription thành công hay chưa, ta tiến hành khi webhook gửi sự kiện về
[Webhook IPN subscription](config_stripe_on_app.md#tạo-webhook-để-xác-nhận-thanh-toán-thành-công). Trong hàm callback webhook thì sẽ tiến hành tạo mới hoặc update invoice cho những thanh toán định kỳ

**Tới đây là chúng ta đã tạo và xử lý thành công thanh toán định kỳ stripe**

##### Refund trong Stripe
Để hoàn trả 1 khoản thanh toán trên stripe thì cũng có 2 cách
- Cách 1: Refund bằng API

Để refund bằng API thì chúng ta dựa vào invoice của mỗi lần thanh toán trên website của mình
![Invoice](/images/stripe/invoice.png)
- Những thanh toán thực sự thành công sẽ có status `paid` thì có thể tiến hành `refund`
- Những thanh toán không thành công thì có thể tiến hành thanh toán lại `Re payment`<br>
(Repayment cũng giống như 1 thanh toán mới. Sẽ tiến hành gửi thông tin thanh toán lên stripe và xác nhận giống như bình thường thôi)

```
class RefundsController < ApplicationController
  before_action :load_invoice, only: :create

  def create
    Stripe.api_key = ENV["STRIPE_SECRET_KEY"]

    refund = Stripe::Refund.create({
      charge: @invoice.charge_id
    })

    if refund.status = "succeeded" && refund.object == "refund"
      @invoice.refuning!
      flash.now[:success] = "Refund payment is processing"
    else
      flash.now[:success] = "Refund payment is failed"
    end
  rescue
    flash.now[:success] = "Refund payment is failed"
  end

  private
  def load_invoice
    @invoice = Invoice.find_by id: params[:invoice_id]
  end
end
```
Chúng ta gọi API `Stripe::Refund.create` để gửi yêu cầu refund lên stripe. Và xác nhận Refund thực sự thành công qua callback  tại đây [Webhook Refund](config_stripe_on_app.md#tạo-contronller-để-lắng-nghe-khi-thanh-toán-thực-sự-thành-công-trên-stripe) và update invoice sang trạng thái refund.


- Cách 2: Refund trên màn hình dashboard

![List Payment](/images/stripe/list_payment.png)
Muốn refund thanh toán nào thì ta click vào thanh toán đó, để chuyển qua trang chi tiết thanh toán và click button refund thôi
![Refund](/images/stripe/refund.png)
