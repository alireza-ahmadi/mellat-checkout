# Mellat Ckeckout:

[![npm version](https://badge.fury.io/js/mellat-checkout.svg)](https://badge.fury.io/js/mellat-checkout)

Unofficial [Behpardakht Mellat Gateway](http://www.behpardakht.com/resources/Vpos.html) implementation in Node.JS

NOTICE: SUBJECT TO BACKWARD INCOMPATIBLE CHANGES (Due to under V1.0.0 version).

- [Installation](#installation)
  - [NPM](#npm)
  - [Yarn](#yarn)

- [Usage](#usage)
  - [Create An Instance](#create-an-instance)
  - [API](#api)
  - [Callbacks Instead Of Promises](#callbacks-instead-of-promises)

- [TODO](#todo)
  - [API](#api)
  - [Helper Methods](#helper-methods)
  - [Code](#code)

- [Contributing](#contributing)
- [ResCodes Descriptions](#rescodes-descriptions)

## Installation

Install the package from `npm` or `yarn`.
### NPM

```bash
npm install mellat-checkout
```
### Yarn

```bash
yarn add mellat-checkout
```
## Usage:

### Create An Instance

Import the package:
```javascript
const MellatCheckout = require('mellat-checkout');
// or (ES6):
import MellatCheckout from 'mellat-checkout';
```
Then create an instance:
```javascript
const mellat = new MellatCheckout({
  terminalId: 'xxxxxxx',
  username: 'xxxxxxx',
  password: 'xxxxxxx',
  timeout: 10000, // Optional, number in millisecond (defaults to 10 sec)
  apiUrl: 'https://bpm.shaparak.ir/pgwchannel/services/pgw?wsdl', // Optional, exists (and may updated) in bank documentation (defaults to this)
});

// Initialize the client, this step is optional
// but gives you more control over your flow
// and speeds up the first (and just first) request.
mellat.initialize().then(function () {
  console.log("Mellat client ready")
})
.catch(function (error) => {
  // you can retry here
  console.log("Mellat client encountered error:", error)
});
```

### API

#### Payment Request:

```javascript
mellat.paymentRequest({
  amount: 1000, // Payment Amount In Rials
  orderId: '12345678912', // OrderID Generated By You
  callbackUrl: 'https://call.back/mellat', // Payment Callback URL
  payerId: '0' // Optional
}).then(function (response) {
  if (response.resCode === 0) {
    console.log(response.refId);
    // Now redirect user to following address with post param
    // { RefId: response.refId }
    // https://bpm.shaparak.ir/pgwchannel/startpay.mellat
  } else {
    console.warn('Gateway Error: ', response.resCode);
  }
}).catch(function (error) {
  console.error(error);
});
```

#### Payment Verification:

```javascript
mellat.verifyPayment({
  orderId: '12345678912', // OrderID Used In Payment Request
  saleOrderId: '12345678912', // Get From Payment Callback Post Params
  saleReferenceId: '5142510', // Get From Payment Callback Post Params
}).then(function (response) {
  if (response.resCode === 0) {
    console.log("Verified, Call settlePayment");
  } else {
    console.warn('Gateway Error: ', response.resCode);
  }
}).catch(function (error) {
  console.error(error);
});
```

#### Payment Settlement (Payment Finalization):

```javascript
mellat.settlePayment({
  orderId: '12345678912', // OrderID Used In Payment Request
  saleOrderId: '12345678912', // Get From Payment Callback Post Params
  saleReferenceId: '5142510', // Get From Payment Callback Post Params
}).then(function (response) {
  if (response.resCode === 0) {
    console.log("Payment Is Done.");
  } else if (response.resCode === 45) {
    console.log("Payment Already Done(Settled Before).");    
  } else {
    console.warn('Gateway Error: ', response.resCode);
  }
}).catch(function (error) {
  console.error(error);
});
```

#### Inquiry Request:

```javascript
mellat.inquiryRequest({
  orderId: '12345678912', // OrderID Used In Payment Request
  saleOrderId: '12345678912', // Get From Payment Callback Post Params
  saleReferenceId: '5142510', // Get From Payment Callback Post Params
}).then(function (response) {
  console.log('Payment Status: ' + response.resCode)
}).catch(function (error) {
  console.error(error);
});
```

#### Reversal Request:

```javascript
mellat.reversalRequest({
  orderId: '12345678912', // OrderID Used In Payment Request
  saleOrderId: '12345678912', // Get From Payment Callback Post Params
  saleReferenceId: '5142510', // Get From Payment Callback Post Params
}).then(function (response) {
  if (response.resCode === 0) {
    console.log("Payment Is Reversed.");
  } else {
    console.warn('Gateway Error: ', response.resCode);
  }
}).catch(function (error) {
  console.error(error);
});
```

### Callbacks Instead Of Promises

All methods can be called by using Callbacks instead of Promises, lets take `paymentRequest` as an example:

```javascript
mellat.paymentRequest({
  amount: 1000, // Payment Amount In Rials
  orderId: '12345678912', // OrderID Generated By You
  callbackUrl: 'https://call.back/mellat', // Payment Callback URL
  payerId: '0' // Optional
}, function (error, response) {
  if (error) {
    console.error(error);
  } else if (response.resCode === 0) {
    console.log(response.refId);
  } else {
    console.warn('Gateway Error: ', response.resCode);
  }
});
```

## TODOS

### API-TODO

- [x] bpPayRequest
- [x] bpVerifyRequest
- [x] bpSettleRequest
- [x] bpInquiryRequest
- [x] bpReversalRequest
- [ ] bpDynamicPayRequest
- [ ] bpCumulativeDynamicPayRequest

### Helper Methods

- [ ] verifyAndSettle (Verify the payment and settle/reverse it)

### Code

- [ ] JSDocs and code comments
- [ ] Unit tests using `mocha`

## Contributing

Contributions are welcome. Please submit PRs or just file an Issue if you see something broken or in need of improving.

## ResCodes Descriptions
| ResCode | Description                                   |
| :-----: | :-------------------------------------------: |
| 0       | تراکنش با موفقیت انجام شد                     |
| 11      | شماره کارت نا معتبر است                       |
| 12      | موجودی کافی نیست                              |
| 13      | رمز نادرست است                                |
| 14      | تعداد دفعات وارد کردن رمز بیش از حد مجاز است  |
| 15      | کارت نامعتبر است                              |
| 16      | دفعات برداشت وجه بیش از حد مجاز است           |
| 17      | کاربر از انجام تراکنش منصرف شده است           |
| 18      | تاریخ انقضای کارت گذشته است                   |
| 19      | مبلغ برداشت وجه بیش از حد مجاز است            |
| 21      | پذیرنده نا معتبر است                          |
| 23      | خطای امنیتی رخ داده است                       |
| 24      | اطلاعات کاربری پذیرنده نا معتبر است            |
| 25      | مبلغ نا معتبر است                             |
| 31      | پاسخ نا معتبر است                             |
| 32      | فرمت اطلاعات وارد شده صحیح نمی باشد            |
| 33      | حساب نا معتبر است                             |
| 34      | خطای سیستمی                                   |
| 35      | تاریخ نا معتبر است                            |
| 41      | شماره درخواست تکراری است                      |
| 42      | تراکنش Sale یافت نشد                          |
| 43      | قبلا درخواست Verify داده شده است               |
| 44      | درخواست Verify یافت نشد                       |
| 45      | تراکنش Settle شده است                         |
| 46      | تراکنش Settle نشده است                        |
| 47      | تراکنش Settle یافت نشد                        |
| 48      | تراکنش Reverse شده است                        |
| 49      | تراکنش Refund یافت نشد                        |
| 51      | تراکنش تکراری است                             |
| 54      | تراکنش مرجع موجود نیست                        |
| 55      | تراکنش نا معتبر است                           |
| 61      | خطا در واریز                                  |
| 111     | صادر کننده کارت نا معتبر است                  |
| 112     | خطای سوییچ صادر کننده کارت                    |
| 113     | پاسخی از صادر کننده ی کارت دریافت نشد         |
| 114     | دارنده کارت مجاز به انجام این تراکنش نیست     |
| 412     | شناسه قبض نادرست است                          |
| 413     | شناسه پرداخت نادرست است                       |
| 414     | سازمان صادر کننده قبض نامعتبر است             |
| 415     | زمان جلسه کاری به پایان رسیده است             |
| 416     | خطا در ثبت اطلاعات                             |
| 417     | شناسه پرداخت کننده نا معتبر است               |
| 418     | اشکال در تعریف اطلاعات مشتری                   |
| 419     | تعداد دفعات ورود اطلاعات از حد مجاز گذشته است  |
| 421     | IP نامعتبر است                                |
