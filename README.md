# GA4GH service-info specification [![](https://travis-ci.org/ga4gh-discovery/ga4gh-service-info.svg?branch=develop)](https://travis-ci.org/ga4gh-discovery/ga4gh-service-info) [![](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/LICENSE)

Service discovery is at the root of any computational workflow using web-based APIs. Traditionally, this is hard-coded into workflows, and discovery is a manual process. Service-info provides a way for an API to expose a set of metadata to help discovery and aggregation of services via computational methods. It also allows a server/implementation to describe its capabilities and limitations.

This document is intended to be used by service-info implementors and consumers. 

# Specification

Service-info is described in our [OpenAPI specification](./service-info.yaml), which can be visualised using [Swagger Editor](https://editor.swagger.io/?url=https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/service-info.yaml).

## Essentials

All API invocations are made to a configurable HTTP(S) endpoint, receive HTTP headers, and return text or other allowed formatting as requested by the client. Successful requests result with HTTP status code 200 and have the appropriate text encoding in the response body. The client and server may mutually negotiate HTTP/2 upgrade using the standard mechanism.

## Internet Media Types Handling

When responding to a request a server MUST use the fully specified media type for that endpoint. When determining if a request is well-formed, a server MUST allow a internet type to degrade like so:

- application/json; charset=utf-8
  - application/json

No vendor specific description has been given here as service-info intends should be incorporated into other specifications.

### HTTP Response Codes

- `200`: indicates a successful request and response
- `4XX`: indicates a unsuccessful response due to a client error
- `5XX`: indicates a unsuccessful response due to a server error

## Security

Service metadata is viewed as public data and can be provided without restriction. However, an implementation may choose to distribute additional metadata, which may be considered sensitive. Effective security measures are essential to protect the integrity and confidentiality of these data.

Sensitive information transmitted on public networks, such as access tokens and human genomic data, MUST be protected using Transport Level Security (TLS) version 1.2 or later, as specified in RFC 5246.

If the data holder requires client authentication and/or authorization, then the client’s HTTPS API request MUST present an OAuth 2.0 bearer access token as specified in [RFC 6750](https://tools.ietf.org/html/rfc6750), in the `Authorization` request header field with the Bearer authentication scheme:

```
Authorization: Bearer [access_token]
```

The policies and processes used to perform user authentication and authorization, and the means through which access tokens are issued, are beyond the scope of this API specification. GA4GH recommends the use of the OAuth 2.0 framework ([RFC 6749](https://tools.ietf.org/html/rfc6749)) for authentication and authorization.

# Issues, contributions and security

## How to contribute

Guidelines for contributing to this repository are listed in [CONTRIBUTING.md](CONTRIBUTING.md).

## How to notify GA4GH of potential security flaws

Please send an email to security-notification@ga4gh.org.

# Contributors

The following people have contributed to the design of this specification.

- Miro Cupak
- Andy Yates
- Jordi Rambla
- Milan Panik
- Juha Tonroos
- Brian O'Connor
