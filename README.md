![](https://www.ga4gh.org/wp-content/themes/ga4gh-theme/gfx/GA-logo-horizontal-tag-RGB.svg)

# GA4GH service-info specification <a href="https://github.com/ga4gh-discovery/ga4gh-service-info/blob/develop/service-info.yaml"><img src="http://validator.swagger.io/validator?url=https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/service-info.yaml" alt="Swagger Validator" height="20em" width="72em"></a> [![](https://travis-ci.org/ga4gh-discovery/ga4gh-service-info.svg?branch=develop)](https://travis-ci.org/ga4gh-discovery/ga4gh-service-info) [![](https://img.shields.io/badge/license-Apache%202-blue.svg)](https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/LICENSE)

Service discovery is at the root of any computational workflow using web-based APIs. Traditionally, this is hard-coded into workflows, and discovery is a manual process. Service-info provides a way for an API to expose a set of metadata to help discovery and aggregation of services via computational methods. It also allows a server/implementation to describe its capabilities and limitations.

This document is intended to be used by service-info implementors and consumers.

## Specification

Service-info is described in our [OpenAPI specification](./openapi/openapi.yaml), which can be visualised in our [Redoc documentation](https://ga4gh-discovery.github.io/ga4gh-service-info/docs/).

### Essentials

All API invocations are made to a configurable HTTP(S) endpoint, receive HTTP headers, and return text or other allowed formatting as requested by the client. Successful requests result with HTTP status code 200 and have the appropriate text encoding in the response body. The client and server may mutually negotiate HTTP/2 upgrade using the standard mechanism.

### Errors

The service info specification only defines a response for a successful request i.e. a request that responds with a 200 payload. All other error codes and payloads reported are left to the implementing service.

### How to use and extend this specification

This specification is meant to be included in other GA4GH specifications to provide the `/service-info` endpoint. The simplest way to perform this inclusion is by adding the following to your OAS 3 specification file:

```yaml
paths:
  /service-info:
    # Links to the latest version of service-info. Use a tag once service-info is released instead.
    $ref: 'https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/service-info.yaml#/paths/~1service-info'
```

Sometimes, you might want to customize the path information, e.g. provide a more detailed description, prescribe a particular service type for your specification, or provide more error responses. To do that, refer to a schema instead of a path, such as (supposing we have an API called `ga4gh-service`):

```yaml
paths:
  /service-info:
    get:
      summary: 'Show information about this ga4gh-service instance'
      operationId: getServiceInfo
      tags:
        - service-info
      responses:
        '200':
          description: 'Use `type: org.ga4gh:ga4gh-service:1.0.0` when implementing this specification.'
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Service'
  schemas:
    Service:
      # Links to the latest version of service-info. Use a tag once service-info is released instead.
      $ref: 'https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/service-info.yaml#/components/schemas/Service'
```

### Extending service info payloads

Your OAS 3 API might want to define additional service information as well. To do that, use the `allOf` OAS 3 construct. Suppose, for example, we want to add a new required string field named `mycustomfield`:

```yaml
    Service:
      allOf:
        - $ref: 'https://raw.githubusercontent.com/ga4gh-discovery/ga4gh-service-info/develop/service-info.yaml#/components/schemas/Service'
        - type: object
          properties:
            mycustomfield:
              type: string
          required:
            - mycustomfield
```

Using this method you can communicate to clients additional information about a service.

### How to implement this specification

While you can implement this specification directly to provide GA4GH-compatible information about your service, the API is primarily designed to be included in other GA4GH specifications, and you'll likely implement it indirectly through an upstream API.

As such, we recommend your service doesn't refer to service-info directly as part of its `type` information - use the type of the upstream API instead, which should pin a particular version of this specification.

As an example of implementation, feel free to check out the [service registry reference implementation](https://github.com/ga4gh-discovery/ga4gh-service-registry-impl).

### Internet Media Types Handling

When responding to a request a server MUST use the fully specified media type for that endpoint. When determining if a request is well-formed, a server MUST allow a media type to degrade like so:

- application/json; charset=utf-8
  - application/json

No vendor specific description has been given here as service-info intends to be incorporated into other specifications.

### Security

Service metadata is viewed as public data and can be provided without restriction. However, an implementation may choose to distribute additional metadata, which may be considered sensitive. Effective security measures are essential to protect the integrity and confidentiality of this data.

Sensitive information transmitted over public networks, such as access tokens and human genomic data, MUST be protected using Transport Level Security (TLS) version 1.2 or later, as specified in [RFC 5246](https://tools.ietf.org/html/rfc5246).

If the data holder requires client authentication and/or authorization, then the client’s HTTPS API request MUST present an OAuth 2.0 bearer access token as specified in [RFC 6750](https://tools.ietf.org/html/rfc6750), in the `Authorization` request header field with the Bearer authentication scheme:

```
Authorization: Bearer [access_token]
```

The policies and processes used to perform user authentication and authorization, and the means through which access tokens are issued, are beyond the scope of this API specification. GA4GH recommends the use of the [OpenID Connect](https://openid.net/connect/) and [OAuth 2.0 framework (RFC 6749)](https://tools.ietf.org/html/rfc6749) for authentication and authorization.

### CORS

Cross-origin resource sharing (CORS) is an essential technique used to overcome the same origin content policy seen in browsers. This policy restricts a webpage from making a request to another website and leaking potentially sensitive information. However the same origin policy is a barrier to using open APIs. GA4GH open API implementers should enable CORS to an acceptable level as defined by their internal policy. For any public API implementations should allow requests from any server.

GA4GH is publishing a [CORS best practices document](https://docs.google.com/document/d/1Ifiik9afTO-CEpWGKEZ5TlixQ6tiKcvug4XLd9GNcqo/edit?usp=sharing), which implementers should refer to for guidance when enabling CORS on public API instances.

## How to contribute

Guidelines for contributing to this repository are listed in [CONTRIBUTING.md](CONTRIBUTING.md).

## How to notify GA4GH of potential security flaws

Please send an email to security-notification@ga4gh.org.

## FAQ

### Should my service specify the version of the service-info specification it implements as part of its type?

No, this API is designed to be included in other GA4GH specifications, and you probably want to implement it indirectly through an upstream specification. See [How to implement this specification](#how-to-implement-this-specification).

### How do I describe a service implementing multiple specifications?

We're in the era of microservices and we don't really care distinguish between cases when multiple specifications are implemented by a single big service or multiple small services. We recommend you provide a `/service-info` endpoint for each or the APIs your service implements, relative to the root of the respective implementation of the specification.

For example, let's say you have a service `foobar`, running at `https://foobar.example.com`, which implements APIs `foo` and `bar`. The roots of `foo` and `bar` implementations might be `https://foobar.example.com/foo` and `https://foobar.example.com/bar`, respectively. If `foo` and `bar` are approved GA4GH standards, they'll extend the service-info API and specify their respective requirements on the response contents. Regardless, your service will expose two `/service-info` endpoints, one for each API: `https://foobar.example.com/foo/service-info` and `https://foobar.example.com/bar/service-info`.

If you want your service to advertise multiple `/service-info` endpoints it provides, we recommend you implement a [Service Registry](https://github.com/ga4gh-discovery/ga4gh-service-registry). Using the `foobar` example service, this would include e.g. a `https://foobar.example.com/services` endpoint.

### How do I describe a service implementing a chain of specifications?

If your service implements multiple specifications transitively, its `type` should reflect the most specialized one. For example, let's say we have specifications `foo`, `bar` and `baz`, where `bar` extends `baz`, and `foo` extends `bar`. Your service implements `foo`, and thus transitively also `bar` and `baz`. Its type should refer to (a version of) `foo`.

If you're dealing with a hierarchy of specifications that don't form a perfect chain, i.e. you're implementing multiple specifications with a common parent, consider providing [multiple /service-info endpoints](#how-do-i-describe-a-service-implementing-multiple-specifications).

### Can I use this specification with my custom, non-GA4GH APIs?

Definitely! Just specify a custom group and artifact as per description of the `type` object.

## Contributors

The following people have contributed to the design of this specification.

- Miro Cupak
- Andy Yates
- Jordi Rambla
- Milan Panik
- Juha Tonroos
- Brian O'Connor
