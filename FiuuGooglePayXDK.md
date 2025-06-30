# 🧾 Google Pay Integration Using Fiuu JS XDK
 - A simplified integration using Fiuu’s hosted JavaScript SDK. Best for merchants who want fast and easy setup with minimal front-end code.

### ✅ Prerequisites

- JavaScript includes:

```html
<script src="https://pay.google.com/gp/p/js/pay.js"></script>
<script src="https://pay.fiuu.com/RMS/GooglePay/xdk_v2.js"></script>
```

### 🧪 PHP Backend Example

```php
$vcode = md5($amount . $merchantID . $orderid . $verifykey);

$paymentData = array(
  "MerchantID" => "TesterID",
  "ReferenceNo" => "TestingOrder123",
  "TxnType" => "SALS",
  "TxnCurrency" => "MYR",
  "TxnAmount" => "1.10",
  "CustName" => "TESTER",
  "CustEmail" => "TESTER@gmail.com",
  "CustContact" => "601532576316",
  "CustDesc" => "Payment for TestingOrder123",
  "Signature" => $vcode,
  "ReturnURL" => "https://xxx.com/return",
  "NotificationURL" => "https://xxx.com/notification",
  "CallbackURL" => "https://xxx.com/callback"
);
```

### 💻 JavaScript Frontend Snippet

```html
<script type="text/javascript">
$(document).ready(function() {
  paymentClient = getGooglePaymentsClient();
  name = "Customer Name";
  origin = "Merchant DBA";
  currency = "MYR";
  amount = "1.10";
  acceptedMethod = '["CC","SHOPEEPAY","TNG-EWALLET"]'; //Include ACCEPTED payment method
  googleMID = "YOUR_GOOGLE_MERCHANT_ID";

  paymentsClient.isReadyToPay(getGoogleIsReadyToPayRequest())
    .then(function(response) {
      if (response.result === true) {
        addGooglePay();
      }
    }).catch(console.error);
});

function addGooglePay() {
  addGooglePayButton();

  $("div#googlepaycheckoutButtonDiv").append(
    "<input type='hidden' id='MerchantID' value='<?= $paymentData['MerchantID'] ?>' />" +
    "<input type='hidden' id='ReferenceNo' value='<?= $paymentData['ReferenceNo'] ?>' />" +
    "<input type='hidden' id='TxnType' value='<?= $paymentData['TxnType'] ?>' />" +
    "<input type='hidden' id='TxnCurrency' value='<?= $paymentData['TxnCurrency'] ?>' />" +
    "<input type='hidden' id='TxnAmount' value='<?= $paymentData['TxnAmount'] ?>' />" +
    "<input type='hidden' id='Signature' value='<?= $paymentData['Signature'] ?>' />" +
    "<input type='hidden' id='CustName' value='<?= $paymentData['CustName'] ?>' />" +
    "<input type='hidden' id='CustEmail' value='<?= $paymentData['CustEmail'] ?>' />" +
    "<input type='hidden' id='CustContact' value='<?= $paymentData['CustContact'] ?>' />" +
    "<input type='hidden' id='CustDesc' value='<?= $paymentData['CustDesc'] ?>' />" +
    "<input type='hidden' id='ReturnURL' value='<?= $paymentData['ReturnURL'] ?>' />" +
    "<input type='hidden' id='NotificationURL' value='<?= $paymentData['NotificationURL'] ?>' />" +
    "<input type='hidden' id='CallbackURL' value='<?= $paymentData['CallbackURL'] ?>' />"
  );
}
</script>

<div class='row' align='center' id='googlepaycheckoutButtonDiv'></div>
```

---

## 📞 Support

- [Fiuu Developer Documentation Direct server API](https://github.com/FiuuPayment/Documentation-Fiuu_API_Spec/blob/main/%5BOfficial%5D%20Fiuu%20Direct%20Server%20API%20v1.7.7.pdf)
- [Google Pay for Web Guide](https://developers.google.com/pay/api/web)
