.. include:: ../common/common_definitions.rst

e-Service PDND
+++++++++++++++++++

The `EIDAS-ARF`_ framework empowers Member States to establish the interfaces, terms, and conditions governing communication between Credential Issuers and Authentic Sources. In the Italian context, interoperability is established by leveraging the following guidelines:

    - "Linee Guida sull'interoperabilità tecnica delle Pubbliche Amministrazioni" (`MODI`_);
    - "Linee Guida sull'infrastruttura tecnologica della Piattaforma Digitale Nazionale Dati per l'interoperabilità dei sistemi informativi e delle basi di dati" (`PDND`_).

To leverage the PDND, entities must formally subscribe, becoming **Participants** (*Aderenti*). Within the PDND infrastructure, Participants can assume the following roles:

    - **Providers** (*Erogatori*): expose e-Services to other Participants.
    - **Consumers** (*Fruitori*): utilize e-Services offered by Providers within the PDND infrastructure.

Access to an e-Service requires Consumers to obtain a specific Access Token, known within the PDND infrastructure as a Voucher.


Requirements and Security Patterns
========================================


.. list-table::
    :widths: 10 70 20
    :header-rows: 1

    * - **ID**
      - **Description**
      - **Type**
    * - R1
      - The Consumer and the Provider have both subscribed to the PDND infrastructure.
      - Architectural
    * - R2
      - The communication between the Consumer and the Provider MUST ensure data integrity, authenticity, non-repudiation and replay protection.
      - Security
    * - R3
      - The Provider MAY require the Consumer to provide tracked data to complement the request. In that case, there MUST be a correlation between the tracked data and the Voucher.
      - Security
    * - R4
      - The e-Services MUST be implemented in REST, thus SOAP protocol MUST NOT be used.
      - Technical

The following security patterns are applicable:

.. list-table::
    :widths: 80 20
    :header-rows: 1

    * - **Security Pattern**
      - **Compliant With**
    * - **[REST_JWS_2021_POP]** JWS POP Voucher Issuing Profile (*Annex 3 - Standards and technical details used for Voucher Authorization* [`PDND`_]): REQUIRED. It adds a proof of possession on the Voucher. The client using the Voucher to access an e-service MUST demonstrate the proof of possession of the private key whose public is attested on the Voucher.
      - R2, R4
    * - **[ID_AUTH_CHANNEL_01]** Direct Trust Transport-Level Security (*Annex 2 - Security Patterns* [`MODI`_]): REQUIRED. It protects the communication between the Consumer and the Provider by ensuring confidentiality, integrity, identification of the Provider, and mitigation against replay attack and spoofing.
      - R1, R2
    * - **[INTEGRITY_REST_02]** REST Payload Integrity in PDND (*Annex 2 - Security Patterns* [`MODI`_]): REQUIRED. It ensures the integrity of the payload of the REST Consumer request, within the PDND Infrastructure.
      - R2, R4
    * - **[AUDIT_REST_02]** Submission of audit data within the REST request with correlation (*Annex 2 - Security Patterns* [`MODI`_]): OPTIONAL. The Provider MAY request additional data tracked in the Consumer's domain, with a correlation between such data and the authentication method. In that case, this pattern MUST be used.
      - R3, R4


The following custom security pattern is additionally applicable:

.. list-table::
    :widths: 80 20
    :header-rows: 1

    * - **Security Pattern**
      - **Compliant With**
    * - REST Response Payload Integrity in PDND: REQUIRED. It ensures the integrity of the payload of the REST Provider response, within the PDND Infrastructure.
      - R2


Some security patterns are not applicable as they do not comply with the requirements defined above:

    - The following patterns can only be used when the Consumer cannot subscribe to the PDND infrastructure (i.e., the trust between the Participants needs to be established in a direct form), thus not complying with **R1**:

        - **[ID_AUTH_CHANNEL_02]** Direct Trust mutual Transport-Level Security (*Annex 2 - Security Patterns* [`MODI`_])
        - **[ID_AUTH_REST_01]** Direct Trust based on X.509 certificate with REST (*Annex 2 - Security Patterns* [`MODI`_]).
        - **[ID_AUTH_REST_02]** Direct Trust based on X.509 certificate with REST and token/message uniqueness (*Annex 2 - Security Patterns* [`MODI`_]).
        - **[INTEGRITY_REST_01]** REST Payload Integrity (*Annex 2 - Security Patterns* [`MODI`_]).

    - The following pattern does not provide correlation between the tracked data and the Voucher, thus not complying with **R3**:

        - **[AUDIT_REST_01]** Submission of audit data within the REST request (*Annex 2 - Security Patterns* [`MODI`_]).

    - The following patterns are based on a SOAP architecture, thus not complying with **R4**:

        - **[ID_AUTH_SOAP_01]** Direct Trust based on X.509 certificate with SOAP (*Annex 2 - Security Patterns* [`MODI`_]).
        - **[ID_AUTH_SOAP_02]** Direct Trust based on X.509 certificate with SOAP and token/message uniqueness (*Annex 2 - Security Patterns* [`MODI`_]).
        - **[INTEGRITY_SOAP_01]** SOAP Payload Integrity (*Annex 2 - Security Patterns* [`MODI`_]).


Voucher Issuance
==========================

The PDND infrastructure defines two different types of Vouchers:
    
    - **For e-Service**: allow Consumers to request data from an e-Service.

    - **For Interoperability API**: allow Participants to request data from the Interoperability API, exposed by the PDND Infrastructure.

The two flows are described below.

Voucher for e-Service
--------------------------

Prerequisites
^^^^^^^^^^^^^^^^^^^^^^^^^^

The **Consumer** MUST comply with the following prerequisites:

    - Has successfully subscribed to the PDND Infrastructure (as per R1).
    - Has created a new `Client e-service` to interact with the intended e-Service. Upon creation, it has been assigned a ``client_id`` by the PDND Platform.
    - Has registered a key pair associated with the `Client e-service`.
    - Has requested to enroll in the intended e-Service.
    - Has defined a new purpose for the e-Service. Upon definition, it has been assigned a ``purposeId`` by the PDND Platform.
    - Has associated the `Client e-service` with the defined purpose.

The **Provider** MUST comply with the following prerequisites:

    - Has successfully subscribed to the PDND Infrastructure (as per R1).
    - Has created a new e-Service and published it within the PDND API Catalogue.
    - Has approved the Consumer's request to enroll in the e-Service.

Flow
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _fig_VoucherIssuance_eService_Flow:

.. figure:: ../../images/Low-Level-Flow-AuthenticSource-Voucher-Issuance-eService.svg
    :figwidth: 100%
    :align: center
    :target: https://www.plantuml.com/plantuml/svg/TT5HRzCm40VmTt-AK_QoHLH2a-1X3sXHkcf52AAbm6FakRV6Ik8pvtDPz-cnGxSYjBr4iMF_xd-Nwo2Q9KwZ4YiZmW-AfaU6TGXHEuEm05oqo7jhK2aTXLoSxf6LkgT7ACkQN8WJSbd2sNfFV500FedMMA-TG56MtweeFTIZHITbMO0EKAyKkYqaujbfbb-NsQYU8kDRHduGsoSjbXpaLCX7iMfF8dc15J5KtvVhJNTtqswwzvSAKjN5ftfrrJ0c7U7ppmtyjearDFaH9tIY-G1RBHhEYsB3sWpMxMxsTelEO55Sg1DIfpjH4DhRwlB3H-Xrw84UuBLh6Riz_t__8davibBF8gDEuBAX1WOyWDUMGQUUF1CzpAUgzkhb1ztXhuX1CcxcqNPsMnJlu6MfGq3EWtfzjvE3nx-VTgoT7DUfHtOa2BH7Xgg5iozsrqjZtP2Rxz4C8SUU3obLjOiv_2bLyEG2vbZQ60oX4nhmc26__biO7xrb39rrkV1rOkCq_W40

    Voucher Issuance for e-Service - Detailed flow

**Steps 1-2 (Preparation of Tracked Data):** The Consumer prepares a JWT (``TrackingEvidence``) containing the tracked data that have to be sent to the Provider. Finally, it computes the SHA-256 hash of ``TrackingEvidence``.

.. code-block::
    :caption: Non-normative example of the ``TrackingEvidence`` header
    :name: code_VoucherIssuance_eService_TrackingEvidence_Header

    {
        "alg": "ES256",
        "kid": "d4c3b2a1-9876-5432-10fe-dcba98765432",
        "typ": "JWT"
    }


.. code-block::
    :caption: Non-normative example of the ``TrackingEvidence`` payload
    :name: code_VoucherIssuance_eService_TrackingEvidence_Payload

    {
        "iss": "82914b3f-60b2-4529-b4d6-3d4e67f0a933",
        "aud": "https://erogatore.example/ente-example/v1",
        "exp": 1733052600,
        "nbf": 1733036450,
        "iat": 1733036400,
        "jti": "a4b5c6d7-e8f9-abcd-ef12-345678901234",
        "dnonce": 6528424213685,
        "purposeId": "b2c3d4e5-f6g7-h8i9-j0k1-lmno12345678",
        "userID": "a8b7c6d5-e4f3-g2h1-i9j0-klmnopqrstuv",
        "loa": "substantial"
    }

.. note::

    Steps 1-2 are required only when complying with the ``AUDIT_REST_02`` security pattern.

**Step 3 (DPoP Key Pair and Proof)**: The Consumer MUST create a new key pair for the DPoP and a fresh DPoP proof JWT following the instruction provided in the Section 4 of :rfc:`9449` for the token request to the PDND Authorization Server.

**Step 4 (Voucher Request)**: The Consumer creates a Voucher Request and sends it to the PDND Authorization Server.

.. code-block:: http
    :caption: Non-normative example of the Voucher Request
    :name: code_VoucherIssuance_eService_Request

    POST /authorization-server/token HTTP/1.1
    Host: interop.pagopa.it
    DPoP: eyJhbGciOiJSUzI1NiIsImtpZCI6ImY0YzRmMjE3LWE4N2It.eyJodHRwIj...
    Content-Type: application/x-www-form-urlencoded

    grant_type=client_credentials&
    client_id=82914b3f-60b2-4529-b4d6-3d4e67f0a933&
    client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&
    client_assertion=eyJhbGciOiJSUzI1NiIsImtpZCI6IjczYTQ3ZTQ5LTJhNDQtNDdjZS.eyJpc...


.. code-block::
    :caption: Non-normative example of the ``client_assertion`` JOSE header
    :name: code_VoucherIssuance_eService_ClientAssertion_Header

    {
        "alg": "ES256",
        "kid": "d4c3b2a1-9876-5432-10fe-dcba98765432",
        "typ": "JWT"
    }


.. code-block::
    :caption: Non-normative example of the ``client_assertion`` payload
    :name: code_VoucherIssuance_eService_ClientAssertion_Payload

    {
        "iss": "82914b3f-60b2-4529-b4d6-3d4e67f0a933",
        "sub": "82914b3f-60b2-4529-b4d6-3d4e67f0a933",
        "aud": "interop.pagopa.it/authorization-server",
        "exp": 1733041440,
        "iat": 1733037840,
        "jti": "7e9f3a4d-c9b2-42f6-a6d4-38e12fb6b8ab",
        "purposeId": "d2b9a653-c497-45c6-b8f1-5bdf124c9d3a",
        "digest": {
            "alg": "SHA256",
            "value": "5db26201b684761d2b970329ab8596773164ba1..."
        }
    }

.. note::

    The ``purposeId`` claim in the ``client_assertion`` payload is required only when requesting a Voucher for e-Service.

.. note::

    The ``digest`` claim in the ``client_assertion`` payload is required only when complying with the ``AUDIT_REST_02`` security pattern.

Upon the receipt of the Voucher Request, the PDND Authorization Server MUST perform the following checks on the Voucher Request body parameters:

    - The claim ``client_assertion_type`` is set to ``urn:ietf:params:oauth:client-assertion-type:jwt-bearer``.
    - The claim ``grant_type`` is set to ``client_credentials``.

The PDND Authorization Server MUST also validate the ``client_assertion`` JWT as follows:

    Header:

        - Ensure that the ``typ`` claim is present and that its value is ``jwt``.

    Signature:

        - Obtain the Consumer's public key corresponding to the ``kid`` header parameter, by interacting with the PDND Interoperability API.
        - Validate the signature of the JWT using the retrieved Consumer's public key and the algorithm specified by the ``alg`` header parameter.

    Payload:

        - The ``iss`` and ``sub`` claims MUST identify a Client registered in the PDND Infrastructure.
        - The ``aud`` claim MUST represent the PDND Authorization Server.
        - The ``exp`` claim MUST represent a time instant after the current time.
        - If the ``nbf`` claim is present, it MUST represent a time instant prior to the current time.
        - The ``iat`` claim MUST represent a time instant prior to the current time.
        - The ``jti`` claim MUST NOT have been previously used.
        - The ``purposeId`` claim MUST identify a purpose registered in the PDND Infrastructure and associated to the Client.

.. note::

    The PDND Authorization Server does not need to perform any checks on the ``digest`` claim.

.. note::

    Verification of the ``exp``, ``nbf``, ``iat``, and ``jti`` claims, as detailed above, MUST be performed for all JWTs described within this section. These checks will not be explicitly mentioned in subsequent references.

**Step 6 (Voucher Issuance)**: In case of successful checks, the PDND Authorization Server issues a Voucher, which is included in the Voucher Response to the Consumer.

.. code-block:: http
    :caption: Non-normative example of the Voucher Response
    :name: code_VoucherIssuance_eService_Response

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        "access_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6IkFiY2Rl.eyJ5bG9hZA...",
        "token_type": "DPoP",
        "expires_in": 3600
    }

.. code-block::
    :caption: Non-normative example of the ``access_token`` JOSE header
    :name: code_VoucherIssuance_eService_AccessToken_Header

    {
        "alg": "ES256",
        "kid": "b839f4c7-1e5d-4a8a-9fc6-72d3b7f091ec",
        "typ": "at+jwt"
    }

.. code-block::
    :caption: Non-normative example of the ``access_token`` payload
    :name: code_VoucherIssuance_eService_AccessToken_Payload

    {
        "iss": "interop.pagopa.it",
        "sub": "82914b3f-60b2-4529-b4d6-3d4e67f0a933",
        "aud": "https://erogatore.example/ente-example/v1",
        "exp": 1733042150,
        "nbf": 1733041945,
        "iat": 1733041920,
        "jti": "c4f5d7e2-b7c8-40f6-9b6a-dc9a4f5aeb57",
        "client_id": "82914b3f-60b2-4529-b4d6-3d4e67f0a933",
        "purposeId": "d2b9a653-c497-45c6-b8f1-5bdf124c9d3a",
        "digest": {
            "alg": "SHA256",
            "value": "5db26201b684761d2b970329ab8596773164ba1..."
        }
    }

.. note::

    The ``digest`` claim in the ``access_token`` payload is required only when complying with the ``AUDIT_REST_02`` security pattern. If present, it corresponds to the value of the ``digest`` claim contained in the ``client_assertion``.

Voucher for Interoperability API
-----------------------------------

Prerequisites
^^^^^^^^^^^^^^^^^^^^^^^^^^

The **Participant** MUST comply with the following prerequisites:

    - Has successfully subscribed to the PDND Infrastructure.
    - Has created a new `Client interop api` to interact with the Interoperability API. Upon creation, it has been assigned a ``client_id`` by the PDND Platform.
    - Has registered a key pair associated with the `Client interop api`.

Flow
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _fig_Voucher_InteroperabilityAPI_Issuance_Flow:

.. figure:: ../../images/Low-Level-Flow-AuthenticSource-Voucher-Issuance-InteroperabilityAPI.svg
    :figwidth: 100%
    :align: center
    :target: https://www.plantuml.com/plantuml/svg/TP51ozf048Rl-ok6UD7GqFOg4WmH8L3Qq41FXR36mIp6tNHcLjHVtrqDwk7xBY4px_Ay3xjh5atYJuCI8mF27-Ux7WagPgzjXv11PGN9ZKXwPmOZLoZgIYdjnLb_sY4fjNCNIDAq3YJJcL5RITE_TiNcXillt_9vwIpxKw0wMdsDs_mjhScTpHvfCKK9pAgpewi265_0oFzLUEcX70p6WWGqGxbLQFSG11uMGMBpSvEcC2jkdl0pHlmuCZSjnbbeaCv84x5eNiHoc-L5itnUoc_yvv45vadItIrQiq-IB_0SDJDIP7wyRUKwuYCwK12QveLIP9qWkA0H163smnnwgOqzAe2on-x8KUf-IAcYoRAyt2vVLZmwonYrTDSXSolkY9D3_7qlnije2BarDXQl0pOy_dy0

    Voucher Issuance for Interoperability API - Detailed flow

**Step 1 (Voucher Request)**: The Participant creates a Voucher Request and sends it to the PDND Authorization Server.

.. code-block:: http
    :caption: Non-normative example of the Voucher Request
    :name: code_Voucher_InteroperabilityAPI_Issuance_Request

    POST /authorization-server/token HTTP/1.1
    Host: interop.pagopa.it
    Content-Type: application/x-www-form-urlencoded

    grant_type=client_credentials&
    client_id=5a3c7f28-91b9-4c4e-89a9-6e2f85d9262b&
    client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&
    client_assertion=eyJhbGciOiJIUzI1NiIsImtpZCI6IjEyMzQ1Ng.eyJlcm5hbWU...

.. code-block::
    :caption: Non-normative example of the ``client_assertion`` JOSE header
    :name: code_Voucher_InteroperabilityAPI_Issuance_ClientAssertion_Header

    {
        "alg": "ES256",
        "kid": "9a4d8e3f-8b7d-4c98-926f-2745c6b1f832",
        "typ": "JWT"
    }

.. code-block::
    :caption: Non-normative example of the ``client_assertion`` payload
    :name: code_Voucher_InteroperabilityAPI_Issuance_ClientAssertion_Payload

    {
        "iss": "5a3c7f28-91b9-4c4e-89a9-6e2f85d9262b",
        "sub": "5a3c7f28-91b9-4c4e-89a9-6e2f85d9262b",
        "aud": "interop.pagopa.it/authorization-server",
        "exp": 1733233500,
        "iat": 1733232300,
        "jti": "d2c9a7b4-3e81-4d27-b6f7-51a8c9f0a3c6"
    }

Upon the receipt of the Voucher Request, the PDND Authorization Server MUST perform the following checks on the Voucher Request body parameters:

    - The claim ``client_assertion_type`` is set to ``urn:ietf:params:oauth:client-assertion-type:jwt-bearer``.
    - The claim ``grant_type`` is set to ``client_credentials``.

The PDND Authorization Server MUST also validate the ``client_assertion`` JWT as follows:

    Header:

        - Ensure that the ``typ`` claim is present and that its value is ``jwt``.

    Signature:

        - Obtain the Consumer's public key corresponding to the ``kid`` header parameter, by interacting with the PDND Interoperability API.
        - Validate the signature of the JWT using the retrieved Consumer's public key and the algorithm specified by the ``alg`` header parameter.

    Payload:

        - The ``iss`` and ``sub`` claims MUST identify a Client registered in the PDND Infrastructure.
        - The ``aud`` claim MUST represent the PDND Authorization Server.
  

**Step 2 (Voucher Issuance)**: In case of successful checks, the PDND Authorization Server issues a Voucher, which is included in the Voucher Response to the Participant.

.. code-block:: http
    :caption: Non-normative example of the Voucher Response
    :name: code_Voucher_InteroperabilityAPI_Issuance_Response

    HTTP/1.1 200 OK
    Content-Type: application/json
    Cache-Control: no-store

    {
        "access_token": "eyJhbGciOiJIUzI1NiIsImtpZCI6Ijg3Y2YxMjM.eyJ5bG9hZA...",
        "token_type": "Bearer",
        "expires_in": 3600
    }

.. code-block::
    :caption: Non-normative example of the ``access_token`` JOSE header
    :name: code_Voucher_InteroperabilityAPI_Issuance_AccessToken_Header

    {
        "alg": "ES256",
        "kid": "b839f4c7-1e5d-4a8a-9fc6-72d3b7f091ec",
        "typ": "at+jwt"
    }

.. code-block::
    :caption: Non-normative example of the ``access_token`` payload
    :name: code_Voucher_InteroperabilityAPI_Issuance_AccessToken_Payload

    {
        "iss": "interop.pagopa.it",
        "sub": "5a3c7f28-91b9-4c4e-89a9-6e2f85d9262b",
        "aud": "https://interop.pagopa.it/api/v1",
        "exp": 1733236680,
        "nbf": 1733233158,
        "iat": 1733233080,
        "jti": "f87e2d5b-9f65-4f0f-8ad4-92e58e6b13c7",
        "client_id": "5a3c7f28-91b9-4c4e-89a9-6e2f85d9262b"
    }

PDND Authorization Server Endpoint
--------------------------------------

The PDND Authorization Server Endpoint issues Vouchers to Participants. These Vouchers serve as credentials for the Consumer to authenticate with the e-Service, or to interact with the Interoperability API.

Voucher Request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The request to the PDND Authorization Server Endpoint adheres to the Client Credentials Grant flow specified in :rfc:`6749`. The client authenticates itself by presenting a JWT-based client assertion as defined in :rfc:`7521` and :rfc:`7523`.

Following the specifications above, the request MUST be an HTTP POST request with a body encoded in ``application/x-www-form-urlencoded`` format.

The Voucher Request MUST include the following HTTP header parameters (unless otherwise specified):

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **DPoP**
      - DPoP proof JWT, to comply with the ``REST_JWS_2021_POP`` security pattern. It is mandatory only if the requested Voucher is for e-Service (i.e., not for Interoperability API).
      - [:rfc:`9449`], [`PDND`_]

The Voucher Request MUST include the following body parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **client_id**
      - The unique identifier of the Participant Client, assigned by the PDND.
      - [:rfc:`6749`], [:rfc:`7521`], [:rfc:`7523`], [`PDND`_]
    * - **client_assertion**
      - A JWT representing the client assertion.
      - [:rfc:`7521`], [:rfc:`7523`], [`PDND`_]
    * - **client_assertion_type**
      - MUST be set to ``urn:ietf:params:oauth:client-assertion-type:jwt-bearer``.
      - [:rfc:`7521`], [:rfc:`7523`]
    * - **grant_type**
      - MUST be set to ``client_credentials``.
      - [:rfc:`6749`], [:rfc:`7523`]

The ``client_assertion`` JWT MUST include the following JOSE header parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **alg**
      - A digital signature algorithm identifier.
      - [:rfc:`7515`]
    * - **kid**
      - Unique identifier of the JWK used by the Participant to sign the ``client_assertion``.
      - [:rfc:`7515`]
    * - **typ**
      - MUST be set to ``JWT``.
      - [:rfc:`7515`], [:rfc:`7519`]

The ``client_assertion`` JWT MUST include the following payload claims (unless otherwise specified):

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **iss**
      - MUST be set to the same value as ``client_id``.
      - [:rfc:`7523`]
    * - **sub**
      - MUST be set to the same value as ``client_id``.
      - [:rfc:`7523`]
    * - **aud**
      - The identifier of the PDND Authorization Server Endpoint.
      - [:rfc:`7523`]
    * - **exp**
      - UNIX timestamp representing the JWT expiration time.
      - [:rfc:`7523`]
    * - **nbf**
      - UNIX timestamp representing the JWT first validity time (optional).
      - [:rfc:`7519`]
    * - **iat**
      - UNIX timestamp representing the JWT issuance time.
      - [:rfc:`7523`]
    * - **jti**
      - Unique identifier of the JWT to prevent replay attacks.
      - [:rfc:`7523`]
    * - **purposeId**
      - The identifier of the purpose registered in the PDND Platform, associated with the intended e-Service. It is mandatory only if the requested Voucher is for e-Service (i.e., not for Interoperability API).
      - [`MODI`_], [`PDND`_]
    * - **digest**
      - JSON object containing the digest of the ``TrackingEvidence`` JWT. It is mandatory only if the requested Voucher is for e-Service (i.e., not for Interoperability API), and when complying with ``AUDIT_REST_02``. If present, it MUST contain the following claims:

        - **alg**: JSON string representing the hashing algorithm;
        - **value**: JSON string representing the value of the digest.
      - [`MODI`_]

Voucher Response
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Voucher Response MUST include the following body parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **access_token**
      - A JWT representing the access token issued by the PDND Authorization Server Endpoint.
      - [:rfc:`6749`], [:rfc:`9449`], [`PDND`_]
    * - **token_type**
      - It MUST be set to:
      
        - ``DPoP`` in case of Voucher for e-Service;
        - ``Bearer`` in case of Voucher for Interoperability API.
      - [:rfc:`6749`], [:rfc:`9449`]
    * - **expires_in**
      - Number that represents the lifetime of the access token in seconds as a positive integer.
      - [:rfc:`6749`], [:rfc:`9449`]

The ``access_token`` JWT MUST include the following JOSE header parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **alg**
      - A digital signature algorithm identifier.
      - [:rfc:`7515`]
    * - **kid**
      - Unique identifier of the JWK used by the PDND Authorization Server Endpoint to sign the ``access_token``.
      - [:rfc:`7515`]
    * - **typ**
      - MUST be set to ``at+jwt``.
      - [:rfc:`9068`]

The ``access_token`` JWT MUST include the following payload claims (unless otherwise specified):

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **iss**
      - The identifier of the PDND Authorization Server.
      - [:rfc:`7519`], [:rfc:`9068`]
    * - **sub**
      - The identifier of the Participant, corresponding to the ``client_id`` body parameter in the Voucher Request.
      - [:rfc:`7519`], [:rfc:`9068`]
    * - **aud**
      - The identifier of the e-Service.
      - [:rfc:`7519`], [:rfc:`9068`]
    * - **exp**
      - UNIX timestamp representing the JWT expiration time.
      - [:rfc:`7519`], [:rfc:`9068`]
    * - **nbf**
      - UNIX timestamp representing the JWT first validity time (optional).
      - [:rfc:`7519`]
    * - **iat**
      - UNIX timestamp representing the JWT issuance time.
      - [:rfc:`7519`], [:rfc:`9068`]
    * - **jti**
      - Unique identifier of the JWT to prevent replay attacks.
      - [:rfc:`7519`], [:rfc:`9068`]
    * - **client_id**
      - MUST correspond to the ``client_id`` contained in the Voucher Request.
      - [:rfc:`7519`], [:rfc:`8963`], [:rfc:`9068`], [`PDND`_]
    * - **purposeId**
      - MUST correspond to the value of the ``client_id`` claim contained in the Voucher Request. It is mandatory only if the requested Voucher is for e-Service (i.e., not for Interoperability API).
      - [`MODI`_], [`PDND`_]
    * - **digest**
      - MUST correspond to the value of the ``digest`` object contained in the Voucher Request. It is mandatory only when complying with ``AUDIT_REST_02``.
      - [`MODI`_]


Key Retrieval
==========================

PDND Keys
-----------

.. _fig_KeyRetrieval_PDND_Flow:

.. figure:: ../../images/Low-Level-Flow-AuthenticSource-KeyRetrieval-PDND.svg
    :figwidth: 100%
    :align: center
    :target: https://www.plantuml.com/plantuml/svg/PS_1IWCn58NXVPxYK7U1-01Tb1Pc8GWgcSMiq_Q8eMbIRsvc79_UgOWAtHxuF-xTLWsYxPoCHat48sTugaIE8S7XfNlgZ0bDIsxdFSMOVVc0jTVTMjpjFflG09T5YOTu2LcnilP-OahkbCkKhLqXL0o6-OWb_XMaZF58kIeF9NFkEDr2pxkcR_2ifSc1w2aZvOVq_P_fUxSPrl1yRwoAroBVf-F4kHepEBVn_VhVj5tAvUil

    Key Retrieval for PDND Keys - Detailed flow

**Step 1 (Keys Request)**: The Provider requests for the keys used by the PDND to sign Vouchers.

.. code-block:: http
    :caption: Non-normative example of the Keys Request
    :name: _code_KeyRetrieval_PDND_Request

    GET /.well-known/jwks.json HTTP/1.1
    Host: interop.pagopa.it

**Step 2 (Keys Response)**: The .well-known Endpoint returns the list of keys used by the PDND to sign Vouchers, as a ``JWK Set`` [:rfc:`7517`].

.. code-block:: http
    :caption: Non-normative example of the Keys Response
    :name: _code_KeyRetrieval_PDND_Response

    HTTP/1.1 200 OK
    Content-Type: application/json

    {
      "keys": [
        {
            "kty": "RSA",
            "n": "qU2Bp7xgkXBQI2w2PZ5LZGo34TIjoir-ul0x4jZ_d9hN6q...",
            "e": "AQAB",
            "alg": "ES256",
            "kid": "b839f4c7-1e5d-4a8a-9fc6-72d3b7f091ec"
        },
        {
            "kty": "RSA",
            "n": "05VukHBwiE1W_kgUS0zkOyHCrRivgw5cfSTmcvD_phieEY...",
            "e": "AQAB",
            "alg": "ES256",
            "kid": "9432c16b-7aae-49df-b9c4-ea61b556652b"
        }
      ]
    }


Participants' Keys
-------------------

Prerequisites
^^^^^^^^^^^^^^^^

The **Participant** who requests the key MUST comply with the following prerequisites:

    - Has successfully subscribed to the PDND Infrastructure (as per R1).
    - Has created a new `Client api interop` to interact with the PDND Interoperability API. Upon creation, it has been assigned a ``client_id`` by the PDND Platform.
    - Has registered a key pair associated with the `Client api interop`.
    - Has obtained a valid Voucher to query the PDND Interoperability API, related to the specific `Client api interop`.

Flow
^^^^^^^^^^^^^^^^

.. _fig_KeyRetrieval_Participant_Flow:

.. figure:: ../../images/Low-Level-Flow-AuthenticSource-KeyRetrieval-Participant.svg
    :figwidth: 100%
    :align: center
    :target: https://www.plantuml.com/plantuml/svg/fSp1IlGm5CNnVPxYa6_nGQ0lC8jXIbkm25fcxIPjx-XbqYGcDsfzUYk85Low_f_pEsMnIShYbB0umYsjE9CafXVhK67OAayShPUib2qIV5b6IagDuGt63ErTQmp-rUybaGBYleSsflT2AKHATcJ7ig8UUcCqR4QloC_Ob6zgltwADy7JsjBhR_I-BlA4nZ5v-SAQRjUQhZhsXpCz5yg2IqZu8V_FY6LqFE5AwuEVGagKO_0p-qT8G8uqyMNBMbzvrbf1zTMnwGv_CKxavxAu3m00

    Key Retrieval for Participant's Key - Detailed flow

**Step 1 (Key Request)**: The Participant requests for the key used by another Participant, corresponding to a specific ``kid``, to the PDND Interoperability API.

.. code-block:: http
    :caption: Non-normative example of the Key Request
    :name: _code_KeyRetrieval_Participant_Request

    GET /keys/c7e3d6a4-5b99-4298-9b84-d8f3a61279f1 HTTP/1.1
    Host: interop.pagopa.it
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzd...

**Step 2 (Key Response)**: The Interoperability API Endpoint returns the requested key, as a ``JWK`` [:rfc:`7517`].

.. code-block:: http
    :caption: Non-normative example of the Key Response
    :name: _code_KeyRetrieval_Participant_Response

    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "kty": "RSA",
        "n": "v0GyA3SHrcHhTVxF0ItL64VThy2qG76KtIlptFyE4...",
        "e": "AQAB",
        "alg": "ES256",
        "kid": "c7e3d6a4-5b99-4298-9b84-d8f3a61279f1"
    }

.. note::

    The Interoperability API includes an event notification endpoint that alerts subscribed Participants about changes within the PDND Infrastructure. Among these notifications, the ``/events/keys`` endpoint provides updates on modifications to cryptographic material, such as additions or deletions of keys. By leveraging this mechanism, Participants can implement a periodic polling strategy to retrieve all changed keys and update their local cache. This eliminates the need to request each key individually during the workflow.


.well-known Endpoint
-------------------------------------

The .well-known Endpoint is part of the PDND Infrastructure and used to retrieve the public keys used by the PDND Authorization Server to sign the Vouchers.

Keys Request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Keys Request is a ``GET`` HTTP request sent to the .well-known Endpoint. This endpoint allows Participants to retrieve the public keys necessary to verify digital signatures on Vouchers issued by the PDND Authorization Server.

Keys Response
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The .well-known Endpoint responds with a ``200`` OK status code and a ``JWK Set`` [:rfc:`7517`] containing the public keys employed by the PDND Authorization Server to sign Vouchers.

Interoperability API Endpoint
-------------------------------------

The Interoperability API Endpoint is part of the PDND Infrastructure and used to retrieve the public keys of other parties enrolled in the PDND.

Key Request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Key Request is a ``GET`` HTTP request sent to the ``/keys/<kid>`` API. This request is used to retrieve a specific key identified by its unique ``kid``.

The Key Request MUST include the following HTTP header parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **Authorization**
      - Voucher released by the PDND Authorization Server.
      - [:rfc:`9449`]

Key Response
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Interoperability API Endpoint responds with a ``200`` OK status code and a ``JWK`` [:rfc:`7517`] representing the public key with the provided ``kid``.


e-Service Usage
===================

Prerequisites
----------------------

The **Consumer** MUST comply with the following prerequisites:

    - Has obtained a valid Voucher to interact with the intended e-Service, related to a specific `Client e-service`.

The **Provider** MUST comply with the following prerequisites:

    - Has created a new key ring associated with the specific e-Service.
    - Has registered a key pair associated with the key ring.

.. note:: 

    The key ring on the Provider's side is the reciprocal entity to the Client on the Consumer's side. This component serves as a store for cryptographic material, thus enabling Consumers to verify the integrity of responses transmitted by Providers.

Flow
-------

.. _fig_Usage_Flow:

.. figure:: ../../images/Low-Level-Flow-AuthenticSource-Usage.svg
    :figwidth: 100%
    :align: center
    :target: https////www.plantuml.com/plantuml/svg/ZP1HRzCm4CVV_IbEtWjOgeHut0DQbQmmX5YLTF1OUSwHM4ryuNpkw3uzph8LAZJK5ykMx_-xd_vNKInB6debNde4NDJ8U-yGxg9jKIcRX48Qxf6LkgTVO4n18QO1sHYukDaJ7nJ0c27U-T460MtCxJ991qNlCOWn6co4OgKmD90HBvnr-RMS6Cl7nFWQOpg8_QCLJm4cD-HduhB-XyYqzyrizea27afyU0rSOQJ43a-PrL_COhNuKAmkrumPWF3v-mOUV4v6tmOhYH7UQ5s_FBH-Uun0mMW5eLpJ6aOc-Z5_LVKbb5PiNlupOPIPzusIwwPwEL0EKRR97d3nnRUTV6J4e02A3jqEj9wGMa-IJzFdXQnp6EDMQ8VX94oZm6te3xE8rikxycQphQxhpVRT-pm9y3_ydmKRjcxdGtizrHySFaMFQ_BbFCzZcIUILRsOMtrk3Hjeh5XrbKVIqFy6

    e-Service Usage - Detailed flow

**Step 1 (Signature Preparation):** The Consumer prepares a JWT (``Signature``) containing the signed headers of the message, to ensure integrity.

.. code-block::
    :caption: Non-normative example of the ``Signature`` header
    :name: _code_Usage_Signature_Header

    {
        "alg": "ES256",
        "kid": "d4c3b2a1-9876-5432-10fe-dcba98765432",
        "typ": "JWT"
    }

.. code-block::
    :caption: Non-normative example of the ``Signature`` payload
    :name: _code_Usage_Signature_Payload

    {
        "iss": "9a8b7c6d-e5f4-g3h2-i1j0-klmnopqrstuv",
        "sub": "9a8b7c6d-e5f4-g3h2-i1j0-klmnopqrstuv",
        "aud": "https://erogatore.example/ente-example/v1",
        "iat": 1733397840,
        "nbf": 1733401628,
        "exp": 1733401440,
        "jti": "d3f7b2c9-274a-42b7-8f8d-2e9d8b1734b0",
        "signed_headers": [
            {"digest": "SHA-256=cFfTOCesrWTLVzxn8fmHl4AcrUs40Lv5D275FmAZ96E="},
            {"content-type": "application/json"}
        ]
    }

.. note::

    Step 1 is required to comply with the ``INTEGRITY_REST_02`` security pattern.

**Step 2 (DPoP Proof for e-Service Endpoint)**: The Consumer MUST create a fresh DPoP Proof JWT following the instruction provided in the Section 4 of [:rfc:`9449`] for the token presentation to the e-Service Endpoint.

**Step 3 (e-Service Request):** The Consumer sends an e-Service Request to the Provider, including the Voucher.

.. code-block:: http
    :caption: Non-normative example of the e-Service Request
    :name: _code_Usage_Request

    POST /ente-example/v1/hello/echo/ HTTP/1.1
    Host: erogatore.example
    Authorization: DPoP eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ...
    DPoP: eyJhbGciOiJSUzI1NiIsImtpZCI6ImYwOGYxNzY2LTlkZWEt.eyJpZCI6I...
    Agid-JWT-Signature: eyJhbGciOiJSUzI1NiIsImtpZCI6IjQ5M2Q1YjE4LTM5.eyJhdWQiOiJl...
    Digest: SHA-256=cFfTOCesrWTLVzxn8fmHl4AcrUs40Lv5D275FmAZ96E=
    Agid-JWT-TrackingEvidence: eyJhbGciOiJSUzI1NiIsImtpZCI6ImQ0YzNiMmExLTk4NzY.eyJhdWQiOi...
    Content-Type: application/json

    {
        "parameter1": "value1",
        "parameter2": "value2"
    }

The Provider MUST validate the DPoP proof [:rfc:`9449`].

The Provider MUST validate the Voucher as follows:

    Header:

        - Ensure that the ``typ`` claim is present and that its value is ``at+jwt``.

    Signature:

        - Retrieve the collection of public keys published at the .well-known endpoint. From this collection, select the public key whose identifier corresponds to the value of the ``kid`` header parameter in the Voucher.
        - Validate the signature of the ``access_token`` using the retrieved public key and the algorithm specified by the ``alg`` header parameter.

    Payload:

        - The ``iss`` claim MUST identify the domain of the PDND Authorization Server.
        - The ``sub`` claim MUST correspond to the ``client_id`` claim.
        - The ``aud`` claim MUST match the intended e-Service.

The Provider MUST validate the ``TrackingEvidence`` JWT as follows:

    Header:

        - Ensure that the ``typ`` claim is present and that its value is ``JWT``.

    Signature:

        - Obtains the Consumer's public key corresponding to the ``kid`` header parameter, by interacting with the PDND Interoperability API.
        - Validate the signature of the JWT using the retrieved Consumer's public key and the algorithm specified by the ``alg`` header parameter.

    Payload:

        - The ``iss`` claim MUST identify the Consumer Client.
        - The ``aud`` claim MUST identify the Provider.

In addition, the Provider MUST ensure that the hash of the ``TrackingEvidence`` JWT matches the value of the ``digest.value`` claim contained in the ``access_token`` payload.

.. note:: 

    The validation of the ``TrackingEvidence`` JWT is required only when complying with the ``AUDIT_REST_02`` security pattern.

The Provider MUST validate the ``Signature`` JWT as follows:

    Header:

        - Ensure that the ``typ`` claim is present and that its value is ``JWT``.

    Signature:
    
        - Validate the signature of the JWT using the retrieved Consumer's public key and the algorithm specified by the ``alg`` header parameter.

    Payload:

        - The ``iss`` and ``sub`` claims MUST identify the Consumer Client.
        - The ``aud`` claim MUST identify the Provider.

In addition, the Provider MUST validate the integrity of the e-Service Request, by checking that:

    - The ``signed_headers.content-type`` claim matches the value of the ``Content-Type`` HTTP header of the e-Service Request.
    - The ``signed_headers.digest`` claim matches the value of the digest of the payload of the e-Service Request.
        

If any of the previous checks fail, the Provider MUST reject the Request.

**Step 4 (e-Service Response):** Upon successful checks, the Provider provides the Consumer with the requested data.

.. code-block:: http
    :caption: Non-normative example of the e-Service Response
    :name: _code_Usage_Response

    HTTP/1.1 200 OK
    Content-Type: application/jwt

    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOi...

    
.. code-block::
    :caption: Non-normative example of the e-Service Response JWT header
    :name: _code_Usage_Response_JWT_Header

    {
        "alg": "ES256",
        "kid": "2802a69-1604-4261-9246-21453e20658e",
        "typ": "JWT"
    }

.. code-block::
    :caption: Non-normative example of the e-Service Response JWT payload
    :name: _code_Usage_Response_JWT_Payload

    {
        "iss": "https://erogatore.example/ente-example/v1",
        "aud": "9a8b7c6d-e5f4-g3h2-i1j0-klmnopqrstuv",
        "exp": 1733401785,
        "nbf": 1733401387,
        "iat": 1733401256,
        "jti": "997532e-871a-4969-9999-123456789abc",
        "requestedField1": "value1",
        "requestedField2": "value2",
        "requestedField3": "value3"
    }


The Consumer MUST perform the following steps to validate the e-Service Response JWT:

    Header:

        - Ensure that the ``typ`` claim is present and that its value is ``JWT``.

    Signature:
    
        - Obtain the Provider's public key corresponding to the ``kid`` header parameter, by interacting with the PDND Interoperability API.
        - Validate the signature of the JWT using the retrieved Provider's public key and the algorithm specified by the ``alg`` header parameter.

    Payload:

        - The ``iss`` claim MUST identify the Provider.
        - The ``aud`` claim MUST identify the Consumer Client itself.


e-Service Endpoint
-------------------------------------

e-Service Request
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The e-Service Request MUST include the following HTTP header parameters (unless otherwise specified):

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **Authorization**
      - Voucher released by the PDND Authorization Server.
      - [:rfc:`9449`], [`MODI`_], [`PDND`_]
    * - **DPoP**
      - DPoP proof JWT, to comply with the ``REST_JWS_2021_POP`` security pattern.
      - [:rfc:`9449`], [`PDND`_]
    * - **Agid-JWT-Signature**
      - JWT containing the signature of the message headers whose integrity needs to be guaranteed, to comply with the ``INTEGRITY_REST_02`` security pattern.
      - [`MODI`_]
    * - **Digest**
      - Digest of the message payload, to comply with the ``INTEGRITY_REST_02`` security pattern. According to :rfc:`3230`, the format MUST be the following: ``<digest-algorithm>=<encoded digest output>``.
      - [:rfc:`3230`], [`MODI`_]
    * - **Agid-JWT-TrackingEvidence**
      - JWT containing the data tracker in the Consumer's domain. It is mandatory only when complying with ``AUDIT_REST_02``.
      - [`MODI`_]

The ``Signature`` JWT, contained in the ``Agid-JWT-Signature`` HTTP header, MUST include the following JOSE header parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **alg**
      - A digital signature algorithm identifier.
      - [:rfc:`7515`]
    * - **kid**
      - Unique identifier of the JWK used by the Consumer to sign the JWT.
      - [:rfc:`7515`]
    * - **typ**
      - MUST be set to ``JWT``.
      - [:rfc:`7515`], [:rfc:`7519`]

The ``Signature`` JWT, contained in the ``Agid-JWT-Signature`` HTTP header, MUST include the following payload claims:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **iss**
      - MUST be set to the same value as ``client_id``.
      - [:rfc:`7519`]
    * - **sub**
      - MUST be set to the same value as ``client_id``.
      - [:rfc:`7519`]
    * - **aud**
      - The identifier of the Provider.
      - [:rfc:`7519`]
    * - **exp**
      - UNIX timestamp representing the JWT expiration time.
      - [:rfc:`7519`]
    * - **nbf**
      - UNIX timestamp representing the JWT first validity time (optional).
      - [:rfc:`7519`]
    * - **iat**
      - UNIX timestamp representing the JWT issuance time.
      - [:rfc:`7519`]
    * - **jti**
      - Unique identifier of the JWT to prevent replay attacks.
      - [:rfc:`7519`]
    * - **signed_headers**
      - JSON object containing the signed headers whose integrity needs to be protected, to with ``INTEGRITY_REST_02``. It MUST contain the following claims:

        - **digest**: JSON string representing the signature of the ``Digest`` HTTP header
        - **content-type**: JSON string representing the signature of the ``Content-Type`` HTTP header
      - [`MODI`_]

If present, the ``TrackingEvidence`` JWT, contained in the ``Agid-JWT-TrackingEvidence`` HTTP header, MUST include the following JOSE header parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **alg**
      - A digital signature algorithm identifier.
      - [:rfc:`7515`]
    * - **kid**
      - Unique identifier of the JWK used by the Consumer to sign the JWT.
      - [:rfc:`7515`]
    * - **typ**
      - MUST be set to ``JWT``.
      - [:rfc:`7515`], [:rfc:`7519`]

If present, the ``TrackingEvidence`` JWT, contained in the ``Agid-JWT-TrackingEvidence`` HTTP header, MUST include the following payload claims:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **iss**
      - MUST be set to the same value as ``client_id``.
      - [:rfc:`7519`]
    * - **aud**
      - The identifier of the Provider.
      - [:rfc:`7519`]
    * - **exp**
      - UNIX timestamp representing the JWT expiration time.
      - [:rfc:`7519`]
    * - **nbf**
      - UNIX timestamp representing the JWT first validity time (optional).
      - [:rfc:`7519`]
    * - **iat**
      - UNIX timestamp representing the JWT issuance time.
      - [:rfc:`7519`]
    * - **jti**
      - Unique identifier of the JWT to prevent replay attacks.
      - [:rfc:`7519`]
    * - **purposeId**
      - The identifier of the purpose registered in the PDND Platform, associated with the intended e-Service.
      - [`MODI`_]
    * - **dnonce**
      - It MUST be a random string composed by integer numbers and with a length of 13 digits.
      - [`MODI`_]

The ``TrackingEvidence`` payload MUST also contains the tracked data agreed upon with the Provider.

e-Service Response
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The e-Service Response is a JWT serialized in ``application/jwt`` format.

The e-Service Response JWT MUST include the following JOSE header parameters:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Parameter**
      - **Description**
      - **Reference**
    * - **alg**
      - A digital signature algorithm identifier.
      - [:rfc:`7515`]
    * - **kid**
      - Unique identifier of the JWK used by the Consumer to sign the JWT.
      - [:rfc:`7515`]
    * - **typ**
      - MUST be set to ``JWT``.
      - [:rfc:`7515`], [:rfc:`7519`]

The e-Service Response JWT MUST include the following payload claims:

.. list-table::
    :widths: 20 60 20
    :header-rows: 1

    * - **Claim**
      - **Description**
      - **Reference**
    * - **iss**
      - The identifier of the e-Service.
      - [:rfc:`7519`]
    * - **aud**
      - The identifier of the Consumer.
      - [:rfc:`7519`]
    * - **exp**
      - UNIX timestamp representing the JWT expiration time.
      - [:rfc:`7519`]
    * - **nbf**
      - UNIX timestamp representing the JWT first validity time (optional).
      - [:rfc:`7519`]
    * - **iat**
      - UNIX timestamp representing the JWT issuance time.
      - [:rfc:`7519`]
    * - **jti**
      - Unique identifier of the JWT to prevent replay attacks.
      - [:rfc:`7523`]

The e-Service Response JWT payload includes specific claims related to the data elements provided to the Consumer.
