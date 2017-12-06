```
SIP: ?
Title: User Authentication & Authorization
Author: Braydon Fuller <braydon@storj.io>
        Dylan Lott <dylan@storj.io>
Status: Draft
Type: Standard
Created: 2017-12-05
```

Abstract
-------

Motivation
----------

- Shard recovery process will need to download files on behalf of another user to be able to recover the shards, this will be done without decrypting the file. It's not currently possible to authenticate as a recovery process on behalf of a user over the bridge API.
- Authentication over [Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication), relies on a shared secret and is open to various man-in-the-middle attacks. It also does not support decentralized authentication as well as public key cryptography.
- Administration tasks and support will require more levels of user authorization and delegation.
- Users currently do not have a way to give authorization to access meta data to download files to public users.
- Current API request signing does not include query arguments or the HTTP method (see [issue #2](https://en.wikipedia.org/wiki/SHA-2) for middleware).
- There is no method to specify or configure levels of authentication for specific user actions.
- Current user model does not consider case of groups and organizations.
- Using secp256k1 for user authentication to a bridge is not well supported across various environments including web browsers and some crypto libraries, more commonly available is secp256r1.
- There are issues with using a nonce in the signature for distributed applications that need to coordinate the state of the nonce, and can cause issues with requests being retried in a load balancer.

Specification
-------------

### Authentication with Digital Signatures

The user will send an HTTP request to a bridge to any user authenticated endpoint with the headers *(example)*:
```
"x-user-id: "user@domain.tld"
"x-user-timestamp": 1502150944050
"x-user-algorithm": "ecdsa-secp256k1-sha256"
"x-user-pubkey": "03919b139ed36a0ad2b709fc639d70c59f75445c0ac6b4adf7ad52b7321761d115"
"x-user-signature: "fd62894f0074c1c6ee56c7ec5e28a0d686e122f388610ddb8a7229e98d2052bc397333..."
"content-type": "application/json"
```

By being able to support multiple signing algorithms and curves, further support can be added without requiring additional cryptographic libraries, the default algorithm will be [ECDSA](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm) with [secp256k1](https://github.com/bitcoin-core/secp256k1) and a [sha256](https://en.wikipedia.org/wiki/SHA-2) hash.

The signature should be of a hash of the following concatenated data:
```
<http-method><bridge-url><timestamp><http-body>
```

- The bridge url must include the query arguments
- The pubkey is associated with the user
- The timestamp is within an error threshold
- The signature is valid

### Resource Authorization


Reference Implementation
------------------------


References
----------
- https://en.wikipedia.org/wiki/Basic_access_authentication
- https://github.com/Storj/service-middleware/issues/2
- https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm
- https://github.com/bitcoin-core/secp256k1
- https://en.wikipedia.org/wiki/SHA-2