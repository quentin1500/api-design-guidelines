# General guidelines
#### 🔴 MUST follow API REST Design Principles
<<🔃 Client>> produces REST APIs.

For 95% of the API designed, we apply the RESTful web service principles to all kind of application service components.
They may be marginal API (the other 5%) that are designed another way to cover specific use-cases (GraphQL, gRPC, SOAP, ...)

The following guideline describes in details the different REST specific rules.

#### 🔴 MUST follow KISS principles
Embrace the KISS principle: Keep It Simple, Stupid. Our API design philosophy prioritizes simplicity and clarity. By choosing straightforward solutions over complexity, we aim to enhance usability and minimize errors.

Follow this principle to create effective and maintainable APIs.

#### 🔴 MUST follow API First principles
Design the API before coding it, so it is compliant with the rules and it deliver the proper business goal.

#### 🔴 MUST design using the OpenAPI Specification format
While designing, we must use the OpenAPI Specification (OAS) format, as described below in the document.

#### 🔴 MUST be compliant with enterprise data model
While designing an API, the business entity and attributes used must be compliant with the enterprise (or program) data catalog. This means that the data must be identified, documented, and exposed.

#### 🔴 MUST use English as description language
All aspects of the API, including documentation, comments, endpoint names, and error messages, should be written exclusively in English. 

This standard ensures consistency and accessibility for developers across different regions and languages. Adhering to this guideline streamlines collaboration and enhances the clarity of communication.

#### 🔴 MUST provide API specification using OpenAPI
We use the OpenAPI specification (OAS) as standard to define API specification files.

We encourage to use OpenAPI 3.0 version. Official documentation available (here)[https://swagger.io/specification/v3/].

The OpenAPI specifications are then automatically processed through the CI/CD pipeline.

# API Basics
## API Basics - Meta information
#### 🔴 MUST contain API meta information in OAS
OpenAPI specifications must contain the following OpenAPI meta information to allow for API management:

- `#/info/title` as (unique) identifying, functional descriptive name of the API
    - The name of the micro-service is the current best practice
- `#/info/contact/{name,url,email}` containing the responsible team
- `#/servers` as access adresses of the API
    - including: <<client environments: DEV, UAT, STG, PRD>>
- `#/info/version` as described in the next section

#### 🔴 MUST use semantic version
OAS allows to specify the API specification version in `#/info/version`. 

It enables to clearly identify the complete version of an API, along its life cycle. It helps to iterate faster and prevents invalid requests from hitting updated endpoints. It also helps smooth over any major API version transitions as it's allowing to continue to offer old API versions for a period of time.

We must use the Semantic Versionning rules.

The format of a version number is: <major>.<minor>.<patch>
Where:
- Major: version when you make incompatible API changes. Two major versions can't be used interchangeably. Changes on resource path or required parameters are considered as breaking changes.
- Minor: version when you add functionality in a backwards compatible manner. It means that a client integrating the version 1.1 will be able to interact with the version 1.2. Addition of new resources or optionnal parameters are considered as backwards compatible.
- Patch: version when you make backwards compatible bug fixes. The change of examples, description and minor information in the contrat are considered as backward compatibles.

#### 🔴 MUST include unique operationId for each endpoint
Each operation must include an unique ID to be identified: **operationId**. An operation is a combination of an endpoint and an http method.

The operationId is located here in the OAS:
- `#/paths/{path}/{method}/operationId`

Where:
- `{path}` as the actual path of the operation
- `{method}` as the http method of the operation

The *operationId* must:
- Be written using PascalCasing
- Follow this template:

`<operationPrefix><resourceName>`
```
<operationPrefix><resourceName>
- operationPrefix: Get|Create|Do|Update|UpdatePartially|Delete (* according to the following mapping)
- resourceName: Plural if Collection, Singular if Item
ex:
  PUT /onboardings/{onboarding-id}/company-information
->UpdateOnboardingCompanyInformation
 ```


| Http Method | Operation Prefix |
|-----------|-----------------------------------------------------------------------------|
| GET | Get |
| POST /{resources}/* | Create |
| POST /{resources}/*/{actions} (in case of verb) ⚠️ | Do ⚠️|
| PUT | Update |
| PATCH | UpdatePartially |
| DELETE | Delete |
| HEAD | GetHead |

Additional Examples:
```
GET /identities/{identity_ref}
GetIdentity

GET /identities
GetIdentities

GET /configurations/legals/optins
GetConfigurationsLegalsOptins

POST /identities
CreateIdentities

PATCH /user-profiles/{user_profile_ref}
UpdatePartiallyUserProfile

POST /users/actions/request-password-reset
DoUsersRequestPasswordReset

POST /users/{username}/actions/request-password-reset
DoUserRequestPasswordReset

POST /acquirer-authorisation-initiations
CreateAcquirerAuthorisationInitiations

PUT /user-profiles/{user_profile_ref}/personal-info/addresses
UpdateUserProfilePersonalInfoAddresses

HEAD /users/{user_ref}/notifications
GetHeadUserNotifications
```

## API Basics - Security
#### 🔴 MUST secure endpoints with Access Tokens
API must secure their endpoints by validating the Access Token provided in the request.

This Access Token is:
- Issued by <🔃 to be completed in each client context>
- Provided by the client consumer in the `Authorization` request header

The Access Token may be: <🔃>
- in [**JWT** (JSON Web Token) format](https://jwt.io/). Major advantage is the token is self-carrying, meaning it doesn't need the authorisation serveur for its introspection (validity check)
- in **opaque** format. Major advantage is the opacity, meaning the identifications and authorizations remain secrets.
- both

JWT Payload example:
```
{
  "sub": "1234567890",
  "name": "John Doe",
  "clientId": "{clientId}",
  "roles": [
    {
      "key": "{roleType}-{roleKey}",
      "realmId": "{clientId-realm}"
      "permissions:":[
        "order/read",
        "billing/read",
        "transaction/read",
      ]
    },
    {
      "key": "{roleType}-{roleKey}"
    }
  ]
}
```

Opaque token example:
```
4e2ad0f9-007b-4aa6-94a0-25657259db74
```

#### 🔴 MUST validate access tokens
Each request to a secured endpoint must include validation of the provided Access Token:
1. Token Integrity:
    - Ensure the Token is authentic and unaltered
1. Token Expiration:
    - Confirm that the token has not expired
1. Roles and Permissions Validation:
    - Ensure that the roles and permissions embedded in the Access Token authorize the requested operation or access to the resource

<*🔀 A choice must be done, it may includes both strategies*>
- In JWT:
    1. Token integrity is validated with the Signature validation: This is the primary aspect of verification where the signature part of the JWT is checked against the header and payload. This is done using the algorithm specified in the header (like HMAC, RSA, or ECDSA) with a secret key or public key.
    1. Token expiration is validated by a standard payload field, `exp`.
    1. Roles and permissions are embeded in the JWT token, enabling the API to grant the access or not.
- In opaque token:
    1. A introspection with the authorization server is needed. The authorization server provides the result of the integrity, expiration verification, and eventually details about the roles and permissions.

If a required permission is absent from the Access Token, request is blocked. This fetches the necessary permissions associated with the role ID or roleId-realmId.

#### 🔴 MUST define and assign permissions (scopes)
Each endpoint must define clearly the permissions required for access. These endpoint permissions should be compared against the user's permissions listed in the Access Token.:
- Access Granted: If a match occurs between endpoint permissions and user permissions.
- Access Denied: If no match is found or if user permissions are insufficient.

<*🔀 eventually describe the standard and expected format of the permissions, depending on the company*>

## API Basics - Data formats
#### 🔴 MUST use standard data formats
OAS allows to define a parameter type and format. You must use these formats, whenever applicable:

| OpenAPI type | OpenAPI format | Specification  | Example   |
|--------------|----------------|----------------|-----------|
| `integer`    | `int32`        | 4 byte signed integer between -2³¹ and 2³¹-1  | `7721071004`                             |
| `integer`    | `int64`        | 8 byte signed integer between -2⁶³ and 2⁶³-1  | `772107100456824`                        |
| `integer`    | `bigint`       | arbitrarily large signed integer number       | `77210710045682438959`                   |
| `number`     | `float`        | `binary32` single precision decimal number — see [IEEE 754-2008/ ISO 60559:2011](https://en.wikipedia.org/wiki/IEEE_754) | `3.1415927`      |
| `number`     | `double`       | `binary64` double precision decimal number — see [IEEE 754-2008 / ISO 60559:2011](https://en.wikipedia.org/wiki/IEEE_754) | `3.141592653589793` |
| `number`     | `decimal`      | arbitrarily precise signed decimal number     | `3.141592653589793238462643383279`       |
| `string`     | `byte`         | `base64url` encoded byte ([RFC 7493 Section 4.4](https://tools.ietf.org/html/rfc7493#section-4.4)) | `"VA=="`          |
| `string`     | `binary`       | `base64url` encoded byte sequence ([RFC 7493 Section 4.4](https://tools.ietf.org/html/rfc7493#section-4.4)) | `"VGVzdA=="`    |
| `string`     | `date`         | [RFC 3339](https://tools.ietf.org/html/rfc3339) internet profile — subset of [ISO 8601](https://tools.ietf.org/html/rfc3339#ref-ISO8601) | `"2019-07-30"`       |
| `string`     | `date-time`    | [RFC 3339](https://tools.ietf.org/html/rfc3339) internet profile — subset of [ISO 8601](https://tools.ietf.org/html/rfc3339#ref-ISO8601) | `"2019-07-30T06:43:40.252Z"` |
| `string`     | `time`         | [RFC 3339](https://tools.ietf.org/html/rfc3339) internet profile — subset of [ISO 8601](https://tools.ietf.org/html/rfc3339#ref-ISO8601) | `"06:43:40.252Z"`   |
| `string`     | `duration`     | [RFC 3339](https://tools.ietf.org/html/rfc3339) internet profile — subset of [ISO 8601](https://tools.ietf.org/html/rfc3339#ref-ISO8601) | `"P1DT30H4S"`       |
| `string`     | `period`       | [RFC 3339](https://tools.ietf.org/html/rfc3339) internet profile — subset of [ISO 8601](https://tools.ietf.org/html/rfc3339#ref-ISO8601) | `"2019-07-30T06:43:40.252Z/PT3H"` |
| `string`     | `password`     | NA | `"secret"`   |

#### 🔴 MUST use standard formats for country, language and currency properties

| Country       | Two letter country codes ([ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2))  |
|---------------|----|
| `France`      | FR |
| `Bulgaria`    | BG |
| `Poland`      | PL |
| `Mexico`      | MX |

| Language     | Two letter language codes ([ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639-1_codes))  |
|--------------|----|
| `English`    | en |
| `French`     | fr |
| `Bulgarian`  | bg |
| `Polish`     | pl |
| `Spanish`    | es |



| Currency     | Three letter currency codes ([ISO 4217](https://en.wikipedia.org/wiki/ISO_4217)) |
|--------------|-------|
| `Euro, €`    | EUR   |

# REST Basics
## REST Basics - URLs
#### 🔴 MUST not use /api as base path
The usage of the micro-service and API as Product approach makes obvious the fact that it is an API.

#### 🔴 MUST follow the path semantic
This entails constructing paths that reflect the RESTful semantic structure, employing resources and their identifiers across distinct levels of the resource hierarchy.

Template: `/<resources>/<resource-identifier>/<sub-resources>/<sub-resource-identifier>`

We advocate for a maximum of three levels of resources to maintain clarity and simplicity in the API design

#### 🟡 SHOULD pluralize resource names
Resource names within our API should be pluralized, reflecting collections of resources rather than individual instances. This standardization enhances consistency and intuitiveness for developers interacting with the API.

Exceptions:
- Actions (materialized by a verb instead of a noun)
- Search (the Search template)
- Singleton (Unique object in the system)
  - ex.: profile (an user only have one and only *profile*)

While pluralization is the norm, these exceptions warrant singular naming to maintain clarity and precision in representing their unique functionalities.

#### 🔴 MUST use <🔀 chosen case> for path segments
<*🔀 A choice must be done, it is recommended to have only one strategy for the company for consistency reasons*>

🔀 #1 - spinal-case

Path segments are restricted to ASCII spinal-case strings. Matching regex `\b[a-z]+(-[a-z]+)*\b`.

Ex.: 
```
/billing-contact
/phone-number
/legal-name
```

🔀 #2 - snake_case

Path segments are restricted to ASCII spinal-case strings. Matching regex `\b[a-z]+(_[a-z]+)*\b`.

Ex.: 
```
/billing_contact
/phone_number
/legal_name
```

🔀 #3 - lowerCamelCase 

Path segments are restricted to ASCII lowerCamelCase strings. Matching regex `\b[a-z]+([A-Z][a-z0-9]*)*\b`.

Ex.: 
```
/billingContact
/phoneNumber
/legalName
```
#### 🔴 MUST use <🔀 chosen case> for query parameters
<*🔀 A choice must be done, it is recommended to have only one strategy for the company for consistency reasons*>

🔀 #1 - spinal-case

Query parameters are restricted to ASCII spinal-case strings. Adantage here is to match the path segment case. Matching regex `\b[a-z]+(-[a-z]+)*\b`.

Ex.: 
```
?billing-contact=""
?phone-number=33
?legal-name=""
```

🔀 #2 - snake_case

Query parameters are restricted to ASCII spinal-case strings. Adantage here is to match the path segment case. Matching regex `\b[a-z]+(-[a-z]+)*\b`.

Ex.: 
```
?billing-contact=""
?phone-number=33
?legal-name=""
```

🔀 #3 - lowerCamelCase

Query parameters are restricted to ASCII lowerCamelCase strings. Advantage here is to match the payload parameters. Matching regex `\b[a-z]+([A-Z][a-z0-9]*)*\b`.

Ex.: 
```
?billingContact=""
?phoneNumber=33
?legalName=""
```

#### 🟡 SHOULD keep URLs verb-free
Prioritizing resources (nouns) over actions fosters a more intuitive and RESTful architecture. Rather than focusing on actions, emphasize the manipulation of resources through standard HTTP methods like GET, POST, PUT, and DELETE. This approach promotes a clearer and more scalable API structure.

By conceptualizing operations as resource manipulations, developers can create a more coherent and predictable API experience for users.

Exceptions:
- Actions (materialized by a verb instead of a noun)

If the HTTP verbs absolutely doesn't fit the action performed by the endpoint, a infinitive verb may be used instead of a noun.

Ex.:
- /passwords/reset
- /cards/{card-id}/check

#### 🟡 SHOULD use "/me" endpoints to represent the current authenticated user
"/me" endpoints provide a shortcut to access or manipulate the resource representing the currently authenticated user without requiring the client to specify a user identifier explicitly. This enhances security and simplifies API consumption.

"/me" endpoints are strictly scoped to the authenticated user's context (based on the provided Token) and must not allow accessing or modifying resources belonging to other users, even indirectly.

Place "/me" as a sub-resource under the user-related root path to emphasize its semantic meaning.

Ex.: `/users/me`

## REST Basics - JSON payload
#### 🔴 MUST use JSON as payload data interchange format
Use JSON [RFC 7159](https://tools.ietf.org/html/rfc7159) to represent structured (resource) data passed with HTTP requests and responses as body payload.

#### 🟡 SHOULD pluralize array names
Array names should be in plural form to signify they hold multiple values, while object names should be singular to maintain consistency.
Ex.: 
```
{
  "name": "Doe",
  "adresses":[
    { "street": "123 Maple Street" },
    { "street": "456 Oak Avenue, Apt 7B" }
  ]
}
```

#### 🔴 MUST use lowerCamelCase for property names
Property names are restricted to ASCII lowerCamelCase strings. Matching regex \b[a-z]+[A-Z][a-zA-Z]*\b.

Ex.: 
```
billingContact
phoneNumber
legalName
```

#### 🔴 MUST declare enum values using UPPERCASE string
ENUM values should be ASCII UPPERCASE strings. Matching regex \b[A-Z]*\b.

Ex.:
```
INITIATED
INPROGRESS
TOBEVALIDATED
```

#### 🟡 SHOULD name date/time properties with proper suffix
Dates and date-time properties should end with the following suffix to distinguish them from boolean properties which otherwise would have very similar or even identical names:

| Type      | Suffix    | Example   | rather than  |
|-----------|-----------|-----------|--------------|
| Date-time | `At`      | `"createdAt: "2019-07-30T06:43:40.252Z"` | `"created": true` |
| Date      | `Date`    | `"modifiedDate: "2019-07-30"` | `"modified": true` |
| Time      | `Time`    | `"occuredTime: "06:43:40.252Z"` | `"occurred": false` |

Note that it is not recommanded to have a *time* and a *date*, for the same event, instead you should prefer the usage of an unique *date-time*.

Example:
- Prefer:
``` JSON
{
  ...
  "createdAt" : "2019-07-30T06:43:40.252Z"
}
```
- Instead of:
``` JSON
{
  ...
  "createdDate" : "2019-07-30",
  "createdTime" : "06:43:40.252Z"
}
```

#### 🔴 MUST avoid the usage of a root field
A generic root field must not be used at the root level of the response. Typical generic field names to avoid:
- data
- {resource}

The root level must be reserved for the root attributes of the requested ressource.

Wrong example:
``` JSON
{
  "data":{
    "name": "Doe",
    "firstName": "John"
  }
}
```

Right example:
``` JSON
{
  "name": "Doe",
  "firstName": "John"
}
```

#### 🔴 MUST use the <🔀 choice made> to name the entitiy identifiers
<*🔀 A choice must be done, it is recommended to have only one strategy for the company for consistency reasons*>

🔀 #1 - explicit ID

Using an explicit ID is recommanded to avoid mistake, especially in an environment where data maturity is low. The explicit ID is composed by the name of the entity followed by the suffix "Id".

Example: `userId`

🔀 #2 - generic ID

Using an generic ID simplifies a lot the description of a ressource, but may leads to risks of misalignment. It is recommended in environments with high data maturity. The generic ID is simply : `id`

=== end of propositions ===

It is mandatory to follow the naming convention that is used for property names: [#### 🔴 MUST use lowerCamelCase for property names](#🔴-MUST-use-lowerCamelCase-for-property-names)

## REST Basics - HTTP requests
#### 🔴 MUST use HTTP methods correctly
Be compliant with the standardized HTTP semantics (see [RFC 7231: Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://datatracker.ietf.org/doc/html/rfc7231#section-4.3) ) as summarized here.

- GET: Retrieve a representation of the resource. This method is idempotent and safe, meaning it should not have side effects on the server. Note: you may consider the retreiving with a payload section in case the amount of payload is too large.
- POST: Submit data to be processed to a specified resource, often causing a change in state or side effects on the server.
- PUT: Replace the target resource with the request payload. This method is idempotent.
- DELETE: Remove the target resource. This method is idempotent.
- PATCH: Apply partial modifications to a resource. This method is not necessarily idempotent.*

#### 🟡 SHOULD locate the resource identifier to create (if in input) in the body
When creating a resource providing its identifier (id) it is advisable to locate the resource identifier in the body (not in the path)

**Right example:**
```
POST /resources
{
  "resourceId":d2c08fc2-b585-4b02-8c21-1920fdb76df5
}
```

**Wrong example:**
```
POST /resources/d2c08fc2-b585-4b02-8c21-1920fdb76df5
```

#### 🟡 SHOULD consider the use-case of retrieving with a payload (Search)
When an endpoint is used to retrieve data (typically utilizing a GET method) and the amount of data to be passed as an input is anticipated to be too large for the client (browser, compiler, ...), it is advisable to consider creating a "Search endpoint".

##### Use-case details

- For many clients, particularly web browsers, the URL size is limited.
  - Internet Explorer, for instance, has a maximum URL length of 2,083 characters
- It is possible that a client may request data to retrieve through a large set of filters, such as (really long) lists of identifiers.

```
GET /company/{company-id}/employees?employee-ids=00001,00002,00003,00004,00005,00006,00007,00008,00009,00010,00011,...
```

These two factors combined necessitate the proposal of a workaround.

##### Search endpoint

The workaround involves implementing a "Search endpoint" using a POST method and the "search" verb instead of a traditional GET endpoint.

```
POST /company/{company-id}/employees/search
{
  "employeeIds":[
    00001,
    00002,
    00003,
    00004,
    00005,
    00006,
    00007,
    00008,
    00009,
    00010,
    00011,
    ...
  ]
}
```

The payload is placed into the body of the request, following the normal body convention: JSON, camelCase, and other relevant points.

**Note**: Some clients (servers or proxies) are not compatible with the usage of a body within a `GET` method. That is why the usage of the `POST` method is recommended, even if it may be not compatible with a RestFul approach.  

## REST Basics - HTTP status codes
#### 🔴 MUST use official HTTP status codes
You should only utilize official HTTP status codes consistently according to their intended semantics.

An overview on the official error codes provides by [Wikipedia](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes).

#### 🔴 MUST specify success and error responses
You must define all success and service specific error responses in your API specification.

Error code response descriptions should provide information about the specific conditions that lead to the error, especially if these conditions can be changed by how the endpoint is used by the clients.

Standard client and server errors, e.g. `401` (unauthenticated), `403` (unauthorized), `404` (not found), `500` (internal server error), or `503` (service unavailable), where the semantic can be easily derived from the endpoint specification may not be explicitely documented, as generic and standard error codes.

## REST Basics - HTTP headers
#### 🟢 MAY use standard headers
APIs may make use of HTTP headers defined by non-obsolete RFCs. The supported headers must be explicitly mentioned in the API specification.

#### 🔴 MUST use Upper-kebab-case for property names
Headers are restricted to ASCII kebab-case strings (with uppercase separate). Matching regex `\b[A-Z][a-zA-Z]*(?:-[A-Z][a-zA-Z]*)*\b`.

Ex.: 
```
Accept
Accept-Language
Custom-Header
```

#### 🔴 MUST not use the "X" prefix for custom headers
The use of the "X-" prefix for custom HTTP headers, although historically common, is explicitly discouraged by [RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648). This specification recommends avoiding such prefixes in favor of clearly named and well-documented headers.

In line with the KISS (Keep It Simple, Stupid) principle and modern best practices, custom headers should be defined without the "X-" prefix.

# REST Design
## REST Design - Performance
#### 🟢 MAY use gzip compression
Servers and clients are encouraged to support `gzip` content encoding for faster response times by reducing data transmission. Exceptions to compression include already compressed content or server resource limitations. 

While `gzip` should be the default, servers must also offer unencoded content for testing through content negotiation using the `Accept-Encoding` header. Successful compression is indicated by the server by the `Content-Encoding` response header. 

Those headers can take multiple values, depending on the supported algorithms. Ex.: `compress`, `deflate`, or `br` in addition to the default `gzip`.

To signal server support for compression the API specification should define both, the `Accept-Encoding` request header and the `Content-Encoding` response header.

1. **RFC 7231 section 3.1.2: Encoding for Compression or Integrity**:
    - [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231#section-3.1.2)
1. RFC 7694: 
    - [RFC 7694](https://datatracker.ietf.org/doc/html/rfc7694)


#### 🟢 MAY support partial responses via filtering
You can reduce network bandwidth need by supporting filtering of returned entity fields. This feature allows the client to precisely specify the subset of fields they wish to obtain by utilizing the `fields` query parameter.

**Unfiltered**
```
GET http://api.example.org/users/123 HTTP/1.1
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": "cddd5e44-dae0-11e5-8c01-63ed66ab2da5",
  "name": "John Doe",
  "address": "1600 Pennsylvania Avenue Northwest, Washington, DC, United States",
  "birthday": "1984-09-13",
  "friends": [ {
    "id": "1fb43648-dae1-11e5-aa01-1fbc3abb1cd0",
    "name": "Jane Doe",
    "address": "1600 Pennsylvania Avenue Northwest, Washington, DC, United States",
    "birthday": "1988-04-07"
  } ]
}
```

**Filtered**
```
GET http://api.example.org/users/123?fields=<🔀 format to be chosen>
HTTP/1.1 200 OK
Content-Type: application/json
{
  "name": "John Doe",
  "friends": [ {
    "name": "Jane Doe"
  } ]
}
```
<*🔀 A choice must be done, it is recommended to have only one strategy for the company for consistency reasons*>

🔀 #1 - **Proposition#1: Custom with Parenthesis**

==> `(name,friends(name))`

Comma-separated list of fields to include or exclude in the response. 

Syntax:
- Use parentheses to group fields.
- Prefix a group with "!" to exclude it.
- Multiple fields should be separated by commas.
- Example format: `(field1,field2(nestedField1,nestedField2),field3!(nestedField4))`

Example:
- Include specific fields:
`  ?fields=(field1,field2) `
- Include nested fields:
`  ?fields=(field1,field2(nestedField1,nestedField2)) `
- Exclude specific nested fields:
`  ?fields=(field1,field2!(nestedField3)) `
- Complex selection and exclusion:
` ?fields=(field1,field2(nestedField1,subField1(subSubField1,subSubField2)),field3!(nestedField4,nestedField5)) `


🔀 #2 - **Proposition#2: JSON Path based**

==> `$.name,$.friends.name`

Comma-separated list of JSONPath expressions specifying fields to include or exclude in the response.

Syntax:
- Use JSONPath syntax to specify fields. 
- Prefix a JSONPath expression with "!" to exclude that field.
- Multiple fields should be separated by commas.

Example:
- Include specific fields: 
` ?fields=$.field1,$.field2 `
- Include nested fields: 
` ?fields=$.field1,$.field2.nestedField1,$.field2.nestedField2 `
- Exclude specific nested fields: 
` ?fields=$.field1,!.field2.nestedField3 `
- Complex selection and exclusion: 
` ?fields=$.field1,$.field2.nestedField1,!.field2.nestedField3,$.field3 `

=== end of propositions ===

This may be complex to implement. It must be set with parsimony on specific use-cases only. (ex.: mobile consumption)

It can be an alternative, easier and better integrated way than actual GraphQL implementation.

An easier way to implement this kind of mechanism may be the predifined fields, described in the next section.

#### 🟢 MAY support partial responses via predefined fields

In the same way, you can reduce network bandwidth need by supporting usage of predifined fields. This feature allows the client to specify the presence or not of a subset of fieldsby utilizing the `include<Fields>` boolean query parameter.

**Unfiltered**
```
GET http://api.example.org/users/123 HTTP/1.1
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": "cddd5e44-dae0-11e5-8c01-63ed66ab2da5",
  "name": "John Doe",
  "address": "1600 Pennsylvania Avenue Northwest, Washington, DC, United States",
  "birthday": "1984-09-13",
  "friends": [ {
    "id": "1fb43648-dae1-11e5-aa01-1fbc3abb1cd0",
    "name": "Jane Doe",
    "address": "1600 Pennsylvania Avenue Northwest, Washington, DC, United States",
    "birthday": "1988-04-07"
  } ]
}
```

**Filtered**
```
GET http://api.example.org/users/123?includeFriends=false
HTTP/1.1 200 OK
Content-Type: application/json
{
  "id": "cddd5e44-dae0-11e5-8c01-63ed66ab2da5",
  "name": "John Doe",
  "address": "1600 Pennsylvania Avenue Northwest, Washington, DC, United States",
  "birthday": "1984-09-13"
}
```

These fields are predetermined in the code and must be explicitely described in the OAS.

#### 🟢 MAY document cacheable `GET`, `HEAD`, and `POST` endpoints
To indicate that a REST API response for a specific endpoint is cacheable, you can use HTTP headers that provide caching directives. Here are the standard headers you might use:

1. **Cache-Control**: This header provides all the caching directives and is the most commonly used for indicating cache policies. Some common directives include:
    - max-age: Specifies the maximum amount of time a resource is considered fresh.
    - public: Indicates that the response may be cached by any cache.
    - private: Indicates that the response is intended for a single user and must not be stored by shared caches.
    - no-cache: Forces caches to submit the request to the origin server for validation before releasing a cached copy.
    - no-store: The cache should not store anything about the client request or server response.
1. **Expires**: This header provides a date/time after which the response is considered stale. It's used for HTTP/1.0 caches, but `Cache-Control` is preferred for modern applications.

ETag: This header provides a unique identifier for a specific version of a resource. If the resource changes, the ETag value should change. Clients can use the If-None-Match header in subsequent requests to check if the resource has changed.

Last-Modified: This header indicates the date and time the resource was last modified. Clients can use the If-Modified-Since header in subsequent requests to check if the resource has changed.

Here's an example of a REST API response with caching headers:
```
HTTP/1.1 200 OK
Cache-Control: public, max-age=3600
Expires: Wed, 05 Jun 2024 20:00:00 GMT
ETag: "686897696a7c876b7e"
Last-Modified: Wed, 05 Jun 2024 19:00:00 GMT
Content-Type: application/json
{
    "id": 123,
    "name": "Example Resource",
    "timestamp": "2024-06-05T19:00:00Z"
}
```
In this example:
- The `Cache-Control` header indicates that the response can be cached by any cache (`public`) and should be considered fresh for 3600 seconds (1 hour).
- The `Expires` header provides a specific date and time after which the response is considered stale.
- The `ETag` header provides a unique identifier for the version of the resource.
- The `Last-Modified` header provides the last modification date of the resource.

These headers together help clients and intermediaries understand how to cache the response properly.

1. **RFC 7234: Hypertext Transfer Protocol (HTTP/1.1): Caching**: This RFC defines HTTP caching and describes the Cache-Control header and other caching mechanisms.
    - [RFC 7234](https://datatracker.ietf.org/doc/html/rfc7234)
1. RFC 7231: Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content: This RFC provides additional context about HTTP headers, including the `Expires`, `ETag`, and `Last-Modified` headers.
    - [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231)

## REST Design - Pagination
#### 🟡 SHOULD support pagination (when necessary)
To ensure optimal performance and prevent service overload, it is essential to implement pagination for data item lists. Pagination not only safeguards the service but also enhances client-side iteration and batch processing. This practice is particularly crucial for lists containing a substantial number of entries, exceeding just a few hundred.

The default way to perform in <<🔃 Client>> is:
- **Page-base pagination**, described in the next section

Less recommended methods includes:
- **Offset-based pagination**
- **Cursor-based pagination**

#### 🟡 SHOULD prefer page-based pagination
Page-based navigation divides data into uniformly sized pages, simplifying data handling and enhancing the developer experience. More specifically, it allows you to specify the page number and the page size (limit). From there, the server can calculate the offset and return a page with items that match the page size.

Advantages:
- Predictability. It effectively eliminates the chances of navigating to nonexistent pages—a common pitfall in offset pagination
- Splitting data into equal-sized pages is that the API can also provide metadata on how many pages there are, as well as links to previous and next pages, boosting the API consumer's user experience.

Here's a simple example of a page-based pagination request:
```
GET /articles?page=3&pageSize=10
```
In this request:
- The endpoint is /api/articles.
- The parameter page=3 specifies the client's request for the third page of results.
- The parameter pageSize=10 defines the number of articles displayed per page as 10.

Disadvantages:
- In environments with large data sets, since it relies on offset and limit calculations, performance issues can arise when calculating offset for higher page numbers. 
- In environments with fast-changing data, such as social feeds, there's a risk of encountering data repetition or skipped data. That's because new data can be added or existing data can be altered or deleted during consecutive page requests.

API endpoints requests and response schemas are described in the dedicated sections.

#### 🟡 SHOULD use page-based pagination request method
Implementing page-based pagination, the requests will indicate the starting point for data retrieval using a "page number" and the quantity of items to be displayed per page, specified as follows in the query:
- `page-index`: indicates the page number
- `page-size`: indicates the number of items per page

In OAS, the following references can be included:
```
    - $ref: ../../../modules/smarter/parameters/query/pageIndex.yaml
    - $ref: ../../../modules/smarter/parameters/query/pageSize.yaml 
```
Example:
```
GET /articles?page-index=3&page-size=10
```
In this request:
- The endpoint is `/articles`.
- The parameter `page=3` specifies the client's request for the third page of results.
- The parameter `pageSize=10` defines the number of articles displayed per page as 10.

#### 🟡 SHOULD use page-based pagination response object
The paging details are provided under the "meta" section in the response body, specifically within the `paging` field. For a detailed structure, please refer to the JSON schema response template.

The paging information includes the following parameters:
- `pageIndex`: indicates the current page being returned
- `pageSize`: indicates the number of items on each page
- `hasPreviousPage`: indicates whether there is a previous page
- `hasNextPage`: indicates if there is a next page
- `totalCount`: indicates the total count of pages available

If a request references a resource outside the valid range, the response may utilize the 206 Partial Content HTTP status code.

A sorting must be applied by default on the response, to keep the default order of the ressources in the response object after iterating on the pages.

Example:
```
HTTP/1.1 200 OK
PageIndex: 3
PageSize: 10
HasPreviousPage: false
HasNextPage: true
TotalCount: 100
```
In this response:
- The header `PageIndex: 3` specifies the response returns the third page of results.
- The header `PageSize: 10` specifies the number of items displayed per page as 10.
- The header `HasPreviousPage: false` specifies that a previous page does exist.
- The header `HasNextPage: false` specifies that a next page does exist.
- The header `TotalCount: 100` specifies that a total of 100 items exists.

#### 🟡 SHOULD avoid a total result count
In pagination responses you should avoid providing a total result count, since processing it is a costly operation that is usually not required by clients.

<*🔀 A choice must be done, it is recommended to have only one strategy for the company for consistency reasons*>

🔀 #1 - **Paginated by Default**

The API always retreives a paginated response, even if it is not explicitely requested by the client. The response object contains the paginated details.

Advantages / disadvantages: Always retreives an object but maybe tricky for the client to detect the pagination.

🔀 #2 - **Mandatory paging parameters**

The API always requires a paginated requestfrom the client. If not the case, a `400` is returned, and no response.

Advantages / disadvantages: Always detectes by the client that is asked to provide a paginated request, but may be more complex to implement.

=== end of propositions ===

This strategy must be explicitely described in the OAS.

#### 🟢 MAY support custom sorting
A generic parameter `sort` may be used to describe sorting rules. Accommodate complex sorting requirements by letting the `sort` parameter token in list of comma separated fields.

- **sort**: Contains the names of the attributes on which the sorting is performed, separated by a comma.
- **desc**: By default the sorting is done in ascending order. If one wishes to sort in descending order, they need to add this parameter (without any value). In some specific cases, one may want to specify which attributes should be used as ascending sort keys and which as descending sort keys. Then, the desc parameter should contain the attributes that will be descending sort keys, the others will be ascending sort keys.

| Request | Description |
|---|---|
| `/restaurants?sort=name` | retrieving the list of restaurants sorted by name. |
| `/restaurants?sort=rating,reviews,name&desc=rating` | retrieving the list of restaurants, sorted by descending rating, then by ascending review count, and finally by ascending name. |

## REST Design - Compatibility
#### 🔴 MUST not break backward compatibility
Change APIs, but keep all consumers running. Consumers usually have different release lifecycles, so, you must focus on stability, and avoid changes that do not provide additional value. APIs are contracts between service providers and service consumers that cannot be broken via unilateral decisions.

Here are the most common cases of breaking changes to avoid (not exhaustive):
1. Change in the paths/endpoints
    - Delete a endpoint
    - Change the  HTTP verb (`GET` becomes `POST`)
    - Change the endpoint
        - Rename a endpoint (`/v1/users` becomes `/v1/members`)
        - Change in the order or structure of the URL (`/user/{id}/posts` becomes `/posts/user/{id}`)
1. Change in the request
    - Delete a parameter
    - Add a mandatory parameter without a default value
    - Change parameter format:
        - Rename or modify a parameter (`firstName` becomes `first_name`)
        - Change the type of a parameter (`int` becomes `string`)
        - Change in the data format (`date ISO 8601` → `timestamp UNIX`)
1. Change in the reponse
    - Delete a field in the response
    - Rename a field in the response
    - Change the data type of a field (`boolean` becomes `string`)
    - Change in response structure (the response is not an object anymore but an array)
    - Add a rule of stict validation (a field is not nullable)
1. Change the authentication/authorization
    - From an authentication way to another (`API key` → `OAuth2`).
    - Add restriction of roles and permission on endpoint that did not have
1. Change of business logic
    - A behaviour or a default value changes (a sorting per date becomes a `sorting per name`)
    - Modify the resultat of an operation (suppression logique au lieu de suppression physique).
1. Change the contract (OpenAPI/Swagger)
    - Any modification of the specification that invalidates internal API calls. (change of the model schema, delete a HTTP Code documented as a 200 or 201)

There are ways to change APIs without introducing breaking change:
- Follow the previous described guidelines
- SHOULD design API conservatively setion below
- MUST follow standard life cycle section below

#### 🟡 SHOULD design API conservatively
API designers should adhere to the following guideline to anticipate RESTful APIs ability to evolve in their life-cycle avoiding un-compatible extensions.

- Endpoint that retrieves resources data (`GET` or `POST /search`) on collections (without identifier) should always return an array object
  - To enable an actual collection of items to be retrieved
  - Ensures consistency in response format
- Add only optional, never mandatory parameters. (Request specific)
- Never change the semantic of fields (ex.: changing the semantic from customer-number to customer-id, as both are different unique customer keys).
- Maintain clear and comprehensive documentation especially but not only in the OAS
- Communicate changes clearly and effectively

#### 🔴 MUST follow standard life cycle
The API must follow the described standard lifecycle. The life cycle flow is the following:

The initial API status is "Live". When introduce a **breaking change** in the API, a new version comes to replace the previous version. Both version still live in parallel:
- the previous one as **Deprecated**
- the new one as **live**

Once the deprecated version is ready to be retire (once no longer consumed, ideally), the version is not available anymore and disappear.

Flow schema:
```
                          Breaking change       ┌▶ Retire V1
                              ▲                 │
                              ┆                 │
 API V1    o────▷[ Live API  ][ Deprecated API ]─────────────────▷
                              ┆
 API V2    o─────────────────▷[           Live API             ]─▷ 
                              ┆
```

The purpose is summarized and declined following those principles:
- Ease consumers consumption
    - Several versions live in parallel (only one is live; the other(s) deprecated)
    - A new version must support all services needed by a consumer (so the consumers don't have to consume different versions of the API)
- Limit the number of parallel versions, to facilite API server management
    - Retire prior versions (once no longer consumed, ideally, approximately 🔀6 months): that includes communicate with the consumers, refer to the section [🟡 SHOULD communicate deprecation to the consumers](#🟡-SHOULD-communicate-deprecation-to-the-consumers)
    - Include as much breaking change as possible in 1 new version (to avoid other multiple new breaking changes and versions)

You must follow the next section to locate the version number of the API.

#### 🔴 MUST include version number in the URL
Version number should be located in the URL at the end of the base path.

Following template `minus v + the version number`, according to the regexp format `v[0-9]*`.

Examples:
```
https://my-enterprise.com/user-referential/v1/users/123
https://api.visa.com/for-ex-rates/v2/foreign-exchange-rates
```

#### 🟡 SHOULD communicate deprecation to the consumers 
When an API is planned to be deprecated, communications should be done to inform the consumers. Two information should be given:
- Deprecation date: the date when the API will become deprecated
- Sunset date: the date when the API will become retired (unavailable)

The communications are done throught the following channel, detailed below:
- Free communication
- API Headers
- OAS

##### Free communication
Send a communication on a free human chanel: mail, newsletter, phone call, ...

##### API headers
Both those parameters should be also indicated in the API headers, inspired from [RFC 9745 - The Deprecation HTTP Response Header Field](https://www.rfc-editor.org/rfc/rfc9745.html) and [RFC 8594 - The Sunset HTTP Header Field](https://www.rfc-editor.org/rfc/rfc8594.html).

The `Deprecation` and `Sunset` headers should be set by the according **date-times**, using the format described in [🔴 MUST use standard data formats](#🔴-MUST-use-standard-data-formats).

Examples:
```
Deprecation: 2026-07-31T05:00:00Z
Sunset: 2026-12-31T23:59:59Z
```

When the deprecation date is reached, the `Deprecation` value is set at `true`.
```
Deprecation: true
Sunset: 2026-12-31T23:59:59Z
```

##### OAS
Those headers must also be described in the OAS with a clear description of the meaning.

The OAS itself should communicate the deprecated status of the API, by setting the endpoints as deprecated, using the field `deprecated: true`. The deprecation and sunset dates may also be communicated on the OAS, in the `description` section.
