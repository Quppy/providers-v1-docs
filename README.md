# Responses

All the API methods return the answer in JSON in the following format:

```
{
    // (string, optional) Error code.
    code,
    // (object, optional) Result data.
    data,
    // (string, optional) Error message.
    message
}

```

The success is determined by the response HTTP code.

# Authorization

Application authorization is performed using a pair of public and private keys. The public key (providerId) is openly sent along with requests in the `X-Provider-Id` HTTP header. The private key (providerSecret) is used to sign requests, it must be stored securely and remain inaccessible to third parties.

Authorization involves 3 HTTP headers:

| Header        | Description                     | Explanation                                                                                                                                       |
|---------------|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------|
| X-Date        | Request creation date and time. | The same format as the standard Date header is used in this case. [RFC 7231 - 7.1.1.2. Date](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) |
| X-Provider-Id | Provider ID.                    |                                                                                                                                                   |
| X-Signature   | Request signature.              |                                                                                                                                                   |

The signature is calculated using the following formula:  
`signature = SHA512(CONCAT(UPPER(providerId), date, UPPER(SHA512(providerSecret)), UPPER(requestBody)))`, where:

*   SHA512 - reception of the sha512 hash function;
*   CONCAT - string merge function;
*   UPPER - function for converting string values to uppercase;
*   providerId - provider ID (public key; value from the `X-Provider-Id` header);
*   date - request generation date and time (value from the `X-Date` header);
*   providerSecret - provider secret (private key);
*   requestBody - the request body or an empty string if absent.

Suppose we have the following parameters:

*   providerId: `example-b16913ea-8468-4d03-b974-c41f656aa247`
*   providerSecret: `example-a99ef1fb-c66f-414d-b712-294f9f9c2af9`
*   date: `Tue, 19 May 2020 08:49:17 GMT`
*   requestBody: `{ "key": "value" }`

Then:

*   UPPER(providerId): `EXAMPLE-B16913EA-8468-4D03-B974-C41F656AA247`
*   UPPER(SHA512(providerSecret)): `9618D83B39E1E9F4D2C177BB61B3593D5E5A53E3D8F278E49DC952BCAADC00B9385AC75BE04E2DC414FB0F803444FB0A2A40400BC42C972780ADBC9BD5CFA8EA`
*   UPPER(requestBody): `{ "KEY": "VALUE" }`
*   CONCAT(...): `EXAMPLE-B16913EA-8468-4D03-B974-C41F656AA247Tue, 19 May 2020 08:49:17 GMT9618D83B39E1E9F4D2C177BB61B3593D5E5A53E3D8F278E49DC952BCAADC00B9385AC75BE04E2DC414FB0F803444FB0A2A40400BC42C972780ADBC9BD5CFA8EA{ "KEY": "VALUE" }`
*   signature: `a7be22a54b3dd74f6f6d6384027f40eb9d5f88220f43a45fe8312947c55debb1dddf38ad78bd77a8145c747f9d1c6e43a34b7f8fb94d5aa08e9f76e9c8d36e1a`
    
The final values of the HTTP headers below are:

*   X-Date: `Tue, 19 May 2020 08:49:17 GMT`
*   X-Provider-Id: `example-b16913ea-8468-4d03-b974-c41f656aa247`
*   X-Signature: `a7be22a54b3dd74f6f6d6384027f40eb9d5f88220f43a45fe8312947c55debb1dddf38ad78bd77a8145c747f9d1c6e43a34b7f8fb94d5aa08e9f76e9c8d36e1a`

# Accounts

In Quppy terminology, an account is a user's container for wallets.

## Account (entity)

| Field              | Type                                            | Description                            |
|--------------------|-------------------------------------------------|----------------------------------------|
| id                 | string                                          | Account ID.                            |
| ref                | string                                          | External account ID.                   |
| address            | [Address](#address) (optional)                  | Address. Only for verified account.    |
| document           | [Document](#document) (optional)                | Document. Only for verified account.   |
| email              | string (optional)                               | Email. Only for verified account.      |
| firstName          | string (optional)                               | First name. Only for verified account. |
| lastName           | string (optional)                               | Last name. Only for verified account.  |
| verificationStatus | [VerificationStatus](#verification-status-enum) | Verification Status.                   |

## Get accounts (method)

Returns accounts.

GET /provider/v1/accounts?offset=0&take=100

Query params:

| Param  | Type                           | Description                                               |
|--------|--------------------------------|-----------------------------------------------------------|
| offset | number (optional, default 0)   | The number of entities to skip.                           |
| take   | number (optional, default 100) | The number of entities to take. The maximum value is 100. |

Result format:

| Param      | Type                         | Description                   |
|------------|------------------------------|-------------------------------|
| items      | [Account](#account-entity)[] | Entities.                     |
| hasMore    | bool                         | Are there more entities.      |
| totalCount | number                       | The total number of entities. |

## Get account (method)

Returns an account.

GET /provider/v1/accounts/{accountId}

Route params:

| Param     | Type   | Description |
|-----------|--------|-------------|
| accountId | string | Account ID. |

Returns [Account](#account-entity) entity.

## Create account (method)

Creates an account.

POST /accounts

Request format:

| Param | Type   | Description                                                                                                                                                             |
|-------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ref   | string | External ID. The maximum length is 100 characters. We recommend using a GUID. <br/> Must be unique. If an account with the same ID already exists, it will be returned. |
| email | string | Email.                                                                                                                                                                  |

Returns [Account](#account-entity) entity.