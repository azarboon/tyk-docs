---
title: Nats
description: Explains an overview of nats input
tags: [ "Tyk Streams", "Stream Inputs", "Inputs", "Nats" ]
---

Subscribe to a NATS subject.

### Common

```yml
# Common config fields, showing default values
input:
  label: ""
  nats:
    urls: [] # No default (required)
    subject: foo.bar.baz # No default (required)
    queue: "" # No default (optional)
    auto_replay_nacks: true
```

### Advanced
```yml
# All config fields, showing default values
input:
  label: ""
  nats:
    urls: [] # No default (required)
    subject: foo.bar.baz # No default (required)
    queue: "" # No default (optional)
    auto_replay_nacks: true
    nak_delay: 1m # No default (optional)
    prefetch_count: 524288
    tls:
      enabled: false
      skip_cert_verify: false
      enable_renegotiation: false
      root_cas: ""
      root_cas_file: ""
      client_certs: []
    auth:
      nkey_file: ./seed.nk # No default (optional)
      user_credentials_file: ./user.creds # No default (optional)
      user_jwt: "" # No default (optional)
      user_nkey_seed: "" # No default (optional)
    extract_tracing_map: root = @ # No default (optional)
```

### Metadata

This input adds the following metadata fields to each message:

``` text
- nats_subject
- nats_reply_subject
- All message headers (when supported by the connection)
```

You can access these metadata fields using [function interpolation]({{< ref "/product-stack/tyk-streaming/configuration/common-configuration/interpolation" >}}).

### Connection Name

When monitoring and managing a production NATS system, it is often useful to
know which connection a message was send/received from. This can be achieved by
setting the connection name option when creating a NATS connection.

Tyk Streams will automatically set the connection name based off the label of the given
NATS component, so that monitoring tools between NATS and Tyk Streams can stay in sync.
### Authentication

There are several components within Tyk Streams which utilize NATS services. You will find that each of these components
support optional advanced authentication parameters for [NKeys](https://docs.nats.io/nats-server/configuration/securing_nats/auth_intro/nkey_auth)
and [User Credentials](https://docs.nats.io/developing-with-nats/security/creds).

An in depth tutorial can be found [here](https://docs.nats.io/running-a-nats-service/nats_admin/security/jwt).

#### NKey file

The NATS server can use these NKeys in several ways for authentication. The simplest is for the server to be configured
with a list of known public keys and for the clients to respond to the challenge by signing it with its private NKey
configured in the `nkey_file` field.

More details [here](https://docs.nats.io/developing-with-nats/security/nkey).

#### User Credentials

NATS server supports decentralized authentication based on JSON Web Tokens (JWT). Clients need an [user JWT](https://docs.nats.io/nats-server/configuration/securing_nats/jwt#json-web-tokens)
and a corresponding [NKey secret](https://docs.nats.io/developing-with-nats/security/nkey) when connecting to a server
which is configured to use this authentication scheme.

The `user_credentials_file` field should point to a file containing both the private key and the JWT and can be
generated with the [nsc tool](https://docs.nats.io/nats-tools/nsc).

Alternatively, the `user_jwt` field can contain a plain text JWT and the `user_nkey_seed`can contain
the plain text NKey Seed.

More details [here](https://docs.nats.io/developing-with-nats/security/creds).

## Fields

### urls

A list of URLs to connect to. If an item of the list contains commas it will be expanded into multiple URLs.


Type: `array`  

```yml
# Examples

urls:
  - nats://127.0.0.1:4222

urls:
  - nats://username:password@127.0.0.1:4222
```

### subject

A subject to consume from. Supports wildcards for consuming multiple subjects. Either a subject or stream must be specified.


Type: `string`  

```yml
# Examples

subject: foo.bar.baz

subject: foo.*.baz

subject: foo.bar.*

subject: foo.>
```

### queue

An optional queue group to consume as.


Type: `string`  

### auto_replay_nacks

Whether messages that are rejected (nacked) at the output level should be automatically replayed indefinitely, eventually resulting in back pressure if the cause of the rejections is persistent. If set to `false` these messages will instead be deleted. Disabling auto replays can greatly improve memory efficiency of high throughput streams as the original shape of the data can be discarded immediately upon consumption and mutation.


Type: `bool`  
Default: `true`  

### nak_delay

An optional delay duration on redelivering a message when negatively acknowledged.


Type: `string`  

```yml
# Examples

nak_delay: 1m
```

### prefetch_count

The maximum number of messages to pull at a time.


Type: `int`  
Default: `524288`  

### `tls`

Custom TLS settings can be used to override system defaults.


Type: `object`  

### tls.enabled

Whether custom TLS settings are enabled.


Type: `bool`  
Default: `false`  

### tls.skip_cert_verify

Whether to skip server side certificate verification.


Type: `bool`  
Default: `false`  

### tls.enable_renegotiation

Whether to allow the remote server to repeatedly request renegotiation. Enable this option if you're seeing the error message `local error: tls: no renegotiation`.


Type: `bool`  
Default: `false`  
  

### tls.root_cas

An optional root certificate authority to use. This is a string, representing a certificate chain from the parent trusted root certificate, to possible intermediate signing certificates, to the host certificate.


Type: `string`  
Default: `""`  

```yml
# Examples

root_cas: |-
  -----BEGIN CERTIFICATE-----
  ...
  -----END CERTIFICATE-----
```

### tls.root_cas_file

An optional path of a root certificate authority file to use. This is a file, often with a .pem extension, containing a certificate chain from the parent trusted root certificate, to possible intermediate signing certificates, to the host certificate.


Type: `string`  
Default: `""`  

```yml
# Examples

root_cas_file: ./root_cas.pem
```

### tls.client_certs

A list of client certificates to use. For each certificate either the fields `cert` and `key`, or `cert_file` and `key_file` should be specified, but not both.


Type: `array`  
Default: `[]`  

```yml
# Examples

client_certs:
  - cert: foo
    key: bar

client_certs:
  - cert_file: ./example.pem
    key_file: ./example.key
```

### tls.client_certs[].cert

A plain text certificate to use.


Type: `string`  
Default: `""`  

### tls.client_certs[].key

A plain text certificate key to use.


Type: `string`  
Default: `""`  

### tls.client_certs[].cert_file

The path of a certificate to use.


Type: `string`  
Default: `""`  

### tls.client_certs[].key_file

The path of a certificate key to use.


Type: `string`  
Default: `""`  

### tls.client_certs[].password

A plain text password for when the private key is password encrypted in PKCS#1 or PKCS#8 format. The obsolete `pbeWithMD5AndDES-CBC` algorithm is not supported for the PKCS#8 format. Warning: Since it does not authenticate the ciphertext, it is vulnerable to padding oracle attacks that can let an attacker recover the plaintext.


Type: `string`  
Default: `""`  

```yml
# Examples

password: foo

password: ${KEY_PASSWORD}
```

### auth

Optional configuration of NATS authentication parameters.


Type: `object`  

### auth.nkey_file

An optional file containing a NKey seed.


Type: `string`  

```yml
# Examples

nkey_file: ./seed.nk
```

### auth.user_credentials_file

An optional file containing user credentials which consist of an user JWT and corresponding NKey seed.


Type: `string`  

```yml
# Examples

user_credentials_file: ./user.creds
```

### auth.user_jwt

An optional plain text user JWT (given along with the corresponding user NKey Seed).


Type: `string`  

### auth.user_nkey_seed

An optional plain text user NKey Seed (given along with the corresponding user JWT).


Type: `string`  

### extract_tracing_map

EXPERIMENTAL: A [Bloblang]({{< ref "/product-stack/tyk-streaming/guides/bloblang/overview" >}}) mapping that attempts to extract an object containing tracing propagation information, which will then be used as the root tracing span for the message. The specification of the extracted fields must match the format used by the service wide tracer.


Type: string 
  

```yml
# Examples

extract_tracing_map: root = @

extract_tracing_map: root = this.meta.span
```
