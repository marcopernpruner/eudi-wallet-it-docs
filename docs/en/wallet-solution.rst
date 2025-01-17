.. include:: ../common/common_definitions.rst

.. _wallet-solution.rst:

Wallet Solution
-------------------

The Wallet Solution is issued by the Wallet Provider in the form of a mobile app and services, such as web interfaces.

The mobile app serves as the primary interface for Users,
allowing them to hold their Digital Credentials and interact with other participants of the ecosystem,
such as Credential Issuers and Relying Parties.

These Credentials are a set of data that can uniquely identify a natural or legal person,
along with other Qualified and non-qualified Electronic Attestations of Attributes,
also known as QEAAs and EAAs respectively, or (Q)EAAs for short[1]. 

Once a User installs the mobile app on their device, such an installation is referred to as a Wallet Instance for the User.

By supporting the mobile app, the Wallet Provider enusers the security and reliability of the entire Wallet Solution,
as it is responsible for issuing the Wallet Attestation,
which is a cryptographic proof about the authenticity and integrity of the Wallet Instance.

Requirements
^^^^^^^^^^^^

This section lists the requirements that are be met by Wallet Providers and Wallet Solutions.

 - The Wallet Provider MUST offer a RESTful set of services for issuing the Wallet Attestations.
 - The Wallet Instance MUST periodically reestablish trust with its Wallet Provider.
 - The Wallet Instance MUST establish trust with other participants of the Wallet ecosystem, such as Credential Issers and Relying Parties.
 - The Wallet Solutions MUST adhere to the specifications set by this document for obtaining Personal Identification (PID) and (Q)EAAs.
 - The Wallet Instance MUST be compatible and functional on both Android and iOS operating systems and available on the Play Store and App Store, respectively.
 - The Wallet Instance MUST provide a mechanism to verify the User's actual possession and full control of their personal device.

Wallet Instance
^^^^^^^^^^^^^^^
The Wallet Instance serves as a unique and secure device for authenticating the User within the Wallet ecosystem.
It establishes a strong and reliable mechanism for the User to engage in various digital transactions in a secure and privacy-preserving manner.

The Wallet Instance allows other entities within the ecosystem to establish trust with it, by consistently
presenting a Wallet Attestation during interactions with PID Providers,
(Q)EAA Providers, and Relying Parties. These verifiable attestations, provided by the Wallet Provider,
serve to authenticate the Wallet Instance itself, ensuring its reliability when engaging with other ecosystem actors.

To guarantee the utmost security, these cryptographic keys MUST be securely stored within the WSCD, which MAY be internal (device's Trusted Execution Environment (TEE)[3]), external, or hybrid. This ensures that only the User can access them, thus preventing unauthorized usage or tampering. For more detailed information, please refer to the `Wallet Attestation section`_ and the `Trust Model section`_ of this document.

Wallet Instance Lifecycle
^^^^^^^^^^^^^^^^^^^^^^^^^
The Wallet Instance has three distinct states: Operational, Valid, and Deactivated.
Each state represents a specific functional status and determines the actions that can be performed[2].

Initialization Process
~~~~~~~~~~~~~~~~~~~~~~
To activate the Wallet Instance, Users MUST install the mobile Wallet application on their device and open it. Furthermore, Users will be asked to set their preferred method of unlocking their device; this can be accomplished by entering a personal identification number (PIN) or by utilizing biometric authentication, such as fingerprint or facial recognition, according to their personal preferences and device's capabilities.

After completing these steps, the Wallet Instance enters the Operational state.

Transition to Valid state
~~~~~~~~~~~~~~~~~~~~~~~~~
To transition from the Operational state to the Valid state, the Wallet Instance MUST obtain a valid Personal Identification (PID). Once a valid PID is acquired, the Wallet Instance becomes Valid.

The Wallet Instance MUST demonstrate to the Credential Issuer adequate security compliance to maintain the Credential at the same LoA at which it was issued.

Once the Wallet Instance is in the Valid state, Users can:

 - Obtain, view, and manage (Q)EAAs from trusted (Q)EAA Providers[1];
 - Authenticate to Relying Parties[1];
 - Authorize the presentation of their digital Credentials to Relying Parties.

Please refer to the relevant sections for further information about PID and (Q)EAAs issuance and presentation.

Return to Operational state
~~~~~~~~~~~~~~~~~~~~~~~~~~~
A Valid Wallet Instance may revert to the Operational state under specific circumstances. These circumstances include the expiration or revocation of the associated PID by its PID Provider.

Deactivation
~~~~~~~~~~~~
Users have the ability to deactivate the Wallet Instance voluntarily. This action removes the operational capabilities of the Wallet Instance and sets it to the Deactivated state. Deactivation provides Users with control over access and usage according to their preferences.

Wallet Provider Endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^

The Wallet Provider that issues the Wallet Attestations MUST make its APIs available in the form of RESTful services, as listed below.

Wallet Provider Metadata
~~~~~~~~~~~~~~~~~~~~~~~~
An HTTP GET request to the **/.well-known/openid-federation** endpoint allows the retrieval of the Wallet Provider Entity Configuration.

The Wallet Provider Entity Configuration is a signed JWT containing the public keys and supported algorithms of the Wallet Provider metadata definition. It is structured in accordance with the `OpenID Connect Federation <https://openid.net/specs/openid-connect-federation-1_0.html>`_ and the Trust Model section outlined in this specification.

The returning Entity Configuration of the Wallet Provider MUST contain the attributes listed below:

Header
^^^^^^
.. list-table::
    :widths: 20 80
    :header-rows: 1

    * - **Key**
      - **Value**
    * - alg
      - Algorithm used to verify the token signature. It MUST be one of the possible values indicated in this `table <https://italia.github.io/eudi-wallet-it-docs/versione-corrente/en/algorithms.html>`_ (e.g., ES256).
    * - kid
      - Thumbprint of the public key used for signing, according to :rfc:`7638`.
    * - typ
      - Media type, set to ``entity-statement+jwt``.

Payload
^^^^^^^
.. list-table::
    :widths: 20 80
    :header-rows: 1

    * - **Key**
      - **Value**
    * - iss
      - Public URL of the Wallet Provider.
    * - sub
      - Public URL of the Wallet Provider.
    * - iat
      - Issuance datetime in Unix Timestamp format.
    * - exp
      - Expiration datetime in Unix Timestamp format.
    * - authority_hints
      - Array of URLs (String) containing the list of URLs of the immediate superior Entities, such as the Trust Anchor or an Intermediate, that MAY issue an Entity Statement related to this subject.
    * - jwks
      - A JSON Web Key Set (JWKS) `RFC 7517 <http://tools.ietf.org/html/rfc7517.html>`_ that represents the public part of the signing keys of the Entity at issue. Each JWK in the JWK set MUST have a key ID (claim kid).
    * - metadata
      - Contains the ``wallet_provider`` and ``federation_entity`` metadata.

wallet_provider metadata
~~~~~~~~~~~~~~~~~~~~~~~~~~

+---------------------------------------------+---------------------------------------------------------------------+
| **Key**                                     | **Value**                                                           |
+---------------------------------------------+---------------------------------------------------------------------+
| jwks                                        | A JSON Web Key Set (JWKS)                                           |
|                                             | that represents the  Wallet                                         |
|                                             | Provider's public keys.                                             |
+---------------------------------------------+---------------------------------------------------------------------+
| token_endpoint                              | Endpoint for obtaining the Wallet                                   |
|                                             | Instance Attestation.                                               |
+---------------------------------------------+---------------------------------------------------------------------+
| nonce_endpoint                              | HTTPs URL indicating the endpoint                                   |
|                                             | where the client can request the nonce.                             |
+---------------------------------------------+---------------------------------------------------------------------+
| aal_values_supported                        | List of supported values for the                                    |
|                                             | certifiable security context. These                                 |
|                                             | values specify the security level                                   |
|                                             | of the app, according to the levels: low, medium, or high.          |
|                                             | Authenticator Assurance Level values supported.                     |
+---------------------------------------------+---------------------------------------------------------------------+
| grant_types_supported                       | The types of grants supported by                                    |
|                                             | the token endpoint. It MUST be set to                               |
|                                             | ``urn:ietf:params:oauth:client-assertion-type:                      |
|                                             | jwt-client-attestation``.                                           |
+---------------------------------------------+---------------------------------------------------------------------+
| token_endpoint_auth_methods_suppor          | Supported authentication methods for                                |
| ted                                         | the token endpoint.                                                 |
+---------------------------------------------+---------------------------------------------------------------------+
| token_endpoint_auth_signing_alg_va          | Supported signature                                                 |
| lues_supported                              | algorithms for the token endpoint.                                  |
+---------------------------------------------+---------------------------------------------------------------------+


.. note::
   The `aal_values_supported` parameter is experimental and under review.

Payload `federation_entity`
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

+-------------------+----------------------------------------------+
| **Key**           | **Value**                                    |
+-------------------+----------------------------------------------+
| organization_name | Organization name.                           |
+-------------------+----------------------------------------------+
| homepage_uri      | Organization's website URL.                  |
+-------------------+----------------------------------------------+
| tos_uri           | URL to the terms of service.                 |
+-------------------+----------------------------------------------+
| policy_uri        | URL to the privacy policy.                   |
+-------------------+----------------------------------------------+
| logo_uri          | URL of the organization's logo in SVG format.|
+-------------------+----------------------------------------------+

Below a non-normative example of the Entity Configuration.

.. code-block:: javascript

  {
    "alg": "ES256",
    "kid": "5t5YYpBhN-EgIEEI5iUzr6r0MR02LnVQ0OmekmNKcjY",
    "typ": "entity-statement+jwt"
  }
  .
  {
  "iss": "https://wallet-provider.example.org",
  "sub": "https://wallet-provider.example.org",
  "jwks": {
    "keys": [
      {
        "crv": "P-256",
        "kty": "EC",
        "x": "qrJrj3Af_B57sbOIRrcBM7br7wOc8ynj7lHFPTeffUk",
        "y": "1H0cWDyGgvU8w-kPKU_xycOCUNT2o0bwslIQtnPU6iM",
        "kid": "5t5YYpBhN-EgIEEI5iUzr6r0MR02LnVQ0OmekmNKcjY"
      }
    ]
  },
  "metadata": {
    "wallet_provider": {
      "jwks": {
        "keys": [
          {
            "crv": "P-256",
            "kty": "EC",
            "x": "qrJrj3Af_B57sbOIRrcBM7br7wOc8ynj7lHFPTeffUk",
            "y": "1H0cWDyGgvU8w-kPKU_xycOCUNT2o0bwslIQtnPU6iM",
            "kid": "5t5YYpBhN-EgIEEI5iUzr6r0MR02LnVQ0OmekmNKcjY"
          }
        ]
      },
      "token_endpoint": "https://wallet-provider.example.org/token",
      "nonce_endpoint": "https://wallet-provider.example.org/nonce",
      "aal_values_supported": [
        "https://wallet-provider.example.org/LoA/basic",
        "https://wallet-provider.example.org/LoA/medium",
        "https://wallet-provider.example.org/LoA/high"
      ],
      "grant_types_supported": [
        "urn:ietf:params:oauth:client-assertion-type:jwt-client-attestation"
      ],
      "token_endpoint_auth_methods_supported": [
        "private_key_jwt"
      ],
      "token_endpoint_auth_signing_alg_values_supported": [
        "ES256",
        "ES384",
        "ES512"
      ]
    },
    "federation_entity": {
      "organization_name": "IT-Wallet Provider",
      "homepage_uri": "https://wallet-provider.example.org",
      "policy_uri": "https://wallet-provider.example.org/privacy_policy",
      "tos_uri": "https://wallet-provider.example.org/info_policy",
      "logo_uri": "https://wallet-provider.example.org/logo.svg"
    }
  },
  "authority_hints": [
    "https://registry.eudi-wallet.example.it"
  ]
  "iat": 1687171759,
  "exp": 1709290159
  }


Wallet Attestation
~~~~~~~~~~~~~~~~~~

Please refer to the `Wallet Attestation section`_.


External references
^^^^^^^^^^^^^^^^^^^^
.. [1] Definitions are inherited from the EUDI Wallet Architecture and Reference Framework, version 1.1.0 at the time of writing. Please refer to `this page <https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/9647a408f628569449af6b30a15fed82cd41129a/arf.md#2-definitions>`_ for extended definitions and details.

.. [2] Wallet Instance states adhere to the EUDI Wallet Architecture and Reference Framework, as defined `here <https://github.com/eu-digital-identity-wallet/eudi-doc-architecture-and-reference-framework/blob/9647a408f628569449af6b30a15fed82cd41129a/arf.md#424-eudi-wallet-instance-lifecycle>`_.

.. [3] Depending on the device operating system, TEE is defined by `Trusty`_ or `Secure Enclave`_ for Android and iOS devices, respectively.

.. _Trust Model section: trust.html
.. _Wallet Attestation section: wallet-attestation.html
.. _Trusty: https://source.android.com/docs/security/features/trusty
.. _Secure Enclave: https://support.apple.com/en-gb/guide/security/sec59b0b31ff/web

