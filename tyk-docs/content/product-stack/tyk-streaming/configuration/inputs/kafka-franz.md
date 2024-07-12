---
title: Kafka Franz
description: Explains an overview of Kafka input for Kafka franz client
tags: [ "Tyk Streams", "Stream Inputs", "Inputs", "Kafka", "Kafka Franz" ]
---

A Kafka input using the [Franz Kafka client library](https://github.com/twmb/franz-go).

## Common

```yml
# Common config fields, showing default values
input:
  label: ""
  kafka_franz:
    seed_brokers: [] # No default (required)
    topics: [] # No default (required)
    regexp_topics: false
    consumer_group: "" # No default (optional)
    auto_replay_nacks: true
```

## Advanced

```yml
# All config fields, showing default values
input:
  label: ""
  kafka_franz:
    seed_brokers: [] # No default (required)
    topics: [] # No default (required)
    regexp_topics: false
    consumer_group: "" # No default (optional)
    client_id: tyk
    rack_id: ""
    checkpoint_limit: 1024
    auto_replay_nacks: true
    commit_period: 5s
    start_from_oldest: true
    tls:
      enabled: false
      skip_cert_verify: false
      enable_renegotiation: false
      root_cas: ""
      root_cas_file: ""
      client_certs: []
    sasl: [] # No default (optional)
    multi_header: false
    batching:
      count: 0
      byte_size: 0
      period: ""
      check: ""
      processors: [] # No default (optional)
```

When a consumer group is specified this input consumes one or more topics where partitions will automatically balance across any other connected clients with the same consumer group. When a consumer group is not specified topics can either be consumed in their entirety or with explicit partitions.

This input often out-performs the traditional `kafka` input as well as providing more useful logs and error messages.

### Metadata

This input adds the following metadata fields to each message:

``` text
- kafka_key
- kafka_topic
- kafka_partition
- kafka_offset
- kafka_timestamp_unix
- kafka_tombstone_message
- All record headers
```


## Fields

### seed_brokers

A list of broker addresses to connect to in order to establish connections. If an item of the list contains commas it will be expanded into multiple addresses.


Type: `array`  

```yml
# Examples

seed_brokers:
  - localhost:9092

seed_brokers:
  - foo:9092
  - bar:9092

seed_brokers:
  - foo:9092,bar:9092
```

### topics

A list of topics to consume from. Multiple comma separated topics can be listed in a single element. When a `consumer_group` is specified partitions are automatically distributed across consumers of a topic, otherwise all partitions are consumed.

Alternatively, it's possible to specify explicit partitions to consume from with a colon after the topic name, e.g. `foo:0` would consume the partition 0 of the topic foo. This syntax supports ranges, e.g. `foo:0-10` would consume partitions 0 through to 10 inclusive.

Finally, it's also possible to specify an explicit offset to consume from by adding another colon after the partition, e.g. `foo:0:10` would consume the partition 0 of the topic foo starting from the offset 10. If the offset is not present (or remains unspecified) then the field `start_from_oldest` determines which offset to start from.


Type: `array`  

```yml
# Examples

topics:
  - foo
  - bar

topics:
  - things.*

topics:
  - foo,bar

topics:
  - foo:0
  - bar:1
  - bar:3

topics:
  - foo:0,bar:1,bar:3

topics:
  - foo:0-5
```

### regexp_topics

Whether listed topics should be interpreted as regular expression patterns for matching multiple topics. When topics are specified with explicit partitions this field must remain set to `false`.


Type: `bool`  
Default: `false`  

### consumer_group

An optional consumer group to consume as. When specified the partitions of specified topics are automatically distributed across consumers sharing a consumer group, and partition offsets are automatically committed and resumed under this name. Consumer groups are not supported when specifying explicit partitions to consume from in the `topics` field.


Type: `string`  

### client_id

An identifier for the client connection.


Type: `string`  
Default: `"tyk"`  

### rack_id

A rack identifier for this client.


Type: `string`  
Default: `""`  

### checkpoint_limit

Determines how many messages of the same partition can be processed in parallel before applying back pressure. When a message of a given offset is delivered to the output the offset is only allowed to be committed when all messages of prior offsets have also been delivered, this ensures at-least-once delivery guarantees. However, this mechanism also increases the likelihood of duplicates in the event of crashes or server faults, reducing the checkpoint limit will mitigate this.


Type: `int`  
Default: `1024`  

### auto_replay_nacks

Whether messages that are rejected (nacked) at the output level should be automatically replayed indefinitely, eventually resulting in back pressure if the cause of the rejections is persistent. If set to `false` these messages will instead be deleted. Disabling auto replays can greatly improve memory efficiency of high throughput streams as the original shape of the data can be discarded immediately upon consumption and mutation.


Type: `bool`  
Default: `true`  

### commit_period

The period of time between each commit of the current partition offsets. Offsets are always committed during shutdown.


Type: `string`  
Default: `"5s"`  

### start_from_oldest

Determines whether to consume from the oldest available offset, otherwise messages are consumed from the latest offset. The setting is applied when creating a new consumer group or the saved offset no longer exists.


Type: `bool`  
Default: `true`  

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
Requires version 3.45.0 or newer  

### tls.root_cas

An optional root certificate authority to use. This is a string, representing a certificate chain from the parent trusted root certificate, to possible intermediate signing certificates, to the host certificate.
<!-- TODO add secret link :::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
::: -->


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
<!-- TODO add secret link :::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
::: -->


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
<!-- TODO add secret link :::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
::: -->


Type: `string`  
Default: `""`  

```yml
# Examples

password: foo

password: ${KEY_PASSWORD}
```

### sasl

Specify one or more methods of SASL authentication. SASL is tried in order; if the broker supports the first mechanism, all connections will use that mechanism. If the first mechanism fails, the client will pick the first supported mechanism. If the broker does not support any client mechanisms, connections will fail.


Type: `array`  

```yml
# Examples

sasl:
  - mechanism: SCRAM-SHA-512
    password: bar
    username: foo
```

### `sasl[].mechanism`

The SASL mechanism to use.


Type: `string`  

| Option | Summary |
|---|---|
| `AWS_MSK_IAM` | AWS IAM based authentication as specified by the 'aws-msk-iam-auth' java library. |
| `OAUTHBEARER` | OAuth Bearer based authentication. |
| `PLAIN` | Plain text authentication. |
| `SCRAM-SHA-256` | SCRAM based authentication as specified in RFC5802. |
| `SCRAM-SHA-512` | SCRAM based authentication as specified in RFC5802. |
| `none` | Disable sasl authentication |


### sasl[].username

A username to provide for PLAIN or SCRAM-* authentication.


Type: `string`  
Default: `""`  

### sasl[].password

A password to provide for PLAIN or SCRAM-* authentication.
<!-- TODO add secret link :::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
::: -->


Type: `string`  
Default: `""`  

### sasl[].token

The token to use for a single session's OAUTHBEARER authentication.


Type: `string`  
Default: `""`  

### sasl[].extensions

Key/value pairs to add to OAUTHBEARER authentication requests.


Type: `object`  

### `sasl[].aws`

Contains AWS specific fields for when the `mechanism` is set to `AWS_MSK_IAM`.


Type: `object`  

### sasl[].aws.region

The AWS region to target.


Type: `string`  
Default: `""`  

### sasl[].aws.endpoint

Allows you to specify a custom endpoint for the AWS API.


Type: `string`  
Default: `""`  

### sasl[].aws.credentials

Optional manual configuration of AWS credentials to use. 

<!-- TODO add link More information can be found [in this document](/docs/guides/cloud/aws). --> -->


Type: `object`  

### sasl[].aws.credentials.profile

A profile from `~/.aws/credentials` to use.


Type: `string`  
Default: `""`  

### sasl[].aws.credentials.id

The ID of credentials to use.


Type: `string`  
Default: `""`  

### sasl[].aws.credentials.secret

The secret for the credentials being used.
<!-- TODO add secret link :::warning Secret
This field contains sensitive information that usually shouldn't be added to a config directly, read our [secrets page for more info](/docs/configuration/secrets).
::: -->


Type: `string`  
Default: `""`  

### sasl[].aws.credentials.token

The token for the credentials being used, required when using short term credentials.


Type: `string`  
Default: `""`  

### sasl[].aws.credentials.from_ec2_role

Use the credentials of a host EC2 machine configured to assume [an IAM role associated with the instance](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_use_switch-role-ec2.html).


Type: `bool`  
Default: `false`  
Requires version 4.2.0 or newer  

### sasl[].aws.credentials.role

A role ARN to assume.


Type: `string`  
Default: `""`  

### sasl[].aws.credentials.role_external_id

An external ID to provide when assuming a role.


Type: `string`  
Default: `""`  

### multi_header

Decode headers into lists to allow handling of multiple values with the same key


Type: `bool`  
Default: `false`  

### batching

<!-- TODO add batching policy link -->
Allows you to configure a batching policy that applies to individual topic partitions in order to batch messages together before flushing them for processing. Batching can be beneficial for performance as well as useful for windowed processing, and doing so this way preserves the ordering of topic partitions.


Type: `object`  

```yml
# Examples

batching:
  byte_size: 5000
  count: 0
  period: 1s

batching:
  count: 10
  period: 1s

batching:
  check: this.contains("END BATCH")
  count: 0
  period: 1m
```

### batching.count

A number of messages at which the batch should be flushed. If `0` disables count based batching.


Type: `int`  
Default: `0`  

### batching.byte_size

An amount of bytes at which the batch should be flushed. If `0` disables size based batching.


Type: `int`  
Default: `0`  

### batching.period

A period in which an incomplete batch should be flushed regardless of its size.


Type: `string`  
Default: `""`  

```yml
# Examples

period: 1s

period: 1m

period: 500ms
```

### batching.check

A [Bloblang]({{< ref "/product-stack/tyk-streaming/guides/bloblang/overview" >}}) query that should return a boolean value indicating whether a message should end a batch.


Type: `string`  
Default: `""`  

```yml
# Examples

check: this.type == "end_of_transaction"
```

### batching.processors

<!-- TODO add processors link -->
A list of processors to apply to a batch as it is flushed. This allows you to aggregate and archive the batch however you see fit. Please note that all resulting messages are flushed as a single batch, therefore splitting the batch into smaller batches using these processors is a no-op.


Type: `array`  

```yml
# Examples

processors:
  - archive:
      format: concatenate

processors:
  - archive:
      format: lines

processors:
  - archive:
      format: json_array
```
