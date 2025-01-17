Update bank account
===================
.. api-name:: Reseller API
   :version: 1

.. warning:: The Reseller API has been deprecated. Only selected partners still have access to this legacy
             functionality. At this moment, it's no longer possible to update bank account information through the API.
             This information can still be updated via our Dashboard.

.. endpoint::
   :method: POST
   :url: https://www.mollie.com/api/reseller/v1/bankaccount-edit

This method allows you to change a merchant's bank account.

Parameters
----------
Make sure to add the :ref:`obligatory parameters <secret-keys>` always. Besides that, add the following
parameters:

.. note:: It is not necessary to set ``username`` and ``password`` if you are using ``partner_id_customer``. Otherwise
   both are required to set.

.. parameter:: username
   :type: string

   The username of the account of which you would like to change the bank account.

.. parameter:: password
   :type: string

   The password of the account of which you would like to change the bank account.

.. parameter:: partner_id_customer
   :type: string

   The partner ID of the account of which you would like to change the bank account. It can be used instead of the
   parameters ``username`` and ``password``.

.. parameter:: id
   :type: string
   :condition: required

   The ID of the bank account you would like to change

.. parameter:: name
   :type: string
   :condition: optional

.. parameter:: iban
   :type: string
   :condition: optional

.. parameter:: bic
   :type: string
   :condition: optional

.. parameter:: bankname
   :type: string
   :condition: optional

.. parameter:: location
   :type: string
   :condition: optional

   City where bank is domiciled.

Response
--------
.. code-block:: none
   :linenos:

   HTTP/1.1 200 OK
   Content-Type: application/xml; charset=utf-8

   <?xml version="1.0"?>
    <response>
        <success>true</success>
        <resultcode>10</resultcode>
        <resultmessage>Bankaccount successfully updated.</resultmessage>
        <bankaccount>
            <id>9d7512a3d2c16b5f9dd49b7aae2d7eaf</id>
            <account_name>JAN JANSEN</account_name>
            <account_iban>NL40RABO0123456789</account_iban>
            <bank_bic>RABONL2U</bank_bic>
            <bank>RABOBANK</bank>
            <location>AMSTERDAM</location>
            <selected>true</selected>
            <verified>false</verified>
        </bankaccount>
    </response>
