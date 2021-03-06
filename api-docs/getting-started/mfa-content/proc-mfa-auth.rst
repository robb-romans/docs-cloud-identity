.. _auth-mfa-enabled-account:

Authenticate from a multifactor-enabled user account
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use these instructions to authenticate to a Rackspace Cloud account that
requires multi-factor authentication.

**Prerequisites**

-  Valid username and password credentials for a Rackspace Cloud
   account that has been :ref:`auth-config-mfa-phone`.

-  Access to the phone associated with the Rackspace Cloud account.

   ..  tip::
     If your account requires multi-factor authentication, make sure to
     :ref:generate bypass codes <auth-mfa-manage> that you can use to
     authenticate if you do not have the phone associated with your
     account.

#. Send a POST tokens authentication request to the Identity service
   with your `username` and `password`:

   .. code::

     $ curl https://identity.api.rackspacecloud.com/v2.0/tokens  \
          -X POST \
          -d '{"auth":{"passwordCredentials":{"username":"myUserName","password":"thePassword"}}}' \
          -H "Content-type: application/json"  --verbose | python -m json.tool

#. Retrieve the multi-factor authentication values generated by the
   Identity service.

   -  In the API response to the initial authentication request, locate
      the session ID value in the ``WWW-Authenticate`` header
      parameter:

      .. code::

          < HTTP/1.1 401 Unauthorized
          * Server Apache-Coyote/1.1 is not blacklisted
          < Server: Apache-Coyote/1.1
          < vary:  Accept, Accept-Encoding, X-Auth-Token
          < WWW-Authenticate: OS-MF sessionId='APU9ymMBWY5W-pTgnHuZEvjKsM5oG_ler4lC0g_EkCPYvPdUBHK55RWtsgpL5RZ22AyDNaVCNCz6mlDOwbJAI-RLFQywI7CgOvjH0MLhL5a6D-c4cd1x8BbZmy8uT8ejm7jzBUX_vDZ5R0Hcia5DkOB80yWNJ8XVKMxVYLg5Qwp0TPA2zx-HQOTM3xqVQE63u1mYDUqikrXQ', factor='PASSCODE'
          < Content-Type: application/json

          {
          "unauthorized": {
              "code": 401,
              "message": "Additional authentication credentials required."
              }
          }
              {
              "key":"value"
              }

   -  Get the multi-factor authentication passcode value from the SMS
      message sent to the phone associated with the account.

       
      **Example: SMS message with passcode**

      .. code::

          Rackspace SMS passcodes (will expire in 10 minutes): 1411594

#. Send a second authentication request that includes the multi-factor
   authentication credentials: ``sessionId`` and ``passcode``.

   **cURL POST token request with multi-factor authentication credentials**

   .. code::

      $ curl https://identity.api.rackspacecloud.com/v2.0/tokens \
            -X POST \
            -d '{"auth": {"RAX-AUTH:passcodeCredentials": {"passcode":"1411594"}}}'\
            -H "X-SessionId: $SESSION_ID" \
            -H "Content-Type: application/json" --verbose | python -m json.tool


   The authentication response returns the authentication token ID
   and the service catalog with a list of available services. Note
   that the information returned in the
   `RAX-AUTH:authenticated By` object shows authentication by both passcode and password.

       .. code::

           < HTTP/1.1 200 OK
           < Vary:  Accept, Accept-Encoding, X-Auth-Token
           < Content-Type: application/json
           < Content-Length: 375
           < Server: Jetty(6.1.25)
           <
           {
               "access": {
                   "serviceCatalog": [],
                   "token": {
                       "RAX-AUTH:authenticatedBy": [
                           "PASSCODE",
                           "PASSWORD"
                       ],
                       "expires": "2014-01-09T15:08:53.645-06:00",
                       "id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
                   },
                   "user": {
                       "RAX-AUTH:defaultRegion": "IAD",
                       "RAX-AUTH:federated": false,
                       "RAX-AUTH:multiFactorEnabled": true,
                       "id": "a64ee2047fc14cc7bc977caa3cfff35f",
                       "name": "myUserName",
                       "roles": [
                           {
                               "description": "User Admin Role.",
                               "id": "3",
                               "name": "identity:user-admin"
                           }
                       ]
                   }
               }
           }

After you authenticate successfully, use the information in the response to
:ref:`submit requests to Rackspace Cloud API services <send-api-requests>`.
For additional information, see the following topics:

-  :ref:`auth-config-mfa-phone`

-  Troubleshooting: :ref:`auth-mfa-manage-accounts`

-  :ref:`Multi-factor authentication API operations <multifactor-operations>`
