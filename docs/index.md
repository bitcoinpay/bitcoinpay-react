# BitcoinPay.com WhiteLabel Gateway

Build fully customizable bitcoin payment gateway on the top of [BitcoinPay.com API](http://docs.bitcoinpaycom.apiary.io/)

## How to use the library
You need to include library on your page in order to use it. If you want to use your own templates, you can also include JSXTransformer, so your templates are compiled to valid JS in browser (see [customization section](customize.md) of the documentation to learn more). You might
also want to include bootstrap.css for some basic styling, but is completely optional.

    ...
    <head>
        <!-- optional bootstrap styling -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css">
        <!-- BitcoinPay.com bundle -->
        <script src="https://bitcoinpay.com/static/js/bitcoinpay-bundle.js"></script>
    </head>
    ...

Once the library is included, you need to create HTML element, where payment states will be rendered.

     <!-- element for rendering states of the payment -->
     <div id="bitcoinpay" style="width: 700px;"></div>

And instantiate the library

    <script>
    var settings = {paymentId: 'payment_id', elementId: 'bitcoinpay'};
    BitcoinPay.api.payment(settings);
    </script>

**Remember, that your notification endpoint (see [Payment Notification](http://docs.bitcoinpaycom.apiary.io/#introduction/payment-notification) must return 200 response, so we can broadcast notification over the websockets to the client, and be sure, that payment status was updated on your side.**


## Customization
You can create your own custom component (template) for each payment state (see [List of payment states](index.md#list-of-payment-states) and register it with the library like this

For example, to override default template for `timeout` state:

    ...
    <!-- it is important to use "text/jsx" type in order to transform JSX to plain js -->
    <script type="text/jsx">
    var timeout = React.createClass({
      render: function() {
        return (
            <h1>Invoice {this.props.invoice.data.payment_id} expired</h1>
        )
      }
    });
    </script>
    ...

You don't need to focus on the `React.createClass` boilerplate, only thing you need to edit is content of the return function.

You can access all attributes of the invoice stored in `{this.props.invoice.data}` object. In the example above, you can see we are using `payment_id` attribute (`{this.props.invoice.data.payment_id}`). See [List of invoice attributes](index.md#list-of-invoice-attributes) to see which attributes are available to use.

To register your custom template, just add it to the templates objects like this

    ...
    <script type="text/javascript">
    templates = {'timeout': timeout};
    var settings = {paymentId: 'payment_id', elementId = 'bitcoinpay', templates: templates};
    BitcoinPay.api.payment(settings);
    </script>
    ...


## List of invoice attributes

* `status` - State of the payment in our system
* `payment_id` - Id of the payment (the one you initialize library with)
* `total_paid` - Total amount paid by customer
* `paid_currency` - Currency of the payment
* `price` - Total fiat amount
* `currency` - Payment currency
* `missing` - Amount missing for the invoice to be considered fully paid
* `timeout_time` - Invoice expiration time
* `created_time` - Invoice creation time
* `address` - Address for payment
* `confirmations` - Number of confirmations
* `txid` - Transaction id(s) associated with the invoice
* `payment_url` - URL with the link to BitcoinPay.com SCI
* `status_label` - Our default status label, can be used as header for instance
* `status_description` - More detailed explanation of the current state of the invoice
* `insufficient_amount` - `bool` - inditaces if user has paid full amount of the invoice or not




## List of payment states

* `pending` - Initial state of the payment, where payment details are displayed
* `received` - Payment has been received,but not confirmed yet
* `insufficient_amount` - Customer sent amount lower than required. Customer can ask for the refund directly from the invoice url
* `invalid` - An error has occured
* `timeout` - Payment has not been paid in given time period and has expired
* `paid_after_timeout` - Payment has been paid too late. Customer can ask for refund directly from the invoice url
* `refund` - Payment has been returned to customer.
* `confirmed` - This is **THE ONLY** payment status, you can consider as final. Payment is credited into balance and will be settled

You can override default template for each state by creating new [React component](https://facebook.github.io/react/docs/component-api.html) and registering it with the library.

See [Customization](customize.md) section of the documentation to learn more.

## Predefined components
We already provide numerous components, which you can use as building blocks in your custom template. All predefined compontents are available under `BitcoinPay.api.components` namespace.

See section [Using predefined components](customize.md#using-predefined-components) to learn how to use them.

* `BitcoinPay.api.components.QrCode` - renders qr code with the payment instructions (`<canvas></canvas>`)

* `BitcoinPay.api.components.ReceivingAddress` - renders bitcoin address for the payment  (`<span class="bitcoinpay-receiving-address">address</span>`)

* `BitcoinPay.api.components.CountDown` - renders countdown to invoice timeout_time (`<div class="bitcoinpay-countdown"></div>`)

* `BitcoinPay.api.components.InsufficientAmount` - renders message about invoice not being fully paid (`<div class="bitcoinpay-insufficient-amount">You just paid <span class="total-paid">total_paid</span>, but still missing <span class="missing">missing</span></div>`)

* `BitcoinPay.api.components.StatusLabel` - renders `status_label` (`<h1 class="bitcoinpay-status-label">status_label</h1>`)

* `BitcoinPay.api.components.StatusDescription` - renders `status_description` (`<h2 class="bitcoinpay-status-description">status_description</h2>`)

* `BitcoinPay.api.components.PaymentId` - renders `payment_id` (`<span class="bitcoinpay-payment-id">payment_id</span>`)

* `BitcoinPay.api.components.Txid` - renders bitcoin transaction id (if any) (`<span class="bitcoinpay-txid">txid</span>`)

* `BitcoinPay.api.components.ToBePaidFiat` - renders amount/currency to be paid (in FIAT) (`<div class="bitcoinpay-tobepaid-fiat"><span class="amount">amount</span><span class="currency">currency</span></div>`)

* `BitcoinPay.api.components.ToBePaidBtc` - renders amount/currency to be paid (in BTC) (`<div class="bitcoinpay-tobepaid-btc"><span class="amount">amount</span><span class="currency">currency</span></div>`)



## Putting it all together
Just try fully working example bellow

    <!doctype html>
    <html>
    <head>
    <meta charset="utf-8"/>
    <!-- optional bootstrap styling, can be removed and restyled -->
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css">

    <!-- BitcoinPay.com bundle -->
    <script src="https://bitcoinpay.com/static/js/bitcoinpay-bundle.js"></script>

    <!-- React JSXTransformer used for compilation of the jsx templates to JS. Used mostly for dev purposes,
     it is recommended to pre-compile templates on the production -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.13.2/JSXTransformer.js"></script>
    </head>

    <body>
     <!-- element for rendering states of the payment -->
     <div id="bitcoinpay" style="width: 700px;"></div>

     <!-- it is important to use "text/jsx" type in order to transform JSX to plain js -->
    <script type="text/jsx">

    // custom React component used to override default template for payment state 'timeout'
    // for more informations on React components see https://facebook.github.io/react/docs/reusable-components.html
    var timeout = React.createClass({
      render: function() {
        return (<h1>Invoice {this.props.invoice.data.payment_id} expired</h1>)
      }
    });

    var settings = {};
    // paymentId from API, see
    // http://docs.bitcoinpaycom.apiary.io/#reference/payment/create-payment-request/create-new-payment-request
    settings.paymentId = "0xGLWT20JgBn9K3W";
    settings.elementId = "bitcoinpay";

    settings.templates = {'timeout': timeout}; // registers custom template created above

    // instantiate the library
    BitcoinPay.api.payment(settings);
    </script>

    </body>
    </html>
