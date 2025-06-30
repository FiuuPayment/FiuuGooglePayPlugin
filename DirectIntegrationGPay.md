# 🧾 Google Pay Web Integration via Fiuu (RMS)

This integration enables Google Pay on your website using [Fiuu](https://pay.fiuu.com)'s payment gateway. It allows you to:

- Generate a transaction session with Fiuu
- Launch Google Pay UI
- Submit the token securely to Fiuu for processing
- Handle timeouts, failures, and user cancellations

---

## 🚀 Getting Started

### ✅ Prerequisites

- jQuery 3.6+
- Google Pay SDK (`pay.js`)
- Fiuu Merchant Account with Google Pay enabled
- Ensure that the Google Pay integration has already been completed. The sample code below serves as an enhancement to connect to FIUU for processing Google Pay transactions

---

## 📦 Script Includes

```html
<script async src="https://pay.google.com/gp/p/js/pay.js"></script>
<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
```

---


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

### 💻 JavaScript Frontend Snippet + Global Variables

```html
<script type="text/javascript">
$(document).ready(function() {
  paymentClient = getGooglePaymentsClient();

  const name = "Merchant DBA";
  const currency = "MYR";
  const amount = "1.10";
  const domain_env = "https://pay.fiuu.com";
  const googleMID = 'YOUR_GOOGLE_MERCHANT_ID';
  const name = 'YOUR_MERCHANT_NAME';

  let tranID = '';
  let duitnowQR = 'false';
  let acceptedMethod = '["CC","SHOPEEPAY","TNG-EWALLET"]'; //Include ACCEPTED payment method

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

## 🧩 Full Google Pay + Fiuu Integration Code (with Comments)

```javascript
// Step 1: Triggered when Google Pay button is clicked
function onGooglePaymentButtonClicked() {
  // Collect input values from DOM
  var merchantRequest = {};
  $('input', 'div#googlepaycheckoutButtonDiv').each(function () {
    merchantRequest[$(this).attr('id')] = $(this).val().trim();
  });
  merchantRequest['paymentMethods'] = acceptedMethod;

  // Create transaction on Fiuu server
  $.ajax({
    type: 'POST',
    cache: false,
    dataType: 'json',
    data: merchantRequest,
    url: domain_env + '/RMS/GooglePay/createTxn.php'
  }).done(function (result) {
    tranID = result.tranID;

    // If transaction creation is successful
    if (result.return_code == "success") {
      const paymentDataRequest = getGooglePaymentDataRequest();

      if (result.useDuitNow === true) {
        duitnowQR = 'true';
      }

      paymentDataRequest.allowedPaymentMethods = result.responseData;
      paymentDataRequest.merchantInfo = {
        merchantId: googleMID,
        merchantName: name,
      };
      paymentDataRequest.transactionInfo = getGoogleTransactionInfo();

      const paymentsClient = getGooglePaymentsClient();

      // Open Google Pay UI
      paymentsClient.loadPaymentData(paymentDataRequest)
        .then(function (paymentData) {
          processPayment(paymentData); // Handle success
        })
        .catch(function (err) {
          console.error(err);
          if (err.statusCode === "CANCELED") {
            handleCancel(tranID); // User cancelled
          }
        });

    } else {
      // Handle error returned from Fiuu
      alert(result.message);
      handleCancel(tranID, result.message);
    }
  }).fail(function () {
    alert('Payment could not process. Kindly retry again.');
  });
}

// Step 2: Handle the token received from Google Pay
function processPayment(paymentData) {
  var merchantRequest = {};
  $('input', 'div#googlepaycheckoutButtonDiv').each(function () {
    merchantRequest[$(this).attr('id')] = $(this).val().trim();
  });

  // Handle timeout countdown based on expiration time
  let time = 60;
  if (merchantRequest.ExpirationTime) {
    const utcDateTime = new Date(merchantRequest.ExpirationTime);
    const offset = 8 * 60 * 60 * 1000;
    const targetTime = new Date(utcDateTime.getTime() + offset);
    const now = new Date(new Date().getTime() + offset);
    if (targetTime >= now) {
      time = Math.floor((targetTime - now) / 1000);
    }
  }

  // Attach token and prepare request
  merchantRequest.GooglePay = paymentData;
  merchantRequest.GooglePay.internalVersion = 2;
  merchantRequest.tranID = tranID;
  if (duitnowQR === 'true') merchantRequest.duitnowQR = duitnowQR;
  merchantRequest.requery = 0;

  // Show countdown UI
  startTimer(time);

  const startTime = Date.now();
  makePayment(merchantRequest, paymentData, startTime, time * 1000);
}

// Countdown logic for displaying time remaining
function startTimer(duration) {
  let timer = duration;
  const interval = setInterval(() => {
    let minutes = String(parseInt(timer / 60)).padStart(2, '0');
    let seconds = String(parseInt(timer % 60)).padStart(2, '0');
    $('#countdown').text(`${minutes}:${seconds}`);
    if (--timer < 0) clearInterval(interval);
  }, 1000);
}

// Step 3: Final payment processing with Fiuu
function makePayment(merchantReq, paymentData, startTime, duration) {
  const elapsed = Date.now() - startTime;
  if (duration - elapsed < 5000) {
    return handleCancel(tranID);
  }

  $.ajax({
    type: 'POST',
    cache: false,
    data: merchantReq,
    url: domain_env + '/RMS/GooglePay/payment_v2.php'
  }).done(function (result) {
    if (paymentData.paymentMethodData.type === "CARD") {
      result = JSON.parse(result);
    }

    if (result['TxnData']) {
      submitRedirectForm(result['TxnData']);
    } else if (result['status'] === 'failed') {
      handleCancel(tranID);
    } else if (result['status'] === 'waiting') {
      merchantReq.requery = 1;
      makePayment(merchantReq, paymentData, startTime, duration);
    } else if (result['status'] === 'success') {
      submitRedirectForm(result['TxnData']);
    }
  }).fail(function () {
    alert('Payment could not process. Kindly retry again.');
  });
}

// Cancel or rollback logic on failure
function handleCancel(tranID, errorDesc = '') {
  var merchantRequest = {};
  $('input', 'div#googlepaycheckoutButtonDiv').each(function () {
    merchantRequest[$(this).attr('id')] = $(this).val().trim();
  });
  merchantRequest.tranID = tranID;
  if (errorDesc) merchantRequest.errorDesc = errorDesc;

  $.ajax({
    type: 'POST',
    cache: false,
    data: merchantRequest,
    dataType: 'json',
    url: domain_env + '/RMS/GooglePay/cancel.php'
  }).done(function (result) {
    if (result['status'] === 'success') {
      submitRedirectForm(result['TxnData']);
    }
  }).fail(function () {
    alert('Payment could not process. Kindly retry again.');
  });
}

// Auto-submit redirect form to Fiuu
function submitRedirectForm(txnData) {
  var form = document.createElement("form");
  form.setAttribute("method", txnData['RequestMethod']);
  form.setAttribute("action", txnData['RequestURL']);

  if (txnData['RequestData']) {
    for (let key in txnData['RequestData']) {
      let input = document.createElement('input');
      input.type = 'hidden';
      input.name = key;
      input.value = txnData['RequestData'][key];
      form.appendChild(input);
    }
  }

  document.body.appendChild(form);
  form.submit();
}
```

## 💳 Flow Summary

### Step 1: Click Button
- When user clicks the Google Pay button, `onGooglePaymentButtonClicked()` is triggered.
- Collects all `<input>` values inside `#googlepaycheckoutButtonDiv`
- Sends them to Fiuu at `/RMS/GooglePay/createTxn.php`
- Receives a transaction ID and `allowedPaymentMethods` from Fiuu

### Step 2: Initiate Google Pay Sheet
- If Fiuu returns success:
  - The `PaymentsClient` is instantiated
  - Google Pay sheet is shown to the user via `loadPaymentData()`

### Step 3: User Confirms Payment
- Google returns a token (`paymentData`) which is processed in `processPayment(paymentData)`:
  - Timeout logic is calculated from optional `ExpirationTime`
  - A countdown UI is shown
  - Payment is posted to `/RMS/GooglePay/payment_v2.php`

### Step 4: Server-Side Verification and Redirection
- Depending on server response:
  - User is redirected via auto-submitted `<form>`
  - In case of failure or timeout, cancellation is triggered via `/RMS/GooglePay/cancel.php`

---

## 🔄 Fallback Handling

- If Google Pay is cancelled by user or errors occur:
  - Cancel request is sent to Fiuu
  - Page will auto-redirect as instructed
- If timeout occurs (after a configurable expiration), payment is aborted

---

## 📁 File Overview

| Function                     | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| `onGooglePaymentButtonClicked()` | Kicks off the full Google Pay flow                                          |
| `processPayment()`               | Handles the `paymentData` after Google Pay sheet is completed              |
| `startTimer()`                  | Starts countdown timer for transaction                                     |
| `makePayment()`                 | Posts the token and merchant data to the backend and handles redirection   |
| `cancel.php`                    | Hit in case of error, timeout, or user cancellation                         |

---

## 📌 Notes

- Customize messages and timeouts as per your UX

## 📞 Support

- Fiuu Developer Docs: [Direct server API](https://github.com/FiuuPayment/Documentation-Fiuu_API_Spec/blob/main/%5BOfficial%5D%20Fiuu%20Direct%20Server%20API%20v1.7.7.pdf)
- Google Pay Docs: [https://developers.google.com/pay](https://developers.google.com/pay)
---

