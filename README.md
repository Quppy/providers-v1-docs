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

| Header | Description | Explanation |
| --- | --- | --- |
| X-Date | Request creation date and time. | The same format as the standard Date header is used in this case. [RFC 7231 - 7.1.1.2. Date](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) |
| X-Provider-Id | Provider ID. |  |
| X-Signature | Request signature. |  |

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
