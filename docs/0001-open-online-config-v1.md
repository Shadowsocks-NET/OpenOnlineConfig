# Open Online Config 1: An HTTPS-based Application Protocol for Configuration Delivery of Censorship Circumvention Services

## 0. Abstract

This document defines version 1 of the Open Online Config protocol. Open Online Config 1 (OOCv1) provides service providers and application developers with a simple and elegant solution for the delivery of censorship circumvention service configurations with unified support for different transport protocols.

## 1. Overview

Open Online Config 1 is an HTTPS-based application protocol for the distribution of censorship circumvention services. The protocol aims to provide a centralized model for the sharing of distributed censorship circumvention services in a community.

Servers are required to implement a basic web API for clients to retrieve configuration in the defined format. Additional API methods may be implemented for advanced operations.

### 1.1. Document Structure

This document describes the Open Online Config protocol version 1 and is structured as follows:

- Section 2 describes transport and delivery details.
- Section 3 defines the API specification, including request and response format.
- Section 4 defines how transport protocols are registered and standardized.

### 1.2. Terms and Definitions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [RFC2119](https://www.rfc-editor.org/info/rfc2119) [RFC8174](https://www.rfc-editor.org/info/rfc8174) when, and only when, they appear in all capitals, as shown here.

Commonly used terms in this document are described below.

- OOCv1: An acronym for the Open Online Config protocol version 1 defined by this document.
- Client: The endpoint that retrieves configuration for use.
- Server: The endpoint that accepts OOCv1 requests and sends configuration.
- Node: A participating server that acts as a transit for Internet traffic.

## 2. Transport and Delivery

Clients communicate with servers by making API transactions.

### 2.1 Transport

The web API MUST use HTTPS as the underlying transport. The minimum TLS version SHOULD be 1.3 to disallow week encryption and protect against TLS downgrade attacks. [HSTS (HTTP Strict Transport Security)](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) SHOULD be deployed on the web server. Optimally, submit the domain name to the [HSTS preload list](https://hstspreload.org/). This prevents browsers from sending the initial request via an unsecured plain HTTP connection if the user enters the domain in the address bar without the protocol prefix.

Certificates from a publicly-trusted CA SHOULD be used. Self-signed certificates MAY be used, in which case an accompanying certificate SHA-256 checksum must be provided for verification.

Clients MUST NOT omit certificate issues or TLS errors to protect against TLS MITM attacks. Clients MAY support importing from or exporting as JSON files as a temporary way of exchanging configuration in case of need.

Servers MUST respond with the correct format, content type and the UTF-8 charater set.

### 2.2 Delivery

To access a web API, the following information is needed:

- Base URL: The base URL for API transactions. The protocol scheme MUST be HTTPS. The URL MUST NOT contain a trailing slash. For example, `https://example.com`, `https://1.1.1.1:853` and `https://[2001:db8::1]:8443` are a valid base URLs. `http://example.com` and `https://example.com/` are invalid for the mentioned reasons.
- Secret: Path to the API endpoint. This is required to conceal the presence of the API. The secret MAY contain zero or more forward slashes (/) to allow flexible path hierarchy. But it's recommended to put non-secret part of the path in the base URL.
- User ID: A distinctive string that represents the user. A UUID or a random secret string MAY be used.
- Certificate SHA-256 Checksum: Optional. Specifying this checksum indicates the use of self-signed certificates. Use lowercase charaters without colons. For example, `0ae384bfd4dde9d13e50c5857c05a442c93f8e01445ee4b34540d22bd1e37f1b`.

An __API token__ MAY be used to convey the API access information in a single string.

``` json
{
    "version": 1,
    "baseUrl": "https://example.com",
    "secret": "8c1da4d8-8684-4a2c-9abb-57b9d5fa7e52",
    "userId": "a117460e-41df-4dbd-b2df-4bd0c16efd2f",
    "certSha256": "0ae384bfd4dde9d13e50c5857c05a442c93f8e01445ee4b34540d22bd1e37f1b"
}
```

The `version` field MUST match the version of the Open Online Config protocol.

The `certSha256` field MUST be present for a self-signed certificate, and MUST be omitted when it's from a publicly-trusted CA. The JSON string MAY be minified to fit in one line.

## 3. API Specification

A `GET` method is required for basic configuration delivery. Implementations MAY implement additional methods.

Request:

``` http
GET /<secret>/ooc/v1/<user_id>
```

Response JSON:

``` json
{
    "username": "nobody",
    "bytesUsed": 274877906944,
    "bytesRemaining": 824633720832,
    "expiryDate": 1625356800,
    "protocols": [ "shadowsocks", "vmess", "trojan-go" ]
}
```

Required fields:

- `protocols`: Protocols utilized by this configuration. Only registered protocols are allowed. See Section 4 for details about protocol registration.

Clients MUST check the `protocols` field against supported protocols. If not all protocols used by the configuration are supported, a warning message MUST be displayed.

Optional fields:

- `username`: Username.
- `bytesUsed`: The amount of data used by the user in bytes. MUST be greater than or equal to 0. An unsigned 64-bit integer MAY be used. Omit this field if no data usage metrics are available.
- `bytesRemaining`: The amount of data remaining to be used by the user in bytes. MUST be greater than or equal to 0. An unsigned 64-bit integer MAY be used. MUST be accompanied by `bytesUsed`. Omit this field if no data limit is in place.
- `expiryDate`: 64-bit Unix Epoch timestamp in seconds. The configuration is valid until this date. Clients MUST pull for updates and reload before this date. If the credentials are updated, the servers MUST stop accepting new connections with expired credentials. Existing connections with expired credentials MUST be maintained.

Custom fields MAY be used to convey additional optional information. They MUST follow the `camelCase` naming convention, and MUST NOT have the same name as any registered protocol object name. The delivered config MUST NOT rely on any custom fields to function.

## 4. Protocol Registration

A transport protocol MUST be registered and approved to become part of the standard.

1. Prerequisites: A person MUST be actively involved in the design and/or implementation of the protocol to qualify for registration. If the protocol is managed by an organization or a company, anyone approved by the entity to represent them is eligible.
2. Registration: Open a pull request in this repository to add the standard document. The standard document MUST be written in English. The proposed changes MUST NOT cause conflicts or compatibility issues with existing configurations.
4. Approval: Once the proposed changes are approved and merged, the registered protocol will officially become part of the standard of Open Online Config 1, and is referred to as `OOCv1:<protocol_name>`. For example, a registered `shadowsocks` protocol would be referred to as `OOCv1:shadowsocks`.
