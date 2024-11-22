---
hidden: true
---

# m10 e-commerce API Description

## Get started

m10 provides an e-commerce API that enables merchants to create payments and refunds as well as to create tokenization on m10 for seamless payment processes. The tokenization process will enable users to make payments through m10 without being redirected to the application. The API is organized around REST. Our API has predictable resource-oriented URLs, accepts form-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes, authentication, and verbs.

The provided document and Swagger link offer a comprehensive overview of the API, detailing its various endpoints. The link to [Swagger](https://develop.m10payments.com/online-acquiring/swagger-ui/index.html)

{% include ".gitbook/includes/warning.md" %}

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

#### Error Codes

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

### **POST** `/api/v1/orders/actions/create-payment`

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

<table><thead><tr><th width="139">Name</th><th width="78">Type</th><th width="99">Required</th><th width="151">Pattern/Format</th><th width="181">Description</th><th>Example</th></tr></thead><tbody><tr><td>orderId</td><td>string</td><td>true</td><td>[a-zA-Z0-9_.-]{20,64}</td><td>Unique order ID for this merchant provided by the merchant.</td><td><code>upLZ6GwJhA3ddTtwoF89vBYLPfwNFPfI</code></td></tr><tr><td>currencyISO</td><td>string</td><td>true</td><td>enum: <code>AZN</code>, <code>USD</code></td><td>Currency ISO of the transaction.</td><td><code>AZN</code></td></tr></tbody></table>

