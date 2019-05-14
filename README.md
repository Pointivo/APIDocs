# Introduction

The Pointivo Platform API enables developers to integrate Pointivo's advanced 3D intelligence platform into other applications and services. Client applications can interact with the platform using our HTTP-based RESTful web services. This document describes the structure and use of the API.

## Design Notes

The API design largely adopts a style known as HATEOAS, or "hypertext as the engine of application state" \(see [HATEOAS; Fielding 2000](http://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm)\).

* All external ID comparisons are ordinal and case sensitive.
* All URI strings are returned in [RFC 3986](http://tools.ietf.org/html/rfc3986) format.
* All date strings are returned in the [W3C DateTime profile](http://www.w3.org/TR/NOTE-datetime) of [ISO 8601](http://en.wikipedia.org/wiki/ISO_8601) format.
* All Guid strings are returned in [RFC4122](http://tools.ietf.org/html/rfc4122) format.

## API Versioning

{% hint style="info" %}
More information coming soon.
{% endhint %}

## HTTP Status Codes

All responses originating from the API return meaningful [HTTP status codes](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html). The expected status codes are outlined for each request in our API documentation. The status code returned provides a way for external systems to understand/confirm the state of their request. For example, when a POST is requested, a `201 (Created)` status code means that the caller is guaranteed that the item was, in fact, created. If a `409 (Conflict)` is returned, then the caller knows that something with that identifier already exists and the request was not fulfilled. To aid in the understanding of these status codes, the API will also return additional details \(where applicable\) as a part of the response.

## HTTP Headers

Unless otherwise specified, all operations observe the following conventions regarding HTTP headers:

* All responses have a [Content-Type](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.17) of `application/vnd.pointivo+json; charset=utf-8`.
* All responses include the following headers:
  * `x-pv-request-id` - Contains the request ID. This header is available in all HTTP responses, but it is most useful in error situations.
  * `x-pv-version` - The server software version.
* All error responses also include the following headers:
  * `x-pv-error-code` - Contains the error code.
  * `x-pv-error-resource` - Contains the error resource URI.

## Documentation Notes

* HTTP request and response examples are formatted for readability in this document; however, requests need only to be valid JSON. Responses sent by Pointivo will omit any insignificant whitespace.
* HTTP request and response examples in this document may be modified for brevity.
* Not all response headers are called out explicitly for every operation; some headers are omitted, especially those that occur on every response.

