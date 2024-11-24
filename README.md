# m10 e-commerce API Description

## Get started

m10 provides an e-commerce API that enables merchants to create payments and refunds as well as to create tokenization on m10 for seamless payment processes. The tokenization process will enable users to make payments through m10 without being redirected to the application. The API is organized around REST. Our API has predictable resource-oriented URLs, accepts form-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes, authentication, and verbs.

The provided document and Swagger link offer a comprehensive overview of the API, detailing its various endpoints. The link to [Swagger](https://develop.m10payments.com/online-acquiring/swagger-ui/index.html)

{% hint style="warning" %}
**Important note:**\
Authorization tokens will be provided manually by m10
{% endhint %}

***

## Authorization

m10 supports HTTP Bearer authentication. This allows merchants to protect the URLs on their web server so that only they and m10 can access them. To authenticate with HTTP, a merchant may generate the token on the admin panel of m10. However, during the initial stages, m10 will provide the token.

***

## Supported endpoints

### URL

| Environment      | URL                                                    |
| ---------------- | ------------------------------------------------------ |
| Test environment | `https://develop.m10payments.com/online-acquiring/...` |
| Prod environment | `https://api.m10.az/acquiring/...`                     |

### Endpoints

| HTTP Method | Endpoint                                     | Description                      |
| ----------- | -------------------------------------------- | -------------------------------- |
| POST        | `/api/v1/orders/actions/create-payment`      | Create payment                   |
| POST        | `/api/v1/orders/actions/create-refund`       | Create refund                    |
| GET         | `/api/v1/orders/{orderId}`                   | Retrieve order information       |
| GET         | `/api/v1/transactions/{transactionId}`       | Retrieve transaction information |
| POST        | `/api/v1/user-tokenization/actions/create`   | Create tokenization              |
| GET         | `/api/v1/user-tokenization/{tokenizationId}` | Get token information            |

***

### Status codes

HTTP response codes are used to indicate general classes of success and error.&#x20;

#### Success Code

| Status Code | Description                    |
| ----------- | ------------------------------ |
| 200         | Successfully processed request |

### Error Codes

Error responses contain more detail about the error in the `x-error-code (header)` and `description` properties.

| Status Code | `x-error-code` (header)  | Description                                                     |
| ----------- | ------------------------ | --------------------------------------------------------------- |
| 400         | absent                   | No `x-error-code` since these cases are not processed yet       |
| 401         | `onlineAcquiring-401001` | Unauthorized                                                    |
| 401         | `onlineAcquiring-401002` | Invalid token data                                              |
| 403         | `onlineAcquiring-403001` | Authentication data mismatch                                    |
| 409         | `onlineAcquiring-409001` | System already has an order with the same ID but different data |
| 409         | `onlineAcquiring-409002` | System is already processing this order                         |
| 500         | `onlineAcquiring-500001` | Internal server error                                           |
| 504         | absent                   | No `x-error-code` since the error occurred on the nginx server  |

***

## 1. Create a payment

### <mark style="color:blue;">**POST**</mark> `/api/v1/orders/actions/create-payment`

Creates a payment operation on m10. Returns on response a dynamic link paymentURL to redirect the user and `transactionId` of the completed operation on m10.

{% hint style="warning" %}
**Important note**

\
Upon completion of the operation (payment/refund), a callback will be sent containing the operation details and status. An example of the callback is provided **in the "Callback structure" section**.

If the callback is not received, use the method **Get order or transaction details** to get the operation details.
{% endhint %}

#### Request parameters

#### **Header**

<table data-full-width="true"><thead><tr><th>Name</th><th width="134">Type</th><th>Required</th><th>Pattern/Format</th><th>Description</th><th>Example</th></tr></thead><tbody><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer\s[a-zA-Z0-9_.-]{5,128}$</td><td>Auth header with Bearer <code>&#x3C;token></code></td><td><code>Bearer &#x3C;AccessToken></code></td></tr><tr><td>X-User-Token</td><td>string</td><td>false</td><td>[a-zA-Z0-9_.-]{20,64}</td><td>Header with token when user has m10 as saved payment method.</td><td><code>8e17cc7f-408b-4211-bd6e-aa38e5d6281a</code></td></tr><tr><td>X-User-Locale</td><td>string</td><td>false</td><td>-</td><td>Set of parameters that define the user's language, region, and any special variant preferences.</td><td><code>en-GB</code></td></tr><tr><td>X-User-Tokenization</td><td>string</td><td>true</td><td><code>PROVIDED</code>, <code>REQUIRED</code>, <code>NOT_REQUIRED</code></td><td>User token processing mode, when a user wants to save m10 for future payments.</td><td><code>PROVIDED</code></td></tr></tbody></table>

***

**Body**

| **Name**        | **Type**   | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                                   | **Example**                                                                                                                                                                                                                        |
| --------------- | ---------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| orderId         | string     | true         | \[a-zA-Z0-9\_-]{20,64}                                      | <p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | upLZGwJhAJGdxTlwoF89v98YLDFWNFPi                                                                                                                                                                                                   |
| currencyISO     | string     | true         | <p>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</p>         | Currency ISO of the transaction                                                                                                                                                                                                                                   | AZN                                                                                                                                                                                                                                |
| amount          | string     | true         | <p>"maximum":<br>1000000000 "minimum":<br>1</p>             | Amount of the transaction                                                                                                                                                                                                                                         | 10.51                                                                                                                                                                                                                              |
| confirmURL      | string     | false        | url                                                         | URL to redirect user in case of success payment                                                                                                                                                                                                                   | https://merchant.com/success/{orderId}                                                                                                                                                                                             |
| cancelURL       | string     | false        | url                                                         | URL to redirect user in case payment was canceled                                                                                                                                                                                                                 | https://merchant.com/cancel/{orderId}                                                                                                                                                                                              |
| errorURL        | string     | false        | url                                                         | URL to redirect user in case of error                                                                                                                                                                                                                             | https://merchant.com/error/{orderId}                                                                                                                                                                                               |
| **extKycInfo**  | **object** | **false**    | **-**                                                       | **User KYC info**                                                                                                                                                                                                                                                 | **-**                                                                                                                                                                                                                              |
| firstName       | string     | false        | <p>maxLength: 100<br>minLength: 0</p>                       | User’s first name                                                                                                                                                                                                                                                 | <p>"extKycInfo": {<br>        "firstName": "TestName",<br>        "lastName": "TestLastName",<br>        "middleName": "TestMiddleName",<br>        "birthdate": "21.02.1988",<br>        "nationality": "Azerbaijan"<br>    }</p> |
| lastName        | string     | false        | <p>maxLength: 100<br>minLength: 0</p>                       | User’s last name                                                                                                                                                                                                                                                  |                                                                                                                                                                                                                                    |
| middleName      | string     | false        | <p>maxLength: 100<br>minLength: 0</p>                       | User’s middle name                                                                                                                                                                                                                                                |                                                                                                                                                                                                                                    |
| birthdate       | string     | false        | <p>maxLength: 100<br>minLength: 0</p>                       | User’s birthdate                                                                                                                                                                                                                                                  |                                                                                                                                                                                                                                    |
| nationality     | string     | false        | <p>maxLength: 100<br>minLength: 0</p>                       | User’s nationality                                                                                                                                                                                                                                                |                                                                                                                                                                                                                                    |
| userdata        | string     | false        | -                                                           | Any userdata, not parsed on m10 side, returned unchanged in callback                                                                                                                                                                                              | -                                                                                                                                                                                                                                  |
| **metadata**    | **object** | **false**    | **-**                                                       | **Additional parameters**                                                                                                                                                                                                                                         | **-**                                                                                                                                                                                                                              |
| additionalProp1 | string     | false        | -                                                           | Additional merchant specific parameters                                                                                                                                                                                                                           |                                                                                                                                                                                                                                    |

**Request example:**

```url
curl -X 'POST' \
  'https://develop.m10payments.com/online-acquiring/api/v1/orders/actions/create-payment' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>' \
  -H 'X-User-Token: b8ec7167-e408-4211-8deb-ae38e65d281a' \
  -H 'X-User-Locale: en-GB' \
  -H 'X-User-Tokenization: NOT_REQUIRED' \
  -H 'Content-Type: application/json' \
  -d '{
  "orderId": "upLZGwJhAJGdxTlwoF89v98YLDFWNFPi",
  "currencyISO": "AZN",
  "amount": "10.51",
  "confirmURL": "https://merchant.com/success/{orderId}",
  "cancelURL": "https://merchant.com/cancel/{orderId}",
  "errorURL": "https://merchant.com/error/{orderId}",
  extKycInfo": {
        "firstName": "TestName",
        "lastName": "TestLastName",
        "middleName": "TestMiddleName",
        "birthdate": "21.02.1988",
        "nationality": "Azerbaijan"
    },
  "userdata": "string",
  "metadata": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}'
```

**Response example**

_**Code:**_ 200 OK

_**Body:**_

```json
{
  "paymentURL": "https://link.develop.m10payments.com/?link=https://develop.m10payments.com/
  acquiring?operationId=de039a25-fd0d-44a5-9969-8d504ba4c4e8&apn=com.m10&isi=1642308007&ibi=
  com.m10.ios",
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Description**

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                      | **Example**                                                                                                                                                            |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| paymentURL    | string   | true         | url                                                         | <p>Response to merchant for create payment on m10.<br>Includes URL for merchant to show m10 payment page to user</p> | https://link.develop.m10payments.com/?link=https://api.m10.az/acquiring?operationId=c8b1a4fa-e846-42d8-a53f-76d62d14f1d6\&apn=com.m10\&isi=1642308007\&ibi=com.m10.ios |
| transactionId | string   | true         | uuid                                                        | Id of successful payment/refund operation obtained from m10                                                          | 3fa85f64-5717-4562-b3fc-2c963f66afa6                                                                                                                                   |

***

### 2. Create a refund

<mark style="color:blue;">**POST**</mark> `/api/v1/orders/actions/create-refund`

Creates a refund operation on m10. The field originalTransactionId from the body of the request is same to transactionId from the payment request response.

**Request parameters:**

_**Header**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                  | **Example**           |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | -------------------------------- | --------------------- |
| Authorization | string   | true         | ^Bearer \[a-zA-Z0-9\_-\|]{5,24}:\[a-zA-Z0-9-]{36}           | Auth header with Bearer \<token> | Bearer \<AccessToken> |

_**Body**_

| **Name**              | **Type**   | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                                   | **Example**                          |
| --------------------- | ---------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| orderId               | string     | true         | \[a-zA-Z0-9\_-]{20,64}                                      | <p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | ktkKscy0WJCbY3ghZcQAu-eOFq           |
| originalTransactionId | string     | true         | uuid                                                        | Unique ID of the original transaction in m10 in case of refund                                                                                                                                                                                                    | 3fa85f64-5717-4562-b3fc-2c963f66afa6 |
| currencyISO           | string     | true         | <p>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</p>         | Currency ISO of the transaction                                                                                                                                                                                                                                   | AZN                                  |
| amount                | string     | true         | <p>"maximum":<br>1000000000 "minimum":<br>1</p>             | Amount to refund. Can be less than total payment amount. Sum of all refunds for the payment must not exceed total payment amount.                                                                                                                                 | 10.51                                |
| userdata              | string     | false        | -                                                           | Any userdata, not parsed on m10 side, returned unchanged in callback                                                                                                                                                                                              | -                                    |
| **metadata**          | **object** | **false**    | **-**                                                       | **Additional parameters**                                                                                                                                                                                                                                         | **-**                                |
| additionalProp1       | string     | false        | -                                                           | Additional merchant specific parameters                                                                                                                                                                                                                           |                                      |

**Request example:**

```url
curl -X 'POST' \
  'https://develop.m10payments.com/online-acquiring/api/v1/orders/actions/create-refund' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>' \
  -H 'Content-Type: application/json' \
  -d '{
  "orderId": "q9NJQhwEd-_6_90SxJhPFjc-Ze7SI7TZn",
  "originalTransactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "currencyISO": "AZN",
  "amount": 10.51,
  "userdata": "string",
  "metadata": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}'
```

**Response example**

_**Code:**_ 200 OK

_**Body:**_

```json
{
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Description**

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                             | **Example**                          |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------ |
| transactionId | string   | true         | uuid                                                        | Id of successful payment/refund operation obtained from m10 | 3fa85f64-5717-4562-b3fc-2c963f66afa6 |

***

### 2. Create a refund

<mark style="color:blue;">**POST**</mark> `/api/v1/orders/actions/create-refund`

Creates a refund operation on m10. The field originalTransactionId from the body of the request is same to transactionId from the payment request response.

**Request parameters:**

_**Header**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                  | **Example**           |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | -------------------------------- | --------------------- |
| Authorization | string   | true         | ^Bearer \[a-zA-Z0-9\_-\|]{5,24}:\[a-zA-Z0-9-]{36}           | Auth header with Bearer \<token> | Bearer \<AccessToken> |

_**Body**_

| **Name**              | **Type**   | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                                   | **Example**                          |
| --------------------- | ---------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| orderId               | string     | true         | \[a-zA-Z0-9\_-]{20,64}                                      | <p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | ktkKscy0WJCbY3ghZcQAu-eOFq           |
| originalTransactionId | string     | true         | uuid                                                        | Unique ID of the original transaction in m10 in case of refund                                                                                                                                                                                                    | 3fa85f64-5717-4562-b3fc-2c963f66afa6 |
| currencyISO           | string     | true         | <p>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</p>         | Currency ISO of the transaction                                                                                                                                                                                                                                   | AZN                                  |
| amount                | string     | true         | <p>"maximum":<br>1000000000 "minimum":<br>1</p>             | Amount to refund. Can be less than total payment amount. Sum of all refunds for the payment must not exceed total payment amount.                                                                                                                                 | 10.51                                |
| userdata              | string     | false        | -                                                           | Any userdata, not parsed on m10 side, returned unchanged in callback                                                                                                                                                                                              | -                                    |
| **metadata**          | **object** | **false**    | **-**                                                       | **Additional parameters**                                                                                                                                                                                                                                         | **-**                                |
| additionalProp1       | string     | false        | -                                                           | Additional merchant specific parameters                                                                                                                                                                                                                           |                                      |

**Request example:**

```url
curl -X 'POST' \
  'https://develop.m10payments.com/online-acquiring/api/v1/orders/actions/create-refund' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>' \
  -H 'Content-Type: application/json' \
  -d '{
  "orderId": "q9NJQhwEd-_6_90SxJhPFjc-Ze7SI7TZn",
  "originalTransactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "currencyISO": "AZN",
  "amount": 10.51,
  "userdata": "string",
  "metadata": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}'
```

**Response example**

_**Code:**_ 200 OK

_**Body:**_

```json
{
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Description**

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                             | **Example**                          |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------ |
| transactionId | string   | true         | uuid                                                        | Id of successful payment/refund operation obtained from m10 | 3fa85f64-5717-4562-b3fc-2c963f66afa6 |

***

### 3. Get order details

<mark style="color:green;">**GET**</mark> `/api/v1/orders/{orderId}`

Allows to get an operation details from m10 within the first three days.

This endpoint allows to retrieve operation details and status within **three days** of the operation's completion.\
In the current implementation, the orderId is stored for three days in our database before being archived. To retrieve operation details after this period, use the **GET /api/v1/transactions/{transactionId} method**.\
You can get transactionId from the payment creation response.

**Request parameters:**

_**Header**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                  | **Example**           |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | -------------------------------- | --------------------- |
| Authorization | string   | true         | ^Bearer \[a-zA-Z0-9\_-\|]{5,24}:\[a-zA-Z0-9-]{36}           | Auth header with Bearer \<token> | Bearer \<AccessToken> |

_**Path variable**_

| **Name** | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                            | **Example**                |
| -------- | -------- | ------------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- |
| orderId  | string   | true         | \[a-zA-Z0-9\_-]{20,64}                                      | <p>orderId supplied by merchant in the initial request</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | ktkKscy0WJCbY3ghZcQAu-eOFq |

**Request example:**

```url
curl -X 'GET' \
  'https://develop.m10payments.com/online-acquiring/api/v1/online-acquiring/orders/
  ktkKscy0WJCbY3ghZcQAu-eOFq' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>'
```

**Response example**

_**Code:**_ 200 OK

```json
{
  "amount": "10.51",
  "currencyISO": "AZN",
  "metadata": null,
  "netAmount": "10.3",
  "orderId": "ktkKscy0WJCbY3ghZcQAu-eOFq",
  "originalTransactionId": null,
  "status": "FAILURE",
  "transactionId": "de039a25-fd0d-44a5-9969-8d504ba4c4e8",
  "transactionType": "PAYMENT",
  "userToken": null,
  "userdata": "string"
}
```

**Description**

| **Name**              | **Type**   | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p>                | **Description**                                                                                                                                                                                                                                                   | **Example**                          |
| --------------------- | ---------- | ------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ |
| amount                | string     | false        | <p>"maximum":<br>1000000000 "minimum":<br>1</p>                            | <p>Transaction amount.<br><em><strong>In case of purchase:</strong></em> amount paid by user.<br><em><strong>In case of refund:</strong></em> amount paid by merchant.</p>                                                                                        | 10.51                                |
| currencyISO           | string     | true         | <p>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</p>                        | Currency ISO of the transaction                                                                                                                                                                                                                                   | AZN                                  |
| **metadata**          | **object** | **false**    | **-**                                                                      | **Additional parameters**                                                                                                                                                                                                                                         | **-**                                |
| additionalProp1       | string     | false        | -                                                                          | Additional merchant specific parameters                                                                                                                                                                                                                           |                                      |
| netAmount             | string     | false        | <p>"maximum":<br>1000000000 "minimum":<br>1</p>                            | <p>Final amount received.<br><em><strong>In case of purchase:</strong></em> amount paid to merchant.<br><em><strong>In case of refund:</strong></em> amount paid to user.</p>                                                                                     | 10.3                                 |
| orderId               | string     | true         | \[a-zA-Z0-9\_-]{20,64}                                                     | <p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | ktkKscy0WJCbY3ghZcQAu-eOFq           |
| originalTransactionId | string     | false        | uuid                                                                       | Unique ID of the original transaction in m10 in case of refund                                                                                                                                                                                                    | null                                 |
| status                | string     | false        | <p>enum:<br>"CREATED”, “IN_PROGRESS”, “SUCCESS”, “CANCEL"<br>“FAILURE"</p> | Order status in m10                                                                                                                                                                                                                                               | FAILURE                              |
| transactionId         | string     | false        | uuid                                                                       | Unique ID of the processed transaction in m10. Null if order was canceled or not processed yet                                                                                                                                                                    | de039a25-fd0d-44a5-9969-8d504ba4c4e8 |
| transactionType       | string     | true         | <p>enum:<br>”PAYMENT”, “REFUND”<br>“TOKENIZATION”</p>                      | Type of transaction                                                                                                                                                                                                                                               | PAYMENT                              |
| userToken             | string     | false        | uuid                                                                       | Token for quick payment with saved m10 payment method when X-User-Tokenization header has value 'REQUIRED'                                                                                                                                                        | null                                 |
| userdata              | string     | false        | -                                                                          | Any userdata previously supplied by the merchant, not parsed on m10 side, returned unchanged                                                                                                                                                                      | -                                    |

***

### 4. Get transaction details

<mark style="color:green;">**GET**</mark> `/api/v1/transactions/{transactionId}`

Allows to get an operation details from m10 after the three days.

**Request parameters:**

_**Header**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                  | **Example**           |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | -------------------------------- | --------------------- |
| Authorization | string   | true         | ^Bearer \[a-zA-Z0-9\_-\|]{5,24}:\[a-zA-Z0-9-]{36}           | Auth header with Bearer \<token> | Bearer \<AccessToken> |

_**Path variable**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                | **Example**                          |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------ |
| transactionId | string   | false        | uuid                                                        | Unique ID of the processed transaction in m10. Null if order was canceled or not processed yet | 3fa85f64-5717-4562-b3fc-2c963f66afa6 |

**Request example:**

```url
curl -X 'GET' \
  'https://develop.m10payments.com/online-acquiring/api/v1/online-acquiring/transactions/
  3fa85f64-5717-4562-b3fc-2c963f66afa6' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>'
```

**Response example**

_**Code:**_ 200 OK

```json
{
  "orderId": "kIC1IAznDeaeHC-7M2Kg1iLV5mY3yyMJo6PPrNDcA4R5QzcwWJ1CR6NEbmNUIAxd",
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "originalTransactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "transactionType": "PAYMENT",
  "currencyISO": "AZN",
  "amount": 10.51,
  "netAmount": 10.3,
  "status": "CREATED",
  "createdAt": "2024-02-14T11:57:38.285Z",
  "userdata": "string",
  "metadata": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

**Description**

| **Name**              | **Type**   | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p>                | **Description**                                                                                                                                                                                                                                                   | **Example**                                                      |
| --------------------- | ---------- | ------------ | -------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| orderId               | string     | true         | \[a-zA-Z0-9\_-]{20,64}                                                     | <p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | kIC1IAznDeaeHC-7M2Kg1iLV5mY3yyMJo6PPrNDcA4R5QzcwWJ1CR6NEbmNUIAxd |
| transactionId         | string     | false        | uuid                                                                       | Unique ID of the processed transaction in m10.                                                                                                                                                                                                                    | 3fa85f64-5717-4562-b3fc-2c963f66afa6                             |
| originalTransactionId | string     | false        | uuid                                                                       | Unique ID of the original transaction in m10 in case of refund                                                                                                                                                                                                    | 3fa85f64-5717-4562-b3fc-2c963f66afa6                             |
| transactionType       | string     | true         | <p>enum:<br>”PAYMENT”, “REFUND”, “TOKENIZATION”</p>                        | Type of transaction                                                                                                                                                                                                                                               | PAYMENT                                                          |
| currencyISO           | string     | true         | <p>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</p>                        | CurrencyISO of the transaction                                                                                                                                                                                                                                    | AZN                                                              |
| amount                | string     | false        | <p>"maximum":<br>1000000000 "minimum":<br>1</p>                            | <p>Transaction amount.<br><em><strong>In case of purchase:</strong></em> amount paid by user.<br><em><strong>In case of refund:</strong></em> amount paid by merchant.</p>                                                                                        | 10.51                                                            |
| netAmount             | string     | false        | <p>"maximum":<br>1000000000 "minimum":<br>1</p>                            | <p>Final amount received.<br><em><strong>In case of purchase:</strong></em> amount paid to merchant.<br><em><strong>In case of refund:</strong></em> amount paid to user.</p>                                                                                     | 10.3                                                             |
| status                | string     | false        | <p>enum:<br>"CREATED”, “IN_PROGRESS”, “SUCCESS”, “CANCEL"<br>“FAILURE"</p> | Order status in m10                                                                                                                                                                                                                                               | CREATED                                                          |
| createdAt             | string     | true         | date-time                                                                  | Moment when transaction was created                                                                                                                                                                                                                               | 2024-02-14T11:57:38.285Z                                         |
| userToken             | string     | false        | uuid                                                                       | Token for quick payment with saved m10 payment method when X-User-Tokenization header has value 'REQUIRED'                                                                                                                                                        | 3fa85f64-5717-4562-b3fc-2c963f66afa6                             |
| userdata              | string     | false        | -                                                                          | Any additional userdata                                                                                                                                                                                                                                           | -                                                                |
| **metadata**          | **object** | **false**    | **-**                                                                      | **Additional parameters**                                                                                                                                                                                                                                         | **-**                                                            |
| additionalProp1       | string     | false        | -                                                                          | -//-                                                                                                                                                                                                                                                              |                                                                  |

***

### 5. Create a tokenization on m10

<mark style="color:blue;">**POST**</mark> `/api/v1/user-tokenization/actions/create`

Creates a tokenization on m10. Returns on response a dynamic link tokenizationUrl to redirect the user and operationId of the completed operation on m10.

**Request parameters:**

_**Header**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                | **Example**           |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | --------------------- |
| Authorization | string   | true         | ^Bearer \[a-zA-Z0-9\_-\|]{5,24}:\[a-zA-Z0-9-]{36}           | Auth header with Bearer \<token>                                                               | Bearer \<AccessToken> |
| X-User-Locale | string   | false        | -                                                           | Set of parameters that defines the user's language, region and any special variant preferences | en-GB                 |

_**Body**_

| **Name**        | **Type**   | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                                   | **Example**                                                      |
| --------------- | ---------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| orderId         | string     | true         | \[a-zA-Z0-9\_-]{20,64}                                      | <p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | kIC1IAznDeaeHC-7M2Kg1iLV5mY3yyMJo6PPrNDcA4R5QzcwWJ1CR6NEbmNUIAxd |
| confirmURL      | string     | false        | url                                                         | Dynamic link or URL to redirect user if tokenization was successful                                                                                                                                                                                               | https://merchant.com/success/{tokenizationId}                    |
| cancelURL       | string     | false        | url                                                         | Dynamic link or URL to redirect user if tokenization was canceled by user                                                                                                                                                                                         | https://merchant.com/cancel/{tokenizationId}                     |
| errorURL        | string     | false        | url                                                         | Dynamic link or URL to redirect user if there was an error during tokenization process                                                                                                                                                                            | https://merchant.com/error/{tokenizationId}                      |
| userdata        | string     | false        | -                                                           | Any userdata, not parsed on m10 side, returned unchanged in callback                                                                                                                                                                                              | -                                                                |
| **metadata**    | **object** | **false**    | **-**                                                       | **Additional parameters**                                                                                                                                                                                                                                         | **-**                                                            |
| additionalProp1 | string     | false        | -                                                           | Additional merchant specific parameters                                                                                                                                                                                                                           |                                                                  |

**Request example:**

```url
curl -X 'POST' \
  'https://.../acquiring/api/v1/user-tokenization/actions/create' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>' \
  -H 'X-User-Locale: en-GB' \
  -H 'Content-Type: application/json' \
  -d '{
  "orderId": "LFMxfXQModI1uu5isDypCgmZGIhPjEYMqts1Q1Rr0Whw3zCMbVsj_H",
  "confirmURL": "https://merchant.com/success/{tokenizationId}",
  "cancelURL": "https://merchant.com/cancel/{tokenizationId}",
  "errorURL": "https://merchant.com/error/{tokenizationId}",
  "userdata": "string",
  "metadata": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}'
```

**Response example**

_**Code:**_ 200 OK

```json
{
  "tokenizationUrl": "https://api.m10.az/online-payments/tokenization/{tokenizationId}",
  "operationId": "c8b1a4fa-e846-42d8-a53f-76d62d14f1d6"
}
```

**Description**

| **Name**        | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                         | **Example**                                                      |
| --------------- | -------- | ------------ | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| tokenizationUrl | string   | true         | url                                                         | URL for merchant to show m10 tokenization page to user                                                                                                                                                                                                  | https://api.m10.az/online-payments/tokenization/{tokenizationId} |
| operationId     | string   | false        | \[a-zA-Z0-9\_-]{20,64}                                      | <p>Operation id of the created tokenization request</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | _c8b1a4fa-e846-42d8-a53f-76d62d14f1d6_                           |

The token will be provided in Callback, but there is an additional endpoint to get token details by using a method **GET /api/v1/user-tokenization/{tokenizationId}**.

The tokenizationId is identical to the orderId from the create tokenization or create payment request.

{% hint style="warning" %}
The token will be provided in Callback, but there is an additional endpoint to get token details by using a method **GET /api/v1/user-tokenization/{tokenizationId}**.

The tokenizationId is identical to the orderId from the create tokenization or create payment request.
{% endhint %}



***

### 6. Get a token information

<mark style="color:green;">**GET**</mark> `/api/v1/user-tokenization/{tokenizationId}`

Allows to get the user token from m10. tokenizationId is same to operationId from the create token response.



**Request parameters:**

_**Header**_

| **Name**      | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                  | **Example**           |
| ------------- | -------- | ------------ | ----------------------------------------------------------- | -------------------------------- | --------------------- |
| Authorization | string   | true         | ^Bearer \[a-zA-Z0-9\_-\|]{5,24}:\[a-zA-Z0-9-]{36}           | Auth header with Bearer \<token> | Bearer \<AccessToken> |

_**Path variable**_

| **Name**       | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                | **Example**                          |
| -------------- | -------- | ------------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------ |
| tokenizationId | string   | false        | uuid                                                        | Unique ID of the processed transaction in m10. Null if order was canceled or not processed yet | 3fa85f64-5717-4562-b3fc-2c963f66afa6 |

**Request example:**

```url
curl -X 'GET' \
  'https://...acquiring/api/v1/user-tokenization/LFMxfXQModI1uu5isDypCgmZGIhPjEYMqts1Q1Rr0Whw3zC
  MbVsj_H' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>'
```

**Response example**

_**Code:**_ 200 OK

```json
{
  "id": "fi1Z2FhZajd7ROD4rtovEpNaLUAC6xR-t_cmjWVxPjF6NDODafaXvBtzQK_-4o",
  "userToken": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

**Description**

| **Name**  | **Type** | **Required** | <p><strong>Pattern/</strong><br><strong>Format</strong></p> | **Description**                                                                                                                                                                                                                                                         | **Example**                                                      |
| --------- | -------- | ------------ | ----------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Id        | string   | true         | \[a-zA-Z0-9\_-]{20,64}                                      | <p>Unique tokenization id for this merchant provided by the merchan</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p> | fi1Z2FhZajd7ROD4rtovEpNaLUAC6xR-t\_cmjWVxPjF6NDODafaXvBtzQK\_-4o |
| userToken | string   | false        | uuid                                                        | Token for quick payment with saved m10 payment method                                                                                                                                                                                                                   | 3fa85f64-5717-4562-b3fc-2c963f66afa6                             |

***

### **Callback structure**

There are two types of callbacks m10 uses during the integration:

#### **Callback with order details after the successful operation (payment or refund)**

After merchant provides a URL, the static URL will be stored in our database during registration. This URL is used to notify merchant’s backend system of successful or erroneous payment statuses. It must be a POST method and accept the same request format as our GET /api/v1/orders/{orderId} endpoint.\
Also contains the \`X-HMAC\` and \`X-Nonce\` headers. We will send a signing key for HMAC validation at the merchant's request.\
&#xNAN;_&#x45;xample of the callback:_

```json
{
  "orderId": "c2U1-rLMOvxg_no0-x67v5K_A2XwWco1-2aSCnAm6Af_S6gFJ0V5oWX6",
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "originalTransactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "transactionType": "PAYMENT",
  "status": "CREATED",
  "currencyISO": "AZN",
  "amount": 1051,
  "netAmount": 1030,
  "userToken": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "userdata": "string",
  "metadata": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

**Important**:

* The callback should be configured using the **POST method**.
* **X-Nonce** is a random string, it is necessary to check that it is not repeated, i.e. each message contains a unique value in this field. It’s generated by m10 for each request.
* **X-HMAC** is a hash of the response body in **hex** format.\
  The key will be provided by m10.\
  The algorithm is HmacSHA256.

<pre><code>{% hint style="warning" %}
**Warning hints**

- The callback should be configured using the POST method.
- X-Nonce is a random string, it is necessary to check that it is not repeated, i.e. each message contains a unique value in this field. It’s generated by m10 for each request.
<strong>- X-HMAC is a hash of the response body in hex format.
</strong><strong>The key will be provided by m10.
</strong>The algorithm is HmacSHA256.

{% endhint %}
</code></pre>

{% hint style="warning" %}
**Important**

* The callback should be configured using the **POST method**.
* **X-Nonce** is a random string, it is necessary to check that it is not repeated, i.e. each message contains a unique value in this field. It’s generated by m10 for each request.
* **X-HMAC** is a hash of the response body in **hex** format.\
  The key will be provided by m10.\
  The algorithm is HmacSHA256.
{% endhint %}

#### **Callback with the token**

After the tokenization process, a callback with the token is triggered. This callback contains the same information as the response from the request GET /api/v1/user-tokenization/{tokenizationId}\
&#xNAN;_&#x45;xample of the callback:_

```json
{
  "id": "CrAYldtUbo86oJDObxnmX17Y3lnhZBOkA9qDdYsRC3GlBMjUsAanavNbIeeTZFYg",
  "userToken": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```

***

### FAQ

1. m10 will provide the **API keys** for both test and production environments.
2. Merchants can use the provided **HMAC\_KEY** to verify the authenticity of messages/requests from m10. The key is unique per merchant.
3.  Use the domains below to test the API:\
    **Test env** - https://develop.m10payments.com/online-acquiring/....\
    **Prod env** - https://api.m10.az/acquiring/....\
    \
    &#xNAN;_&#x48;ere is the example of the payment request on the **Prod env**:_

    ```url
    curl --location 'https://api.m10.az/acquiring/api/v1/orders/actions/create-payment' \
    --header 'Authorization: Bearer token:token_id' \
    --header 'X-User-Tokenization: NOT_REQUIRED' \
    --header 'Content-Type: application/json' \
    --header 'Cookie: __ddg1_=S26256mepmhZLXlD3r6h' \
    --data '{
      "orderId": "45e4758c-ffd1-4bf3-8fa3-5ad983cbc141",
      "currencyISO": "AZN",
      "amount": "1.90",
       "userdata": "string",
        "extKycInfo": {
            "firstName": "TestName",
            "lastName": "TestLastName",
            "middleName": "TestMiddleName",
            "birthdate": "21.02.1988",
            "nationality": "Azerbaijan"
        }
    }'
    ```
4. m10 works with **two types of URLs**:\
   **a)** Three URLs in the request body of the create payment/token endpoint, specifically confirmURL, cancelURL, and errorURL. These are used to redirect the user once an operation is completed, cancelled or there is an error. You may provide unique URLs for each operation/request.\
   **b)** a static URL for your backend system, which we store in our database during the beginning of the integration. This URL is used to notify your backend system of successful or erroneous operation statuses. It must be a **POST method** and accept the same request format as our GET /api/v1/orders/{orderId} endpoint.\
   So:\
   \- the URLs from the step _**“a”**_ are to redirect users. Merchant will specify them per each request.\
   \- the URL from the step _**“b”**_ is to notify merchant backend about the operation status. Merchant can give us the URL once and we will store it in our system.
5. To ensure that the merchant's backend receives callbacks for both successful and unsuccessful operations from m10, the merchant needs to provide the static callback URL. This URL will be saved in our system for future use.
6. To retrieve the operation details (status, token info or any other information) use the below methods:\
   **a)** GET /api/v1/orders/{orderId} - within the three days after the operation is created.\
   **b)** GET /api/v1/transactions/{transactionId} - to check the operation details after three days.
7. The orderId should be unique for each request and is generated by the merchant.
8. The field originalTransactionId from a refund request is same to transactionId from the payment request response.
