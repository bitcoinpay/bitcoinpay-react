# Customization tutorial

Step by step tutorial on how to implement [BitcoinPay.com](https://bitcoinpay.com) javascript library. You can clone working copy of the tutorial from [our repository](https://github.com/bitcoinpay)

## Prerequisites
We will use [Flask](http://flask.pocoo.org/) microframework on the backend, since it's very minimalistic and well known. You will also need some server space to host our app, since it has to be accessible from the internet to work properly (it will receive notifications about state changes from [bitcoinpay](https://bitcoinpay.com). You can use [Heroku](https://www.heroku.com/) to host our app for free.

Next thing you will need is [BitcoinPay.com merchant account](https://bitcoinpay.com). We acccount with USD settlement. If you want to use another settlement currency, just change `settled_currency` to currency of your choice. You can set up settlement account in [payout section of the settings](https://bitcoinpay.com/en/setup/payout/).

Installed Python

## Install Flask

    pip install flask

## Create base of our app
Create file named `app.py` with following content in it

    import flask
    app = flask.Flask(__name__)

    @app.route("/")
    def checkout():
        return "Hello World!"

    if __name__ == "__main__":
        app.run(debug=True)


Now, you can verify our app works by running

    python app.py

You can check your browser on http://localhost:5000/ and see Hello World! message from Flask.

## Templates
Create new directory called `templates`

Our project directory should now look like this

    bitcoinpay-tutorial/
        app.py
        templates/

We will now create two templates in our template directory, which we´ll use in our tutorial. First, very simple `checkout.html` template, which will serve as our "checkout" page. It will contain nothing more, than simple form

    <!-- templates/checkout.html -->
    <!doctype html>
    <html>
      <head><meta charset="utf-8"/></head>
      <body>
        <h1>Sample checkout page</h1>
        <form method="post" action='/sci'>
          <label>Amount to be paid</label>
          <input type="number" name="amount" value="1"/>
          <input type="submit" value="Pay with bitcoin">
        </form>
      </body>
    </html>

And second template `sci.html. This will be template where all the magic will happen. We will include just `bitcoinpay-bundle.js`.

    <!-- templates/sci.html -->
    <!doctype html>
    <html>
      <head>
        <meta charset="utf-8"/>
        <script src="https://bitcoinpay.com/static/js/bitcoinpay-bundle.js"></script>
      </head>
      <body>
        <h1>Sample SCI page</h1>
        <div id="bitcoinpay"></div>
      </body>
    </html>

## Endpoint for receiving notifications
Each time payment state changes, we will receive notification to `notify_url`, url we will specify in our [Create Payment Request](http://docs.bitcoinpaycom.apiary.io/#reference/payment), so we need to create such endpoint in our app, in order to receive these.

We will update our `app.py` file and add `/notify` endpoint. Our updated file should look like this:

    import flask
    app = flask.Flask(__name__)

    @app.route("/")
    def checkout():
        return flask.render_template("checkout.html")

    @app.route("/notify", methods=['POST',])
    def notify():
        # there we should update payment status in
        # our database, but is out of the scope of
        # this tutorial, so we will just return 'ok'
        # instead. Remember, that it's necesarry to
        # return 200 response from notify_url
        return 'ok'

    if __name__ == "__main__":
        app.run(debug=True)

Make sure your endpoint can receive POST and make sure, that for instance CSRF protection is not enabled.

## Create payment
Last thing missing is creation of the payment itself. Please check our [Apiary documentation](http://docs.bitcoinpaycom.apiary.io/#reference/payment) to learn more about creating payments.

Modify your `app.py` so it looks like example bellow:

    from urllib2 import Request, urlopen
    import json
    import flask
    app = flask.Flask(__name__)

    SERVER_URL = 'http://example.com'

    # you can get your API key in your BitcoinPay.com account
    # (https://bitcoinpay.com/en/setup/api)
    API_TOKEN = 'bxkdxiVDwKUVsd1f2PiGuKlf'

    @app.route("/")
    def checkout():
        return flask.render_template("checkout.html")

    @app.route("/notify", methods=["POST",])
    def notify():
        # there we should update payment status in
        # our database, but is out of the scope of
        # this tutorial, so we will just return 'ok'
        # instead. Remember, that it's necesarry to
        # return 200 response from notify_url
        return "ok"

    @app.route("/sci", methods=["POST",])
    def sci():

        # currency of the settlement. Must be set in your account
        # (https://bitcoinpay.com/en/setup/payout/)
        settled_currency = 'USD'

        # please note, that BitcoinPay.com must be able to POST
        # to this URL
        notify_url = SERVER_URL + '/notify'

        # amount from our checkout form
        amount = flask.request.form.get('amount')
        currency = 'USD'

        reference = 'test payment'

        # dictionary with our post data
        data = {
            "settled_currency": settled_currency,
            "notify_url": notify_url,
            "price": amount,
            "currency": currency,
            "reference": reference
        }
        headers = {
            'Content-Type': 'application/json',
            'Authorization': 'Token {0}'.format(API_TOKEN)
        }
        request = Request('https://private-anon-abb4df6b7-bitcoinpaycom.apiary-proxy.com/api/v1/payment/btc', data=json.dumps(data), headers=headers)
        response_body = json.loads(urlopen(request).read())

        payment_id = response_body['data']['payment_id']
        return flask.render_template('sci.html', payment_id=payment_id)



    if __name__ == "__main__":
        app.run(debug=True)

Note that instead of using `payment_id` and our javascript library on your site, you could easilly redirect user to our gateway using `payment_url` from response_body. Its completelly up to you.
