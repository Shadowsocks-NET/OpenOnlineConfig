# Open Online Config 1: Shadowsocks

## 1. Overview

Shadowsocks is a simple stateless proxy protocol that uses symmetric key ciphers to encrypt network traffic.

This document defines the JSON document format for Shadowsocks AEAD configurations in Open Online Config 1, with support for plugins.

## 2. API Specification

Request:

``` http
GET /<secret>/ooc/v1/<user_id>
```

Response JSON:

``` json
{
    "protocols": [ "shadowsocks" ],
    "shadowsocks": [
        {
            "id": "27b8a625-4f4b-4428-9f0f-8a2317db7c79",
            "name": "ServerName",
            "address": "example.com",
            "port": 8388,
            "method": "chacha20-ietf-poly1305",
            "password": "example",
            "pluginName": "plugin-name",
            "pluginVersion": "1.0",
            "pluginOptions": "whatever",
            "pluginArguments": "-vvvvvv"
        }
    ]
}
```

The protocol name is `shadowsocks`. Shadowsocks servers are placed in an array named `shadowsocks`. Only Shadowsocks AEAD servers are allowed. Custom fields are allowed but MUST NOT be related to the protocol itself.

Required fields:

- `id`: UUID of the server. Used by clients to distinguish and persist servers when reloading.
- `name`: Server name.
- `address`: Server address. Do not enclose IPv6 addresses in brackets. Examples: `example.com`, `1.1.1.1`, `2001:db8::1`.
- `port`: Server port number.
- `method`: AEAD encryption method. Valid values: `aes-128-gcm`, `aes-256-gcm`, `chacha20-ietf-poly1305`.
- `password`: Password. Not the derived key.

Optional plugin fields:

- `pluginName`: Plugin name. Required when using a plugin.
- `pluginVersion`: Required version of the plugin. Optional when using a plugin.
- `pluginOptions`: Value of environment variable `SS_PLUGIN_OPTIONS`. Optional when using a plugin.
- `pluginArguments`: Plugin executable startup arguments. Optional when using a plugin.

Implementations MUST follow [SIP003](https://shadowsocks.org/en/wiki/Plugin.html), with the addition of `pluginArguments` field to allow more flexible configurations.

Plugin support at both sides is optional. But clients without plugin support MUST be able to recognize servers with plugins and reject them.

Client implementations with plugin support MUST maintain a user-configurable list of locally installed plugins with their name, version, and binary path on it. When connecting to a plugin-enabled server, the client resolves the plugin from the list by matching plugin name and version.

If a server doesn't use a plugin, all plugin fields MUST be omitted.
