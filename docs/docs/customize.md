# Customization tutorial

In this tutorial, we will customize default gateway to our needs. We will continue working with our tutorial app from previous page.

# Template file
We can easily write templates inline to our sci.html template, but we will use separate javascript file, since it's cleaner and we can later share it accross our other sites for instance.

First, we need to create folder for our static files. It will be called `static` and it will contain one file, `bitcoinpay-templates.js`. Our directory structure should look like this:

    bitcoinpay-tutorial/
        app.py
        templates/
            checkout.html
            sci.html
        static/
            bitcoinpay-templates.js

Now we just need to make slight modifications in our `templates/sci.html`. First we will include React JSX Transformer, so our templates are compiled directly in the browser and we also need to tell the library to use our brand new templates.

This is our file with all the changes:

    <!-- templates/sci.html -->
    <!doctype html>
    <html>
      <head>
        <meta charset="utf-8"/>
        <script src="https://bitcoinpay.com/static/js/bitcoinpay-bundle.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/react/0.13.2/JSXTransformer.js"></script>
        <script type="text/jsx" src="/static/bitcoinpay-templates.js"></script>
      </head>
      <body>
        <h1>Sample SCI page</h1>
        <div id="bitcoinpay"></div>
        <script type="text/jsx">
          var settings = {paymentId: '{{ payment_id }}', elementId: 'bitcoinpay', templates: bitcoinpayTemplates};
          BitcoinPay.api.payment(settings);
        </script>
      </body>
    </html>

# Custom templates
Now we can start to modify our template file.

We will start with creating `bitcoinpayTemplates` object, which we added to the library in previous step:

    // /static/bitcoinpay-templates.js
    var bitcoinpayTemplates = {};

First template we want to override will be template for `pending` state, since it's the first thing our client see.

    var bitcoinpayTemplates = {

        'pending': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Our custom pending template</h1>
                    </div>
                )
            }
        }),

    };

As you can see, the template itself is almost pure HTML. You don't have to worry about `React.createClass`. It is just boilerplate code. You can read more about [React compontents here](https://facebook.github.io/react/docs/reusable-components.html). You can do almost anything in your custom component using React. We will focus just on the HTML part inside `return` function in this tutorial.

# Using predefined components
Except standard HTML tags, you can also use prefenider components, which represent common objects appearing on the invoice page, such as qr codes, bitcoin addresses, etc. You can see full list of predefined components [here](index.md#predefined-components)

We will add `QrCode` and `ReceivingAddress` to our custom pending template, so our clients can make payments

    var bitcoinpayTemplates = {

        'pending': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Our custom pending template</h1>
                        <BitcoinPay.api.components.QrCode/>
                        <BitcoinPay.api.components.ReceivingAddress/>
                    </div>
                )
            }
        }),

    };

# Using invoice attributes
In example above, we used `BitcoinPay.api.components.ReceivingAddress` to render address for the payment. This address is rendered like HTML `span` elements, but we want to have address in the `div` with our custom HTML `class`. We can use invoice attributes and rewrite example above as following

    var bitcoinpayTemplates = {

        'pending': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Our custom pending template</h1>
                        <BitcoinPay.api.components.QrCode/>
                        <div className="our-custom-class">{this.props.invoice.data.address}</div>
                    </div>
                )
            }
        }),

    };

Please note, that only difference from standard HTML notation is `className` instead of `class`. This is due to `class` being reserved javascript keyword.

For full list of attributes see [list of available attributes](index.md#list-of-invoice-attributes)


Our clients also need to know how much they need to pay in fiat and bitcoin, so lets add this information

    var bitcoinpayTemplates = {

        'pending': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Payment id: {this.props.invoice.data.payment_id}</h1>
                        <h2>Waiting for your payment</h2>
                        <BitcoinPay.api.components.QrCode/>
                        <div className="our-custom-class">{this.props.invoice.data.address}</div>
                        <div>Amount to be paid: {this.props.invoice.data.missing} BTC</div>
                    </div>
                )
            }
        }),

    };

We can add amount in FIAT currency and countdown till invoice expiration time, so our users know how much time do they have to pay for invoice. We will use predefined `CountDown` component for countdown and `ToBePaidFiat` for the fiat amount, but we could use invoice attributes as well.

    var bitcoinpayTemplates = {

        'pending': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Payment id: {this.props.invoice.data.payment_id}</h1>
                        <h2>Waiting for your payment</h2>
                        <BitcoinPay.api.components.QrCode/>
                        <div className="our-custom-class">{this.props.invoice.data.address}</div>
                        <div>Amount to be paid: {this.props.invoice.data.missing} BTC</div>
                        <BitcoinPay.api.components.ToBePaidFiat/>
                        <BitcoinPay.api.components.CountDown/>
                    </div>
                )
            }
        }),

    };

This should be enough for user to be able to pay for invoice. We will also override default template for `timeout` state (see [list of all possible payment states](index.md#list-of-payment-states)). We will add very simple template, just to inform user that payment has expired.

    var bitcoinpayTemplates = {

        'pending': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Payment id: {this.props.invoice.data.payment_id}</h1>
                        <h2>Waiting for your payment</h2>
                        <BitcoinPay.api.components.QrCode/>
                        <div className="our-custom-class">{this.props.invoice.data.address}</div>
                        <div>Amount to be paid: {this.props.invoice.data.missing} BTC</div>
                        <BitcoinPay.api.components.ToBePaidFiat/>
                        <BitcoinPay.api.components.CountDown/>
                    </div>
                )
            }
        }),

        'timeout': React.createClass({
            render: function() {
                return (
                    <div>
                        <h1>Payment {this.props.invoice.data.payment_id} has expired.</h1>
                    </div>
                )
            }
        }),

    };

Now you should be able to fully customize gateway to your needs. You migh also want to checkout how to apply inline styling with React [here](https://facebook.github.io/react/tips/inline-styles.html). If you have any questions regarding implementation, do not hesitate to contact us any time on tech@bitcoinpay.com.

