# Rest Api with dotnet core

## add libraries
```
dotnet add package Microsoft.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.AspNetCore.Authentication.jwtBearer
```

## Rest consists of the following entities:
**Resource:** uniquely identifiable entities. Like data from a db, images etc...

**Endpoint:** ressource that can be accessed trough a url

**HTTP method:** HTTP-method is the type of request a client sends to a sever. (GET, PUT, POST, DELETE)

**HTTP header:** key-value pair used to share additional information between client and sever like:
- Type of data beeing sent. (JSON, XML)
- Type of encription supported.
- Authentication token.
- Additional Customer data based on application need. 

### JWT Token - JSON Web Token
Bearer token to secure the REST-Api. Open standard for transferring data securely between two entities (where auth is required)

A JWT is digitally signed using a secret by the token provider (api or auth-sever). 

JWT consists of the following parts:

**Header:** encoded data of token type and algorithm used to sign the data

**Payload:** encoded data of claims intended for share. (ex: Admin=true, claimedUsername=BobRoss, time_till_token_expires=31.1.2023/11:39...)

**Signature:**: created by signing using a secret key.

#### JWT - workflow
1. **Client requesting token:** Client sends a request to auth-server with necessary information to prove its identity. (ex. username and pw)
2. **Token creation:** auth-server receives the token request and verifies the identity. If valid a token will be created with the necessary claims encoded into it and sent back.
3. **Client sends token to resource server:** for each request to the resource-Api-server the client needs to include that token in the header.
4. **Resource server verifies the token:** (this can happen directly in the api-server or that server sends back a request to check the claims to the auth-server)
    - Read the token from the header.
    - Split the Header, Payload and Signature from the token.
    - Create signature of received header and payload using the same secret key used when creating the token.
    - check wether both the created signature and the signature received in the request are valid.
    - if signatures are the same, tokens are valid and provide access tot he requested resource. So an StatusOk Response with the payload can be sent back.
    - if signatures are different and unauthorized response will be sent back. 