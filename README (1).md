---
layout:
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 🇬🇧 m10 e-commerce API Description

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

<table data-full-width="true"><thead><tr><th width="418">Environment</th><th>URL</th></tr></thead><tbody><tr><td>Test environment</td><td><code>https://develop.m10payments.com/online-acquiring/...</code></td></tr><tr><td>Prod environment</td><td><code>https://api.m10.az/acquiring/...</code></td></tr></tbody></table>

### Endpoints

<table data-full-width="true"><thead><tr><th>HTTP Method</th><th>Endpoint</th><th>Description</th></tr></thead><tbody><tr><td>POST</td><td><code>/api/v1/orders/actions/create-payment</code></td><td>Create payment</td></tr><tr><td>POST</td><td><code>/api/v1/orders/actions/create-refund</code></td><td>Create refund</td></tr><tr><td>GET</td><td><code>/api/v1/orders/{orderId}</code></td><td>Retrieve order information</td></tr><tr><td>GET</td><td><code>/api/v1/transactions/{transactionId}</code></td><td>Retrieve transaction information</td></tr><tr><td>POST</td><td><code>/api/v1/user-tokenization/actions/create</code></td><td>Create tokenization</td></tr><tr><td>GET</td><td><code>/api/v1/user-tokenization/{tokenizationId}</code></td><td>Get token information</td></tr></tbody></table>

***

### Status codes

HTTP response codes are used to indicate general classes of success and error.&#x20;

#### Success Code

<table data-full-width="true"><thead><tr><th width="318">Status Code</th><th>Description</th></tr></thead><tbody><tr><td>200</td><td>Successfully processed request</td></tr></tbody></table>

### Error Codes

Error responses contain more detail about the error in the `x-error-code (header)` and `description` properties.

<table data-header-hidden data-full-width="true"><thead><tr><th width="144"></th><th width="279"></th><th></th></tr></thead><tbody><tr><td>Status Code</td><td><code>x-error-code</code> (header)</td><td>Description</td></tr><tr><td>400</td><td>absent</td><td>No <code>x-error-code</code> since these cases are not processed yet</td></tr><tr><td>401</td><td><code>onlineAcquiring-401001</code></td><td>Unauthorized</td></tr><tr><td>401</td><td><code>onlineAcquiring-401002</code></td><td>Invalid token data</td></tr><tr><td>403</td><td><code>onlineAcquiring-403001</code></td><td>Authentication data mismatch</td></tr><tr><td>409</td><td><code>onlineAcquiring-409001</code></td><td>System already has an order with the same ID but different data</td></tr><tr><td>409</td><td><code>onlineAcquiring-409002</code></td><td>System is already processing this order</td></tr><tr><td>500</td><td><code>onlineAcquiring-500001</code></td><td>Internal server error</td></tr><tr><td>504</td><td>absent</td><td>No <code>x-error-code</code> since the error occurred on the nginx server</td></tr></tbody></table>

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

<table data-full-width="true"><thead><tr><th width="153">Name</th><th width="89">Type</th><th width="81">Required</th><th width="156">Pattern/Format</th><th width="285">Description</th><th>Example</th></tr></thead><tbody><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer\s[a-zA-Z0-9_.-]{5,128}$</td><td>Auth header with Bearer <code>&#x3C;token></code></td><td><code>Bearer &#x3C;AccessToken></code></td></tr><tr><td>X-User-Token</td><td>string</td><td>false</td><td>[a-zA-Z0-9_.-]{20,64}</td><td>Header with token when user has m10 as saved payment method.</td><td><code>8e17cc7f-408b-4211-bd6e-aa38e5d6281a</code></td></tr><tr><td>X-User-Locale</td><td>string</td><td>false</td><td>-</td><td>Set of parameters that define the user's language, region, and any special variant preferences.</td><td><code>en-GB</code></td></tr><tr><td>X-User-Tokenization</td><td>string</td><td>true</td><td><code>PROVIDED</code>, <code>REQUIRED</code>, <code>NOT_REQUIRED</code></td><td>User token processing mode, when a user wants to save m10 for future payments.</td><td><code>PROVIDED</code></td></tr></tbody></table>

***

**Body**

<table data-header-hidden data-full-width="true"><thead><tr><th width="145"></th><th width="100"></th><th width="106"></th><th></th><th width="249"></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>upLZGwJhAJGdxTlwoF89v98YLDFWNFPi</td></tr><tr><td>currencyISO</td><td>string</td><td>true</td><td>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</td><td>Currency ISO of the transaction</td><td>AZN</td></tr><tr><td>amount</td><td>string</td><td>true</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Amount of the transaction</td><td>10.51</td></tr><tr><td>confirmURL</td><td>string</td><td>false</td><td>url</td><td>URL to redirect user in case of success payment</td><td>https://merchant.com/success/{orderId}</td></tr><tr><td>cancelURL</td><td>string</td><td>false</td><td>url</td><td>URL to redirect user in case payment was canceled</td><td>https://merchant.com/cancel/{orderId}</td></tr><tr><td>errorURL</td><td>string</td><td>false</td><td>url</td><td>URL to redirect user in case of error</td><td>https://merchant.com/error/{orderId}</td></tr><tr><td><strong>extKycInfo</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>User KYC info</strong></td><td><strong>-</strong></td></tr><tr><td>firstName</td><td>string</td><td>false</td><td>maxLength: 100<br>minLength: 0</td><td>User’s first name</td><td>"extKycInfo": {<br>        "firstName": "TestName",<br>        "lastName": "TestLastName",<br>        "middleName": "TestMiddleName",<br>        "birthdate": "21.02.1988",<br>        "nationality": "Azerbaijan"<br>    }</td></tr><tr><td>lastName</td><td>string</td><td>false</td><td>maxLength: 100<br>minLength: 0</td><td>User’s last name</td><td></td></tr><tr><td>middleName</td><td>string</td><td>false</td><td>maxLength: 100<br>minLength: 0</td><td>User’s middle name</td><td></td></tr><tr><td>birthdate</td><td>string</td><td>false</td><td>maxLength: 100<br>minLength: 0</td><td>User’s birthdate</td><td></td></tr><tr><td>nationality</td><td>string</td><td>false</td><td>maxLength: 100<br>minLength: 0</td><td>User’s nationality</td><td></td></tr><tr><td>userdata</td><td>string</td><td>false</td><td>-</td><td>Any userdata, not parsed on m10 side, returned unchanged in callback</td><td>-</td></tr><tr><td><strong>metadata</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>Additional parameters</strong></td><td><strong>-</strong></td></tr><tr><td>additionalProp1</td><td>string</td><td>false</td><td>-</td><td>Additional merchant specific parameters</td><td></td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
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
{% endcode %}

**Response example**

_**Code:**_ 200 OK

_**Body:**_

{% code fullWidth="true" %}
```json
{
  "paymentURL": "https://link.develop.m10payments.com/?link=https://develop.m10payments.com/
  acquiring?operationId=de039a25-fd0d-44a5-9969-8d504ba4c4e8&apn=com.m10&isi=1642308007&ibi=
  com.m10.ios",
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
{% endcode %}

**Description**

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th width="90"></th><th width="105"></th><th width="117"></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>paymentURL</td><td>string</td><td>true</td><td>url</td><td>Response to merchant for create payment on m10.<br>Includes URL for merchant to show m10 payment page to user</td><td>https://link.develop.m10payments.com/?link=https://api.m10.az/acquiring?operationId=c8b1a4fa-e846-42d8-a53f-76d62d14f1d6&#x26;apn=com.m10&#x26;isi=1642308007&#x26;ibi=com.m10.ios</td></tr><tr><td>transactionId</td><td>string</td><td>true</td><td>uuid</td><td>Id of successful payment/refund operation obtained from m10</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr></tbody></table>

***

### 2. Create a refund

<mark style="color:blue;">**POST**</mark> `/api/v1/orders/actions/create-refund`

Creates a refund operation on m10. The field originalTransactionId from the body of the request is same to transactionId from the payment request response.

**Request parameters:**

_**Header**_

<table data-header-hidden data-full-width="true"><thead><tr><th width="149"></th><th width="99"></th><th width="108"></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer [a-zA-Z0-9_-|]{5,24}:[a-zA-Z0-9-]{36}</td><td>Auth header with Bearer &#x3C;token></td><td>Bearer &#x3C;AccessToken></td></tr></tbody></table>

_**Body**_

<table data-header-hidden data-full-width="true"><thead><tr><th width="212"></th><th width="89"></th><th width="82"></th><th width="187"></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>ktkKscy0WJCbY3ghZcQAu-eOFq</td></tr><tr><td>originalTransactionId</td><td>string</td><td>true</td><td>uuid</td><td>Unique ID of the original transaction in m10 in case of refund</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr><tr><td>currencyISO</td><td>string</td><td>true</td><td>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</td><td>Currency ISO of the transaction</td><td>AZN</td></tr><tr><td>amount</td><td>string</td><td>true</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Amount to refund. Can be less than total payment amount. Sum of all refunds for the payment must not exceed total payment amount.</td><td>10.51</td></tr><tr><td>userdata</td><td>string</td><td>false</td><td>-</td><td>Any userdata, not parsed on m10 side, returned unchanged in callback</td><td>-</td></tr><tr><td><strong>metadata</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>Additional parameters</strong></td><td><strong>-</strong></td></tr><tr><td>additionalProp1</td><td>string</td><td>false</td><td>-</td><td>Additional merchant specific parameters</td><td></td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
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
{% endcode %}

**Response example**

_**Code:**_ 200 OK

_**Body:**_

{% code fullWidth="true" %}
```json
{
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
{% endcode %}

**Description**

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>transactionId</td><td>string</td><td>true</td><td>uuid</td><td>Id of successful payment/refund operation obtained from m10</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr></tbody></table>

***

### 2. Create a refund

<mark style="color:blue;">**POST**</mark> `/api/v1/orders/actions/create-refund`

Creates a refund operation on m10. The field originalTransactionId from the body of the request is same to transactionId from the payment request response.

**Request parameters:**

_**Header**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer [a-zA-Z0-9_-|]{5,24}:[a-zA-Z0-9-]{36}</td><td>Auth header with Bearer &#x3C;token></td><td>Bearer &#x3C;AccessToken></td></tr></tbody></table>

_**Body**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>ktkKscy0WJCbY3ghZcQAu-eOFq</td></tr><tr><td>originalTransactionId</td><td>string</td><td>true</td><td>uuid</td><td>Unique ID of the original transaction in m10 in case of refund</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr><tr><td>currencyISO</td><td>string</td><td>true</td><td>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</td><td>Currency ISO of the transaction</td><td>AZN</td></tr><tr><td>amount</td><td>string</td><td>true</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Amount to refund. Can be less than total payment amount. Sum of all refunds for the payment must not exceed total payment amount.</td><td>10.51</td></tr><tr><td>userdata</td><td>string</td><td>false</td><td>-</td><td>Any userdata, not parsed on m10 side, returned unchanged in callback</td><td>-</td></tr><tr><td><strong>metadata</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>Additional parameters</strong></td><td><strong>-</strong></td></tr><tr><td>additionalProp1</td><td>string</td><td>false</td><td>-</td><td>Additional merchant specific parameters</td><td></td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
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
{% endcode %}

**Response example**

_**Code:**_ 200 OK

_**Body:**_

{% code fullWidth="true" %}
```json
{
  "transactionId": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
{% endcode %}

**Description**

<table data-header-hidden data-full-width="true"><thead><tr><th width="151"></th><th width="88"></th><th width="109"></th><th width="100"></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>transactionId</td><td>string</td><td>true</td><td>uuid</td><td>Id of successful payment/refund operation obtained from m10</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr></tbody></table>

***

### 3. Get order details

<mark style="color:green;">**GET**</mark> `/api/v1/orders/{orderId}`

Allows to get an operation details from m10 within the first three days.

This endpoint allows to retrieve operation details and status within **three days** of the operation's completion.\
In the current implementation, the orderId is stored for three days in our database before being archived. To retrieve operation details after this period, use the **GET /api/v1/transactions/{transactionId} method**.\
You can get transactionId from the payment creation response.

**Request parameters:**

_**Header**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer [a-zA-Z0-9_-|]{5,24}:[a-zA-Z0-9-]{36}</td><td>Auth header with Bearer &#x3C;token></td><td>Bearer &#x3C;AccessToken></td></tr></tbody></table>

_**Path variable**_

<table data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>orderId supplied by merchant in the initial request</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>ktkKscy0WJCbY3ghZcQAu-eOFq</td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
```url
curl -X 'GET' \
  'https://develop.m10payments.com/online-acquiring/api/v1/online-acquiring/orders/
  ktkKscy0WJCbY3ghZcQAu-eOFq' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>'
```
{% endcode %}

**Response example**

_**Code:**_ 200 OK

{% code fullWidth="true" %}
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
{% endcode %}

**Description**

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>amount</td><td>string</td><td>false</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Transaction amount.<br><em><strong>In case of purchase:</strong></em> amount paid by user.<br><em><strong>In case of refund:</strong></em> amount paid by merchant.</td><td>10.51</td></tr><tr><td>currencyISO</td><td>string</td><td>true</td><td>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</td><td>Currency ISO of the transaction</td><td>AZN</td></tr><tr><td><strong>metadata</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>Additional parameters</strong></td><td><strong>-</strong></td></tr><tr><td>additionalProp1</td><td>string</td><td>false</td><td>-</td><td>Additional merchant specific parameters</td><td></td></tr><tr><td>netAmount</td><td>string</td><td>false</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Final amount received.<br><em><strong>In case of purchase:</strong></em> amount paid to merchant.<br><em><strong>In case of refund:</strong></em> amount paid to user.</td><td>10.3</td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>ktkKscy0WJCbY3ghZcQAu-eOFq</td></tr><tr><td>originalTransactionId</td><td>string</td><td>false</td><td>uuid</td><td>Unique ID of the original transaction in m10 in case of refund</td><td>null</td></tr><tr><td>status</td><td>string</td><td>false</td><td>enum:<br>"CREATED”, “IN_PROGRESS”, “SUCCESS”, “CANCEL"<br>“FAILURE"</td><td>Order status in m10</td><td>FAILURE</td></tr><tr><td>transactionId</td><td>string</td><td>false</td><td>uuid</td><td>Unique ID of the processed transaction in m10. Null if order was canceled or not processed yet</td><td>de039a25-fd0d-44a5-9969-8d504ba4c4e8</td></tr><tr><td>transactionType</td><td>string</td><td>true</td><td>enum:<br>”PAYMENT”, “REFUND”<br>“TOKENIZATION”</td><td>Type of transaction</td><td>PAYMENT</td></tr><tr><td>userToken</td><td>string</td><td>false</td><td>uuid</td><td>Token for quick payment with saved m10 payment method when X-User-Tokenization header has value 'REQUIRED'</td><td>null</td></tr><tr><td>userdata</td><td>string</td><td>false</td><td>-</td><td>Any userdata previously supplied by the merchant, not parsed on m10 side, returned unchanged</td><td>-</td></tr></tbody></table>

***

### 4. Get transaction details

<mark style="color:green;">**GET**</mark> `/api/v1/transactions/{transactionId}`

Allows to get an operation details from m10 after the three days.

**Request parameters:**

_**Header**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer [a-zA-Z0-9_-|]{5,24}:[a-zA-Z0-9-]{36}</td><td>Auth header with Bearer &#x3C;token></td><td>Bearer &#x3C;AccessToken></td></tr></tbody></table>

_**Path variable**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>transactionId</td><td>string</td><td>false</td><td>uuid</td><td>Unique ID of the processed transaction in m10. Null if order was canceled or not processed yet</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
```url
curl -X 'GET' \
  'https://develop.m10payments.com/online-acquiring/api/v1/online-acquiring/transactions/
  3fa85f64-5717-4562-b3fc-2c963f66afa6' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>'
```
{% endcode %}

**Response example**

_**Code:**_ 200 OK

{% code fullWidth="true" %}
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
{% endcode %}

**Description**

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>kIC1IAznDeaeHC-7M2Kg1iLV5mY3yyMJo6PPrNDcA4R5QzcwWJ1CR6NEbmNUIAxd</td></tr><tr><td>transactionId</td><td>string</td><td>false</td><td>uuid</td><td>Unique ID of the processed transaction in m10.</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr><tr><td>originalTransactionId</td><td>string</td><td>false</td><td>uuid</td><td>Unique ID of the original transaction in m10 in case of refund</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr><tr><td>transactionType</td><td>string</td><td>true</td><td>enum:<br>”PAYMENT”, “REFUND”, “TOKENIZATION”</td><td>Type of transaction</td><td>PAYMENT</td></tr><tr><td>currencyISO</td><td>string</td><td>true</td><td>enum:<br>"AZN",<br>"USD",<br>"EUR",<br>"RUB"</td><td>CurrencyISO of the transaction</td><td>AZN</td></tr><tr><td>amount</td><td>string</td><td>false</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Transaction amount.<br><em><strong>In case of purchase:</strong></em> amount paid by user.<br><em><strong>In case of refund:</strong></em> amount paid by merchant.</td><td>10.51</td></tr><tr><td>netAmount</td><td>string</td><td>false</td><td>"maximum":<br>1000000000 "minimum":<br>1</td><td>Final amount received.<br><em><strong>In case of purchase:</strong></em> amount paid to merchant.<br><em><strong>In case of refund:</strong></em> amount paid to user.</td><td>10.3</td></tr><tr><td>status</td><td>string</td><td>false</td><td>enum:<br>"CREATED”, “IN_PROGRESS”, “SUCCESS”, “CANCEL"<br>“FAILURE"</td><td>Order status in m10</td><td>CREATED</td></tr><tr><td>createdAt</td><td>string</td><td>true</td><td>date-time</td><td>Moment when transaction was created</td><td>2024-02-14T11:57:38.285Z</td></tr><tr><td>userToken</td><td>string</td><td>false</td><td>uuid</td><td>Token for quick payment with saved m10 payment method when X-User-Tokenization header has value 'REQUIRED'</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr><tr><td>userdata</td><td>string</td><td>false</td><td>-</td><td>Any additional userdata</td><td>-</td></tr><tr><td><strong>metadata</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>Additional parameters</strong></td><td><strong>-</strong></td></tr><tr><td>additionalProp1</td><td>string</td><td>false</td><td>-</td><td>-//-</td><td></td></tr></tbody></table>

***

### 5. Create a tokenization on m10

<mark style="color:blue;">**POST**</mark> `/api/v1/user-tokenization/actions/create`

Creates a tokenization on m10. Returns on response a dynamic link tokenizationUrl to redirect the user and operationId of the completed operation on m10.

**Request parameters:**

_**Header**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer [a-zA-Z0-9_-|]{5,24}:[a-zA-Z0-9-]{36}</td><td>Auth header with Bearer &#x3C;token></td><td>Bearer &#x3C;AccessToken></td></tr><tr><td>X-User-Locale</td><td>string</td><td>false</td><td>-</td><td>Set of parameters that defines the user's language, region and any special variant preferences</td><td>en-GB</td></tr></tbody></table>

_**Body**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Unique order id for this merchant provided by the merchant</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td>kIC1IAznDeaeHC-7M2Kg1iLV5mY3yyMJo6PPrNDcA4R5QzcwWJ1CR6NEbmNUIAxd</td></tr><tr><td>confirmURL</td><td>string</td><td>false</td><td>url</td><td>Dynamic link or URL to redirect user if tokenization was successful</td><td>https://merchant.com/success/{tokenizationId}</td></tr><tr><td>cancelURL</td><td>string</td><td>false</td><td>url</td><td>Dynamic link or URL to redirect user if tokenization was canceled by user</td><td>https://merchant.com/cancel/{tokenizationId}</td></tr><tr><td>errorURL</td><td>string</td><td>false</td><td>url</td><td>Dynamic link or URL to redirect user if there was an error during tokenization process</td><td>https://merchant.com/error/{tokenizationId}</td></tr><tr><td>userdata</td><td>string</td><td>false</td><td>-</td><td>Any userdata, not parsed on m10 side, returned unchanged in callback</td><td>-</td></tr><tr><td><strong>metadata</strong></td><td><strong>object</strong></td><td><strong>false</strong></td><td><strong>-</strong></td><td><strong>Additional parameters</strong></td><td><strong>-</strong></td></tr><tr><td>additionalProp1</td><td>string</td><td>false</td><td>-</td><td>Additional merchant specific parameters</td><td></td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
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
{% endcode %}

**Response example**

_**Code:**_ 200 OK

{% code fullWidth="true" %}
```json
{
  "tokenizationUrl": "https://api.m10.az/online-payments/tokenization/{tokenizationId}",
  "operationId": "c8b1a4fa-e846-42d8-a53f-76d62d14f1d6"
}
```
{% endcode %}

**Description**

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>tokenizationUrl</td><td>string</td><td>true</td><td>url</td><td>URL for merchant to show m10 tokenization page to user</td><td>https://api.m10.az/online-payments/tokenization/{tokenizationId}</td></tr><tr><td>operationId</td><td>string</td><td>false</td><td>[a-zA-Z0-9_-]{20,64}</td><td><p>Operation id of the created tokenization request</p><p><em>Description of pattern:</em><br><em>letters: a-z, A-Z.</em><br><em>numbers: 0-9.</em><br><em>symbols:</em> “_” and “-”.<br><em>Min: 20 characters</em><br><em>Max: 64 characters</em></p></td><td><em>c8b1a4fa-e846-42d8-a53f-76d62d14f1d6</em></td></tr></tbody></table>

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

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>Authorization</td><td>string</td><td>true</td><td>^Bearer [a-zA-Z0-9_-|]{5,24}:[a-zA-Z0-9-]{36}</td><td>Auth header with Bearer &#x3C;token></td><td>Bearer &#x3C;AccessToken></td></tr></tbody></table>

_**Path variable**_

<table data-header-hidden data-full-width="true"><thead><tr><th></th><th></th><th></th><th></th><th></th><th></th></tr></thead><tbody><tr><td><strong>Name</strong></td><td><strong>Type</strong></td><td><strong>Required</strong></td><td><strong>Pattern/</strong><br><strong>Format</strong></td><td><strong>Description</strong></td><td><strong>Example</strong></td></tr><tr><td>tokenizationId</td><td>string</td><td>false</td><td>uuid</td><td>Unique ID of the processed transaction in m10. Null if order was canceled or not processed yet</td><td>3fa85f64-5717-4562-b3fc-2c963f66afa6</td></tr></tbody></table>

**Request example:**

{% code fullWidth="true" %}
```url
curl -X 'GET' \
  'https://...acquiring/api/v1/user-tokenization/LFMxfXQModI1uu5isDypCgmZGIhPjEYMqts1Q1Rr0Whw3zC
  MbVsj_H' \
  -H 'accept: */*' \
  -H 'Authorization: Bearer <AccessToken>'
```
{% endcode %}

**Response example**

_**Code:**_ 200 OK

{% code fullWidth="true" %}
```json
{
  "id": "fi1Z2FhZajd7ROD4rtovEpNaLUAC6xR-t_cmjWVxPjF6NDODafaXvBtzQK_-4o",
  "userToken": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
{% endcode %}

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

{% code fullWidth="true" %}
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
{% endcode %}

**Important**:

* The callback should be configured using the **POST method**.
* **X-Nonce** is a random string, it is necessary to check that it is not repeated, i.e. each message contains a unique value in this field. It’s generated by m10 for each request.
* **X-HMAC** is a hash of the response body in **hex** format.\
  The key will be provided by m10.\
  The algorithm is HmacSHA256.

<pre data-full-width="true"><code>{% hint style="warning" %}
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

{% code fullWidth="true" %}
```json
{
  "id": "CrAYldtUbo86oJDObxnmX17Y3lnhZBOkA9qDdYsRC3GlBMjUsAanavNbIeeTZFYg",
  "userToken": "3fa85f64-5717-4562-b3fc-2c963f66afa6"
}
```
{% endcode %}

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
