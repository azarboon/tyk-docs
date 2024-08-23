---
title: While
description: Explains an overview of while processor
tags: [ "Tyk Streams", "Stream Processors", "Processors", "While" ]
---

A processor that checks a [Bloblang query]({{< ref "/product-stack/tyk-streaming/guides/bloblang/overview" >}}) against each batch of messages and executes child processors on them for as long as the query resolves to true.

## Common

```yml
# Common config fields, showing default values
label: ""
while:
  at_least_once: false
  check: ""
  processors: [] # No default (required)
```

## Advanced

```yml
# All config fields, showing default values
label: ""
while:
  at_least_once: false
  max_loops: 0
  check: ""
  processors: [] # No default (required)
```

The field [at_least_once](#at_least_once), if true, ensures that the child processors are always executed at least one time (like a do .. while loop.)

The field [max_loops](#max_loops), if greater than zero, caps the number of loops for a message batch to this value.

If following a loop execution the number of messages in a batch is reduced to zero the loop is exited regardless of the condition result. If following a loop execution there are more than 1 message batches the query is checked against the first batch only.

The conditions of this processor are applied across entire message batches using [batching]({{< ref "/product-stack/tyk-streaming/configuration/common-configuration/batching" >}}).

## Fields

### at_least_once

Whether to always run the child processors at least one time.


Type: `bool`  
Default: `false`  

### max_loops

An optional maximum number of loops to execute. Helps protect against accidentally creating infinite loops.


Type: `int`  
Default: `0`  

### check

A [Bloblang query]({{< ref "/product-stack/tyk-streaming/guides/bloblang/overview" >}}) that should return a boolean value indicating whether the while loop should execute again.


Type: `string`  
Default: `""`  

```yml
# Examples

check: errored()

check: this.urls.unprocessed.length() > 0
```

### processors

A list of child processors to execute on each loop.


Type: `array`  