# Appendix A. Pulsar Admin API

Throughout the book, I’ve used the Pulsar Admin REST API to perform many tasks. I shared the URLs and the API methods but didn’t spend much time talking about the REST API. That was partly due to it not fitting into the flow of the book, but also knowing I would reserve this section of the book to talk about it. This appendix will quickly cover the Pulsar Admin API and go through all the available methods in the API. It’s here to serve as a reference (see [Table A-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app01.html#available_methods_on_the_pulsar_admin_a)).

For an exhaustive list of what is available in the Pulsar Admin API, you can consult the [official API reference](https://oreil.ly/U4Wml).

| Object                | Methods                                                      |
| :-------------------- | :----------------------------------------------------------- |
| `Bookies`             | Methods for getting information about currently running bookies as well as editing and deleting metadata about running bookies |
| `Broker-Stats`        | Methods for retrieving statistics about brokers in the cluster |
| `Brokers`             | Methods for getting, deleting, and setting broker-level configurations |
| `Clusters`            | Methods for getting, deleting, and setting cluster-level configurations |
| `Namespaces`          | Methods for getting, deleting, creating, and setting namespaces |
| `Nonpersistent topic` | Methods for retrieving information on nonpersistent topics as well as creating and editing them |
| `Persistent topic`    | Methods for retrieving information on persistent topics as well as creating and editing them |
| `Resource-Quotas`     | Methods for retrieving and setting resource quotas           |
| `ResourceGroups`      | Methods for getting, creating, and deleting resource groups  |
| `Schemas`             | Methods for retrieving, creating, editing, and deleting schemas |
| `Tenants`             | Methods for retrieving, creating, editing, and deleting tenants |

# Use Cases

The Pulsar Admin API can perform any function that would require using the CLI, but can do so via RESTful semantics. REST APIs can be more easily exposed to the internet and are useful for automation, auditing, and building integrations. Following are some useful things that have been done with the Pulsar Admin API:

- Billing integration

  In a company I worked at previously, we collected usage statistics from the metrics API as well as metadata from the REST API to create a dashboard for usage of our Pulsar cluster. This dashboard showed us estimated costs per topic based on our cloud costs and usage in the cluster.

- Topic creation

  I’ve used the REST API to ensure that topics were only created by authorized parties and in the correct semantics. As part of a merge request, developers needed to include the REST command used to create their topic, and we validated it before approving the merge request. Additionally, our automated CI/CD system would run the REST command and check it for correctness.

- Auditing

  At a previous company, we got started using the Pulsar Admin CLI and did not have any tight controls on administration. After some time, we needed to audit what was out there and we used the Pulsar Admin API to generate lists of topics and their metadata as well as other resources. We were able to use this list to form policies and enforce them going forward.

# Examples

Here are a few examples of using the Pulsar Admin REST API to perform some useful work in a cluster. It’s not an exhaustive list, but just intended to give you an idea of how the API bodies and methods work.

## Creating a Partitioned Topic

This command creates a partitioned topic with 10 partitions:

```
PUT https://pulsar.apache.org/admin/v2/non-persistent/
  {tenant}/{namespace}/{topic}/partitions
-body 10
```

## Deleting a Partitioned Topic

This command will force the deletion of a nonpersistent and partitioned topic:

```
DELETE https://pulsar.apache.org/admin/v2/non-persistent/
  {tenant}/{namespace}/{topic}/partitions?force=true
```

## Creating a Namespace with Specific Policies

With this command you can set every possible configuration for a namespace. You’ll notice many of the values look familiar, as we’ve used them throughout the book. You can even set arbitrary properties which can be useful for tracking company-specific information about a namespace:

```
PUT https://pulsar.apache.org/admin/v2/namespaces/{tenant}/{namespace}

Body 

{
  "auth_policies": {
    "topicAuthentication": {
      "property1": {
        "property1": [
          "produce"
        ],
        "property2": [
          "produce"
        ]
      },
      "property2": {
        "property1": [
          "produce"
        ],
        "property2": [
          "produce"
        ]
      }
    },
    "subscriptionAuthentication": {
      "property1": [
        "string"
      ],
      "property2": [
        "string"
      ]
    },
    "namespaceAuthentication": {
      "property1": [
        "produce"
      ],
      "property2": [
        "produce"
      ]
    }
  },
  "replication_clusters": [
    "string"
  ],
  "bundles": {
    "boundaries": [
      "string"
    ],
    "numBundles": 0
  },
  "backlog_quota_map": {
    "property1": {
      "limitSize": 0,
      "limitTime": 0,
      "policy": "producer_request_hold"
    },
    "property2": {
      "limitSize": 0,
      "limitTime": 0,
      "policy": "producer_request_hold"
    }
  },
  "clusterDispatchRate": {
    "property1": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    },
    "property2": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    }
  },
  "topicDispatchRate": {
    "property1": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    },
    "property2": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    }
  },
  "subscriptionDispatchRate": {
    "property1": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    },
    "property2": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    }
  },
  "replicatorDispatchRate": {
    "property1": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    },
    "property2": {
      "dispatchThrottlingRateInMsg": 0,
      "dispatchThrottlingRateInByte": 0,
      "relativeToPublishRate": true,
      "ratePeriodInSecond": 0
    }
  },
  "clusterSubscribeRate": {
    "property1": {
      "subscribeThrottlingRatePerConsumer": 0,
      "ratePeriodInSecond": 0
    },
    "property2": {
      "subscribeThrottlingRatePerConsumer": 0,
      "ratePeriodInSecond": 0
    }
  },
  "persistence": {
    "bookkeeperEnsemble": 0,
    "bookkeeperWriteQuorum": 0,
    "bookkeeperAckQuorum": 0,
    "managedLedgerMaxMarkDeleteRate": 0
  },
  "deduplicationEnabled": true,
  "autoTopicCreationOverride": {
    "topicType": "string",
    "defaultNumPartitions": 0,
    "allowAutoTopicCreation": true
  },
  "autoSubscriptionCreationOverride": {
    "allowAutoSubscriptionCreation": true
  },
  "publishMaxMessageRate": {
    "property1": {
      "publishThrottlingRateInMsg": 0,
      "publishThrottlingRateInByte": 0
    },
    "property2": {
      "publishThrottlingRateInMsg": 0,
      "publishThrottlingRateInByte": 0
    }
  },
  "latency_stats_sample_rate": {
    "property1": 0,
    "property2": 0
  },
  "message_ttl_in_seconds": 0,
  "subscription_expiration_time_minutes": 0,
  "retention_policies": {
    "retentionTimeInMinutes": 0,
    "retentionSizeInMB": 0
  },
  "deleted": true,
  "encryption_required": true,
  "delayed_delivery_policies": {
    "tickTime": 0,
    "active": true
  },
  "inactive_topic_policies": {
    "inactiveTopicDeleteMode": "delete_when_no_subscriptions",
    "maxInactiveDurationSeconds": 0,
    "deleteWhileInactive": true
  },
  "subscription_auth_mode": "None",
  "max_producers_per_topic": 0,
  "max_consumers_per_topic": 0,
  "max_consumers_per_subscription": 0,
  "max_unacked_messages_per_consumer": 0,
  "max_unacked_messages_per_subscription": 0,
  "max_subscriptions_per_topic": 0,
  "compaction_threshold": 0,
  "offload_threshold": 0,
  "offload_deletion_lag_ms": 0,
  "max_topics_per_namespace": 0,
  "schema_auto_update_compatibility_strategy": "AutoUpdateDisabled",
  "schema_compatibility_strategy": "UNDEFINED",
  "is_allow_auto_update_schema": true,
  "schema_validation_enforced": true,
  "offload_policies": {
    "managedLedgerOffloadPrefetchRounds": 0,
    "s3ManagedLedgerOffloadRegion": "string",
    "s3ManagedLedgerOffloadBucket": "string",
    "s3ManagedLedgerOffloadServiceEndpoint": "string",
    "s3ManagedLedgerOffloadMaxBlockSizeInBytes": 0,
    "s3ManagedLedgerOffloadReadBufferSizeInBytes": 0,
    "s3ManagedLedgerOffloadCredentialId": "string",
    "s3ManagedLedgerOffloadCredentialSecret": "string",
    "s3ManagedLedgerOffloadRole": "string",
    "s3ManagedLedgerOffloadRoleSessionName": "string",
    "gcsManagedLedgerOffloadRegion": "string",
    "gcsManagedLedgerOffloadBucket": "string",
    "gcsManagedLedgerOffloadMaxBlockSizeInBytes": 0,
    "gcsManagedLedgerOffloadReadBufferSizeInBytes": 0,
    "gcsManagedLedgerOffloadServiceAccountKeyFile": "string",
    "fileSystemProfilePath": "string",
    "fileSystemURI": "string",
    "managedLedgerOffloadBucket": "string",
    "managedLedgerOffloadedReadPriority": "BOOKKEEPER_FIRST",
    "managedLedgerOffloadRegion": "string",
    "managedLedgerOffloadServiceEndpoint": "string",
    "managedLedgerOffloadMaxBlockSizeInBytes": 0,
    "managedLedgerOffloadReadBufferSizeInBytes": 0,
    "managedLedgerOffloadThresholdInBytes": 0,
    "managedLedgerOffloadDeletionLagInMillis": 0,
    "managedLedgerOffloadDriver": "string",
    "offloadersDirectory": "string",
    "managedLedgerOffloadMaxThreads": 0
  },
  "deduplicationSnapshotIntervalSeconds": 0,
  "subscription_types_enabled": [
    "string"
  ],
  "properties": {
    "property1": "string",
    "property2": "string"
  },
  "resource_group_name": "string"
}
```

## Deleting a Namespace

This command will delete a namespace and all the topics in that namespace (use with caution):

```
DELETE https://pulsar.apache.org/admin/v2/namespaces/{tenant}/{namespace}
```

# Summary

Pulsar has an undeniably rich REST Admin API. As an operator of Pulsar, you can perform any task you need to from the REST Admin API. However, with great power comes great responsibility. Before running Pulsar at scale and using the Admin API, it’s recommended that you spend some time familiarizing yourself with what the API is capable of as well as what it means. You should also follow authorization best practices to prevent someone from using the REST Admin API to do irreversible damage in the cluster.