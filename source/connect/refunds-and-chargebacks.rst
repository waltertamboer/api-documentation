Refunds and chargebacks
=======================
Refund and chargeback processing works differently based on whether you work with
:doc:`Application fees </connect/application-fees>` or with :doc:`Routed payments </connect/routing-payments>`.

Refunding a payment with application fees
-----------------------------------------
When using :doc:`Application fees </connect/application-fees>`, the connected merchant account is in full control of the
payment, and any refunds and chargebacks are also processed on their account.

As a platform, you can create refunds on behalf of the connected account by using the
:doc:`Refunds API </reference/v2/refunds-api/create-refund>` with the connected account's permission. Refunding
previously charged application fees is not possible, however.

For more fine-grained control over the refund and chargeback flows, consider using
:doc:`Routed payments </connect/routing-payments>` instead.

Refunding a routed payment
--------------------------
When using :doc:`Routed payments </connect/routing-payments>`, your platform is liable for refunds and chargebacks.

You can issue a refund for a routed payment by :doc:`creating a refund </reference/v2/refunds-api/create-refund>` on the
original payment, like you would with any other payment. By default, the full refund will be deducted from the platform
balance. In other words, by default the parts of the payment that were sent to connected accounts will remain untouched.

If you wish to pull back the money that was sent to a connected account, you can do so by 'reversing the routes' when
issuing the refund request.

For a full reversal of the routing that was specified during payment creation, simply set ``reverseRouting=true`` when
creating the refund.

In the example below we will refund the €10,00 payment from earlier, and pull back the €7,50 that was sent to connected
account ``org_23456``.

.. code-block:: bash
   :linenos:

   curl -X POST https://api.mollie.com/v2/payments/tr_7UhSN1zuXS/refunds \
       -H "Authorization: Bearer access_vR6naacwfSpfaT5CUwNTdV5KsVPJTNjURkgBPdvW" \
       -d "amount[currency]=EUR" \
       -d "amount[value]=10.00" \
       -d "reverseRouting=true"

.. code-block:: http
   :linenos:

   HTTP/1.1 201 Created
   Content-Type: application/hal+json; charset=utf-8

   {
       "resource": "refund",
       "id": "re_gj08ZdgmVx",
       "amount": {
           "currency": "EUR",
           "value": "10.00"
       },
       "status": "pending",
       "paymentId": "tr_7UhSN1zuXS",
       "routingReversal": [
           {
               "amount": {
                    "value": "7.50",
                    "currency": "EUR"
               },
               "source": {
                    "organizationId": "org_23456"
               }

           }
       ]
       "...": { }
   }
