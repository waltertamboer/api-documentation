Mollie Components
=================

*Mollie Components* is a set of Javascript APIs that allow you to add the fields needed for credit card holder data to
your own checkout, in a way that is fully :abbr:`PCI-DSS SAQ-A (Payment Card Industry Data Security Standard
Self-Assessment Questionnaire A)` compliant.

.. figure:: ../images/mollie-components-preview@2x.jpg

At a high level, Mollie Components works by using a Javascript API to add fields to your checkout that your customer will use to enter
their credit card details, such as their card number.

Mollie Components does not give you access to the card data. Instead you need to create a payment upfront using the
:doc:`Create Payment </reference/v2/payments-api/create-payment>` or :doc:`Create Order </reference/v2/orders-api/create-order>` APIs.
A successful call to this API will return a transaction identifier [TODO] that you need to pass to Mollie Components to initialize
the frontend and let the shopper pay.

Depending on various factors, the payment will either be completed frictionless (immediately) or the shopper needs to perform
the `3D Secure authentication <https://help.mollie.com/hc/en-us/articles/115000696789-What-is-3D-Secure-and-how-do-I-activate-it->`_.
If the customer authenticates successfully, the payment is completed.

The 3D Secure will be shown via an iFrame on your checkout in either a lightbox/modal or in a DOM target defined by you.

Implementation steps
--------------------

Follow these steps to implement Mollie Components in your checkout:

.. figure:: ../images/mollie-components-flow@2x.png

[TODO]

#. Add the Mollie Components Javascript library to your checkout.
#. Initialize the ``Mollie`` object.
#. Create and mount the four Components for the four credit card fields (card holder, card number, expiry date and
   :abbr:`CVC (Card Verification Code)`). This will add the fields to your checkout.
#. Add a ``submit`` event listener to your form so you can create :doc:`Create Payment API </reference/v2/payments-api/create-payment>` or
   :doc:`/reference/v2/orders-api/create-order` respectively for a JIT (just in time) checkout
#. In your frontend, send the transaction identifier to the ``finalizeCreditCardPayment`` method.
#. Mollie Components will figure out if 3D Secure is necessary and if so it will present a challenge to the customer.
#. After 3D Secure has been handled correctly by the customer the payment will be captured
#. Just like any other payment attempt, you will receive a webhook when the payment status changes

Mollie has created `example implementations <https://github.com/mollie/components-examples>`_ you can use to get started.

Add the Mollie Components Javascript library to your checkout
-------------------------------------------------------------

Start by including ``mollie.js`` in to your project. It should be added just before the ``</body>`` tag.

The Javascript file is located at ``https://js.mollie.com/v1/mollie.js``.

.. code-block:: html
   :linenos:

    <html>
      <head>
        <title>My Checkout</title>
      </head>
      <body>
        <script src="https://js.mollie.com/v1/mollie.js"></script>
      </body>
    </html>

.. note:: If you are using `Content Security Policy <https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP>`_, you
          should whitelist the ``js.mollie.com`` domain. We recommend using a strict CSP on your checkout.

Initialize the Mollie object
----------------------------

First, you need the Profile Id of the website profile that you want to use. This can be found on the
`Developers - API-keys <https://www.mollie.com/dashboard/developers/api-keys>`_ page in the Dashboard or retrieved
programmatically using the :doc:`Get Current Profile API </reference/v2/profiles-api/get-profile-me>`.

After the script has loaded you can use the :ref:`components-mollie-constructor` function. This will return
an object that you can use for creating the four Components your customer will use to enter their card holder data.

.. code-block:: js
   :linenos:

   var mollie = Mollie('pfl_3RkSN1zuPE', { locale: 'nl_NL', testmode: false });

.. note:: Be aware the Profile Id is *not* your API key. Your API key is private and should never be used in a browser
          context. The Profile Id starts with ``pfl_``, where as API keys start with ``live_`` or ``test_``.

[TODO - How test mode is going to happen]

Create and mount the card holder data Components
------------------------------------------------

After initializing the Mollie object, you should create the four card holder data Components using the
:ref:`components-mollie-create-component` function and mount them in your checkout using the
:ref:`components-mollie-component-mount` function:

.. code-block:: html
   :linenos:

   <form>
     <div id="card-number"></div>
     <div id="card-number-error"></div>

     <div id="card-holder"></div>
     <div id="card-holder-error"></div>

     <div id="expiry-date"></div>
     <div id="expiry-date-error"></div>

     <div id="verification-code"></div>
     <div id="verification-code-error"></div>

     <button type="button">Pay</button>
   </form>

.. code-block:: js
   :linenos:

   var cardNumber = mollie.createComponent('cardNumber');
   cardNumber.mount('#card-number');

   var cardHolder = mollie.createComponent('cardHolder');
   cardHolder.mount('#card-holder');

   var expiryDate = mollie.createComponent('expiryDate');
   expiryDate.mount('#expiry-date');

   var verificationCode = mollie.createComponent('verificationCode');
   verificationCode.mount('#verification-code');

This will add the input fields to your checkout and make them visible for your customer. To add styling to the Components,
see :doc:`styling`.

Handling errors
---------------

Add a change event listener to each component to listen for errors. Displaying the error is up to you. The example below
assumes an empty element in which the error can be rendered.

Errors will be localized according to the locale defined when initializing Mollie Components.

.. code-block:: js
   :linenos:

   var cardNumberError = document.querySelector('#card-number-error');

   cardNumber.addEventListener('change', event => {
     if (event.error && event.touched) {
       cardNumberError.textContent = event.error;
     } else {
       cardNumberError.textContent = '';
     }
   });

Add a submit event listener to your form
----------------------------------------

Add a submit event listener to your form. In this method we want to retrieve the transaction identifier. This will be the input for the
``finalizeCreditCardPayment`` method. Take a look at the the reference for the [TODO! LINK TO METHOD].

You may already have the transaction identifier upfront. This is scenario is possible and valid as long as the amount and currency of your
payment is final and will not change. This is for example the case if you use a multi step form checkout.

.. code-block:: js
   :linenos:

   form.addEventListener('submit', async e => {
     e.preventDefault();
     // From this point we need the payment detail transaction identifier.
     try {
       // Fetch the transaction identifier if needed. Your backend can do that by calling the Mollie API
       const { transactionID } = await fetch('www.yourApiDomain.com/getTransactionId')
       .then(response => response.json());

       // This call will try to finalize the payment and show the 3D secure in a lightbox. For customization see the api reference
       const Response = await mollie.finalizeCreditCardPayment(transactionID);

      if(Response.data.success === true){
        // Now the payment is finalized, you can redirect the user to a success page, for example:
        // window.location.href = Response.data.redirectUrl;
      }

     } catch (e) {
       // Something wrong happened while creating the token. Handle this situation gracefully.
       console.log(e);
     }
   });

[TODO make code ES5]

Browser support
---------------

Mollie Components supports the current and previous major release of the following browsers:

- Chrome
- Chrome for Android
- Safari
- Safari iOS
- Opera
- Firefox
- Edge

The latest release of Microsoft Internet Explorer 11 is supported as well.

If you need to support older browsers, you cannot use Mollie Components.
