[![Python 3.6+](https://img.shields.io/badge/python-3.6+-blue.svg)](https://www.python.org/downloads/release/python-360/)

[![SDK-Python3](https://www.liqpay.ua/logo_liqpay.svg?v=1651580791759)](https://www.liqpay.ua/)

* Version: 1.0.0
* Web: https://www.liqpay.ua/
* Download: https://pypi.org/project/aioliqpay
* Source: https://github.com/toxazhl/aioliqpay
* Documentation: https://www.liqpay.ua/documentation/en/
* Keywords: aioliqpay, liqpay, privat24, privatbank, python, internet acquiring, P2P payments, two-step payments, asyncio


What python version is supported?
============
- Python 3.6, 3.7, 3.8, 3.9, 3.10

Get Started
============
1. Sign up in https://www.liqpay.ua/en/authorization.
2. Create a company.
3. In company settings, on API tab, get **Public key** and **Private key**.
4. Done.

Installation
============
From pip
```pip install aioliqpay```

Working with LiqPay Callback locally
============
If you need debugging API Callback on local environment use https://localtunnel.github.io/www/

How it use?
============

Example 1: Basic
-------

**Backend**
Get payment button (html response)
```
liqpay = LiqPay(public_key, private_key)
html = liqpay.cnb_form(
    action='pay',
    amount=1,
    currency='UAH',
    description='description text',
    order_id='order_id_1',
    language='ua'
)
```

Get plain checkout url:

```
liqpay = LiqPay(public_key, private_key)
html = liqpay.checkout_url({
    action='auth',
    amount=1,
    currency='UAH',
    description='description text',
    order_id='order_id_1',
    language='ua',
    recurringbytoken=1'
})
# Response:

str: https://www.liqpay.ua/api/3/checkout/?data=<decoded data>&signature=<decoded signature>

```


**Frontend**

Variable ``html`` will contain next html form

```
    <form method="POST" action="https://www.liqpay.ua/api/3/checkout" accept-charset="utf-8">
        <input type="hidden" name="data" value="eyAidmVyc2lvbiIgOiAzLCAicHVibGljX2tleSIgOiAieW91cl9wdWJsaWNfa2V5IiwgImFjdGlv
        biIgOiAicGF5IiwgImFtb3VudCIgOiAxLCAiY3VycmVuY3kiIDogIlVTRCIsICJkZXNjcmlwdGlv
        biIgOiAiZGVzY3JpcHRpb24gdGV4dCIsICJvcmRlcl9pZCIgOiAib3JkZXJfaWRfMSIgfQ=="/>
        <input type="hidden" name="signature" value="QvJD5u9Fg55PCx/Hdz6lzWtYwcI="/>
        <input type="image"
        src="//static.liqpay.ua/buttons/p1ru.radius.png"/>
    </form>
```

Example 2: Integrate Payment widget to Django
-------
`Payment widget documentation` 
https://www.liqpay.ua/documentation/en/api/aquiring/widget/

*Backend*

`views.py`

```

    from aioliqpay import LiqPay

    from django.views.generic import TemplateView
    from django.shortcuts import render
    from django.http import HttpResponse

    class PayView(TemplateView):
    template_name = 'billing/pay.html'

    def get(self, request, *args, **kwargs):
        liqpay = LiqPay(settings.LIQPAY_PUBLIC_KEY, settings.LIQPAY_PRIVATE_KEY)
        params = {
            'action': 'pay',
            'amount': 100,
            'currency': 'USD',
            'description': 'Payment for clothes',
            'order_id': 'order_id_1',
            'sandbox': 0, # sandbox mode, set to 1 to enable it
            'server_url': 'https://test.com/billing/pay-callback/', # url to callback view
        }
        signature = liqpay.cnb_signature(params)
        data = liqpay.cnb_data(**params)
        return render(request, self.template_name, {'signature': signature, 'data': data})

    @method_decorator(csrf_exempt, name='dispatch')
    class PayCallbackView(View):
        def post(self, request, *args, **kwargs):
            liqpay = LiqPay(settings.LIQPAY_PUBLIC_KEY, settings.LIQPAY_PRIVATE_KEY)
            data = request.POST.get('data')
            signature = request.POST.get('signature')
            sign = liqpay.str_to_sign(settings.LIQPAY_PRIVATE_KEY + data + settings.LIQPAY_PRIVATE_KEY)
            if sign == signature:
                print('callback is valid')
            response = liqpay.decode_data_from_str(data)
            print('callback data', response)
            return HttpResponse()
```
`urls.py`

```

    from django.conf.urls import url

    from billing.views import PayView, PayCallbackView


    urlpatterns = [
        url(r'^pay/$', PayView.as_view(), name='pay_view'),
        url(r'^pay-callback/$', PayCallbackView.as_view(), name='pay_callback'),
    ]
```
*Frontend*

```

    <div id="liqpay_checkout"></div>
    <script>
        window.LiqPayCheckoutCallback = function() {
            LiqPayCheckout.init({
                data: "{{ data }}",
                signature: "{{ signature }}",
                embedTo: "#liqpay_checkout",
                mode: "embed" // embed || popup,
            }).on("liqpay.callback", function(data){
                console.log(data.status);
                console.log(data);
            }).on("liqpay.ready", function(data){
                // ready
            }).on("liqpay.close", function(data){
                // close
            });
        };
    </script>
    <script src="//static.liqpay.ua/libjs/checkout.js" async></script>
```
