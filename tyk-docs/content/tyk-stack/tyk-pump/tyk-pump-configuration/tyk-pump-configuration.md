---
date: 2017-03-27T15:47:05+01:00
title: Tyk Pump Configuration
menu:
  main:
    parent: Tyk Pump
weight: 4
url: /tyk-pump/configuration
aliases:
  - /tyk-configuration-reference/tyk-pump-configuration/
  - /configure/tyk-pump-configuration/
---

The Tyk Pump is our Open Source analytics purger that moves the data generated by your Tyk nodes to any back-end. By moving the analytics into your supported database, it allows the Tyk Dashboard to display traffic analytics across all your Tyk Gateways.

## Supported Backends
A list of supported data stores can be found [here]({{< ref "/content/tyk-stack/tyk-pump/other-data-stores.md" >}}).

### Configuration

Please visit the public [GitHub Readme](https://github.com/TykTechnologies/tyk-pump) for instructions on installing and configuring the various Pump backends.

#### Tyk Dashboard

The Tyk Dashboard uses the `mongo-pump-aggregate` collection to display analytics. This is different than the standard `mongo` pump plugin that will store individual analytic items into MongoDB. The aggregate functionality was built to be fast, as querying raw analytics is expensive in large data sets. See [Pump Dashboard Config](/docs/tyk-configuration-reference/tyk-pump-dashboard-config/) for more details.

### Capping analytics data

Tyk Gateways can generate a lot of analytics data. Be sure to read about [capping your Dashboard analytics](/docs/analytics-and-reporting/capping-analytics-data-storage/)

### Omitting the configuration file

From Tyk Pump 1.5.1+, you can configure an environment variable to omit the configuration file with the `TYK_PMP_OMITCONFIGFILE` variable.
This is specially useful when using Docker, since by default, the Tyk Pump has a default configuration file with pre-loaded pumps.

### Sharding analytics to different data sinks

In a multi-organisation deployment, each organisation, team, or environment might have their preferred analytics tooling. This capability allows the Tyk Pump to send analytics for different organisations or various APIs to different destinations. 
E.g.  Org A can send their analytics to MongoDB + DataDog 
while Org B can send their analytics to DataDog + expose the Prometheus metrics endpoint.

#### Configuring the sharded analytics

You can achieve the sharding by setting both allow list and block list, meaning that some data sinks can receive information for all orgs, whereas other data sinks will not receive certain organisation's analytics if it was block listed.

This feature makes use of the field called `filters`, which can be defined per pump. This is its structure:
```
"filters":{
  "api_ids":[],
  "org_ids":[],
  "skip_api_ids":[],
  "skip_org_ids":[]
     }
```
- `api_ids` and `org_ids` works as allow list (APIs and orgs where we want to send the analytic records).
- `skip_api_ids` and `skip_org_ids` works as block list (APIs and orgs where we want to filter out and not send their the analytic records). 

The priority is always blacklisted configurations over whitelisted.

An example of configuration would be:
 ```
"csv": {
  "type": "csv",
  "filters": {
    "org_ids": ["org1","org2"]
  },
  "meta": {
    "csv_dir": "./bar"
  }
},
"elasticsearch": {
  "type": "elasticsearch",
  "filters": {
    "skip_api_ids": ["api_id_1"],
    },
  "meta": {
    "index_name": "tyk_analytics",
    "elasticsearch_url": "https://elasticurl:9243",
    "enable_sniffing": false,
    "document_type": "tyk_analytics",
    "rolling_index": false,
    "extended_stats": false,
    "version": "6"
  }
}
```
With this configuration, all the analytics records related to `org1` or `org2` will go to the `csv` backend and everything but analytics records from `api_id_1` to `elasticsearch`.