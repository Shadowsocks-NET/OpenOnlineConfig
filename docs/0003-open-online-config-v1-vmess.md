# Open Online Config 1: VMess

## 1. Overview

This document defines the JSON document format for VMess AEAD configurations in Open Online Config 1.

## 2. API Specification

Request:

``` http
GET /<secret>/ooc/v1/<user_id>
```

Response JSON:

``` json
{
    "protocols": [ "vmess" ],
    "vmess": [
        {
            "id": "27b8a625-4f4b-4428-9f0f-8a2317db7c79",
            "name": "ServerName",
            "address": "example.com",
            "port": 8388,
            "security": "auto",
            "password": "example",
            "stream": {
                "network": "tcp",
                "security": "none",
            },
            "mux": 0,
        }
    ]
}
```

The protocol name is `vmess`. Custom fields are allowed but MUST NOT be related to the protocol itself.

Required fields:

- `id`: UUID of the server. Used by clients to distinguish and persist servers when reloading.
- `name`: Server name.
- `address`: Server address. Do not enclose IPv6 addresses in brackets. Examples: `example.com`, `1.1.1.1`, `2001:db8::1`.
- `port`: Server port number.
- `security`: encryption method. Valid values: `auto`, `zero`, `none`, `aes-128-gcm`, `chacha20-poly1305`.
- `password`: Password. Not the derived key.

Optional fields:

- `stream`: StreamSettingsObject.
    - `network`: Stream Netwokr Type. Valid values: `tcp`, `kcp`, `ws`, `http`, `quic`, `grpc`.
    - `security`: Transport layer encryption. Valid values: `none`, `tls`.
- `mux`: 0 Means disabled. Used by clients to distinguish and persist servers when reloading.