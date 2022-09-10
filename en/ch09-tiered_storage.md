# Chapter 9. Tiered Storage

Pulsar has immutable storage as a primitive and interfaces for interacting with that storage via client libraries, Pulsar Functions, and Pulsar IO. This means we can do things like replay topic data from the beginning (or at a specific offset) as well as handle events like consumers going offline for periods of time. Supporting high-performance writing and retrieval of data in Pulsar requires the bookies in a Pulsar cluster to utilize disk technology, which is expensive relative to alternatives. In [Chapter 5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch05.html#consumers), you learned how Pulsar operators can ensure that data is deleted when it’s no longer used. What if we want to keep the data indefinitely but store it in a more cost-effective way?

Pulsar tiered storage is a mechanism for offloading data that doesn’t have immediate value on BookKeeper to a cheaper and more flexible storage solution (see [Figure 9-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#in_pulsarapostrophes_tiered_storage_eco)).

![In Pulsar’s tiered storage ecosystem, data moves from bookies to other services like object storage or distributed file storage for long-term data storage needs.](../img/mapu_0901.png)

*Figure 9-1. In Pulsar’s tiered storage ecosystem, data moves from bookies to other services like object storage or distributed file storage for long-term data storage needs.*



In this chapter we’ll cover some of the motivation for tiered storage and discuss how to set up tiered storage using Pulsar Admin APIs and the console.

# Storing Data in the Cloud

In the world of data storage, not all mechanisms of storing data are considered equal. Each approach to storing data is a trade-off between cost, efficiency, and fault tolerance. Since Pulsar is a cloud-native technology, we’ll focus on the following three cloud mechanisms for storing data:

- File
- Block
- Object

[Table 9-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#summary_of_cloud-based_storage_solution) provides a summary of common ways these three data storage mechanisms are used in the cloud.

| Storage mechanism | Cost | Reliability                                               | Use cases                      |
| :---------------- | :--- | :-------------------------------------------------------- | :----------------------------- |
| File              | $$   | Great reliability in cloud settings.                      | Network attached storage.      |
| Block             | $$$  | Great reliability in cloud settings.                      | Cloud server storage (hot).    |
| Object            | $    | Cloud applications are impeccable, reliable, and durable. | Cold storage/big data storage. |

The most common type of storage for a BookKeeper cluster is block storage. Popular public cloud providers like Amazon Web Services and Azure Cloud typically bundle block storage with their server offerings. Block storage is the most expensive of the three types, but it provides an easy way to scale up when needed for the cloud environment. As we consider a Pulsar cluster, when the storage needs of the cluster grow, we will encounter more costs in terms of servers and block storage (see [Figure 9-2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#pricing_tiersdot_block_storage_left_par)).

![Pricing tiers. Block storage (typically what’s found for a cloud application) is the most expensive because it has performance requirements for quick writing and reading. File storage is cheaper as it has fewer requirements than block storage. Object storage is even cheaper and is suitable for long-term storage.](../img/mapu_0902.png)

*Figure 9-2. Pricing tiers. Block storage (typically what’s found for a cloud application) is the most expensive because it has performance requirements for quick writing and reading. File storage is cheaper as it has fewer requirements than block storage. Object storage is even cheaper and is suitable for long-term storage.*



It would be ideal if, as our needs changed in the cloud, we could move our data to a cheaper storage solution. As our application matures, we may have different needs to meet in our Pulsar cluster. For example, we may keep higher performance data for our clusters on block storage and, as the data becomes stale, we move it to file (the file system on your laptop) or block storage (see Figures [9-3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#bookkeeper_allows_for_a_scale-out_archi) and [9-6](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#an_example_of_how_pulsar_tiered_storage)).

![BookKeeper allows for a scale-out architecture, but as we add more nodes to our cluster, the costs associated with the server and storage also grow.](../img/mapu_0903.png)

*Figure 9-3. BookKeeper allows for a scale-out architecture, but as we add more nodes to our cluster, the costs associated with the server and storage also grow.*



What we need is a way to reserve precious cluster resources (block storage) and move old data from Pulsar to another system. Pulsar tiered storage provides a way to move data from BookKeeper (where data is stored in Pulsar) to other storage mechanisms, including object storage and file storage.

While the file storage option for offloading data in Pulsar is interesting, we’re going to focus on object storage because object storage is intentionally built to store data of many types and has APIs for solving a wide range of problems. It’s worth taking a moment to dive into some of the solutions object stores provide in the wider data ecosystem.

## Object Storage

Object storage is not a new concept, having originated in the early 1990s.[^i]Object stores have a different paradigm from file storage in that an object is composed of:

- Data (the actual bytes that need to be stored)
- Metadata (arbitrary values about the object)
- Permissions (who can access the object)
- An ID (a unique identifier for the object)

[^i]: M. Factor, K. Meth, D. Naor, O. Rodeh, and J. Satran, “Object storage: The future building block for storage systems,” in *2005 IEEE International Symposium on Mass Storage Systems and Technology* (New York: IEEE, 2005): 119–123, doi: 10.1109/LGDI.2005.1612479. 



Instead of being stored in a hierarchical filesystem, objects are stored in what are called buckets (see [Figure 9-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#an_example_of_object_storagedot_objects)).

![An example of object storage. Objects combine metadata, data, permissions, and an identifier that are stored in a bucket.](../img/mapu_0904.png)

*Figure 9-4. An example of object storage. Objects combine metadata, data, permissions, and an identifier that are stored in a bucket.*



A bucket is a fully qualified namespace within the storage system. Object stores allow for a scalable solution for storing data redundancy. However, since object stores are not a filesystem, they are accessed through different APIs. Object stores have semantics such as the following (see [Figure 9-5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#object_storage_clients_perform_putcomma)):

- `GET`

  Retrieve an object by ID.

- `PUT`

  Place a name object with an ID in a bucket.

- `LIST`

  List all objects in a bucket.

- `DELETE`

  Delete an object by ID.

![Object storage clients perform PUT, GET, LIST, and DELETE on objects within a bucket.](../img/mapu_0905.png)

*Figure 9-5. Object storage clients perform PUT, GET, LIST, and DELETE on objects within a bucket.*



For most consumer use, object stores did not make sense. Consumers were used to filesystems that had some similarities to object stores but were limited in different ways that we won’t get into for this discussion. In the early 2000s, the internet began to attract different types of users and applications, and object stores became relevant. Today object stores are a critical component in public cloud usage. Object storage on the public cloud offers essentially unlimited storage for minimal costs (see [Table 9-2](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#summary_of_object_storage_offerings_on)).

| Public cloud provider                                        | Object storage offering | Cost per GB of storage[a](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#ch01fn27) |
| :----------------------------------------------------------- | :---------------------- | :----------------------------------------------------------- |
| Amazon Web Services                                          | S3                      | $0.023 per GB per month                                      |
| Google Cloud Platform                                        | Cloud Storage           | $0.020 per GB per month                                      |
| Azure Cloud                                                  | Blob Store              | $0.018 per GB per month                                      |
| IBM Cloud                                                    | Cloud Object Storage    | $0.0144 per GB per month                                     |
| Oracle Cloud                                                 | Oracle Object Storage   | $0.0255 per GB per month                                     |
| [a](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#ch01fn27-marker) Prices as of October 2021. There are several caveats to this pricing, including bulk discounts. |                         |                                                              |

![An example of how Pulsar tiered storage works. Most of the value can be found in the newest data as it is being written to Pulsar. As data gets older, moving it to object storage becomes more economical.](../img/mapu_0906.png)

*Figure 9-6. An example of how Pulsar tiered storage works. Most of the value can be found in the newest data as it is being written to Pulsar. As data gets older, moving it to object storage becomes more economical.*



In [Chapter 4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch04.html#pulsar_internals), we talked a bit about how BookKeeper can be used as an object store for some projects. BookKeeper ledgers have some similar properties to objects in an object store. [Table 9-3](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#a_comparison_of_objects_stored_in_objec) summarizes their similarities and differences.

|                  | Object store                         | BookKeeper ledger                                    |
| :--------------- | :----------------------------------- | :--------------------------------------------------- |
| **Data (bytes)** | Object stores store bytes of data.   | Ledgers store bytes of data.                         |
| **Metadata**     | Metadata is required for objects.    | Metadata is required for most use cases.             |
| **Permissions**  | Permissions are explicitly required. | Permissions are not required but can be implemented. |
| **Unique ID**    | Every object has a unique ID.        | Every ledger has a unique ID.                        |

With an immutable ledger, knowing when to move data is easy to determine, and since object storage can store arbitrarily large objects (and arbitrarily many), it’s also a good destination for our offloading. Now we can ask the question, what else do we get out of storing data in object storage?

# Use Cases

In addition to offloading data from a Pulsar cluster for cost reasons, there are other reasons for adopting tiered storage, including the following:

- Out-of-band replication
- Adopting an event sourcing model for your application
- Managing disaster recovery scenarios

## Replication

Replication is the process of keeping data synchronized. The canonical example of replication in software systems is replicating from a transaction database to a data warehouse. By definition, Pulsar’s tiered storage is a replication mechanism. It replicates data from a source to an offloaded storage engine (see [Figure 9-7](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#pulsar_tiered_storage_can_be_used_to_re)).

![Pulsar tiered storage can be used to replicate data from Pulsar to object storage, where it can be used in another system.](../img/mapu_0907.png)

*Figure 9-7. Pulsar tiered storage can be used to replicate data from Pulsar to object storage, where it can be used in another system.*



Pulsar’s tiered storage doesn’t replicate data in real time; it replicates it based on configuration. From an end-user perspective, we tell Pulsar at what age data is considered “old,” or how large the ledger should be before data should be offloaded to the object store. Thinking of tiered storage as a replication technique has some powerful implications. For example, we can build applications that utilize object storage which can scale to multiple clients, instead of using Pulsar for that purpose.

## CQRS

Most of the applications we use provide a way for us to create, edit, view, and delete things. For a concrete example, think about your favorite photo sharing application. You can add new pictures to the service, edit your pictures and descriptions, view your photos and photos of other users in the service, and delete your photos if you desire. Of course, each of these interactions requires some interaction with permanent storage in a database or an event streaming platform like Pulsar.

If we examine the needs of an application tasked with creating new entries, viewing entries, editing entries, and deleting them, there may be some differences in how we might handle these concerns from a development angle. When you open the photo application to share a photo, any perceived latency or failure to post the photo could be detrimental to your usage. Instead, the application developers might consider a fast and reliable path for saving new pictures, like a Pulsar client writing messages to the broker.

When viewing other users’ pictures, the experience may not need to be updated in real time and may be better read from an offloaded source, such as directly from the object storage. Utilizing one path for writing data and an utterly different path for reading data is the command query responsibility segregation (CQRS) pattern (see [Figure 9-8](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#in_this_example_of_the_command_query_re)).

![In this example of the Command Query Responsibility Segregation pattern, the application writes data to Topic A; as the data reaches a certain maturity, is it offloaded to object storage. From there, applications that are responsible for retrieving data can retrieve it from the object store rather than the messaging system.](../img/mapu_0908.png)

*Figure 9-8. In this example of the Command Query Responsibility Segregation pattern, the application writes data to Topic A; as the data reaches a certain maturity, is it offloaded to object storage. From there, applications that are responsible for retrieving data can retrieve it from the object store rather than the messaging system.*



In this example, we can keep all our new photos on a fresh stream in a topic. When photos get older, we can offload them to object storage. If a user wants to retrieve an old photo, it is retrieved from object storage.

## Disaster Recovery

Apache Pulsar clusters run on the internet and have many moving parts. While each component of Pulsar is measurably reliable independently, none of the components can withstand the complete disaster of a public cloud provider’s infrastructure or disasters that may occur in an on-premises environment. One defense against this is to create more redundancy in the system. Tiered storage provides a way to offload data from a reliable system (BookKeeper) to an even more reliable system (an object store); see [Figure 9-9](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#in_disaster_scenarios_where_pulsar_data).

![In disaster scenarios where Pulsar data is destroyed, having offloaded data can help speed system recovery.](../img/mapu_0909.png)

*Figure 9-9. In disaster scenarios where Pulsar data is destroyed, having offloaded data can help speed system recovery.*



As an example, let’s consider Amazon S3, perhaps the most popular object storage platform in the world. [Amazon states](https://oreil.ly/qZkyJ) that S3 has 11 nines (99.999999999) of reliability for any object stored on S3. We’ve talked about BookKeeper and the reliability of the data stored there. As operators of BookKeeper, we increase our quorum sizes to deal with potential disasters, or even come up with a multiple-availability-zone solution. That said, having an additional level of redundancy offloaded from ledgers only helps the recovery scenario.

# Offloading Data

Let’s take a look at how offloading data from Pulsar bookies to object storage works. In the following subsections, we’ll discuss the schema and columnar offloader in Pulsar and how we can configure our namespace for offloading.

## Pulsar Offloaders

Apache Pulsar uses Apache jClouds for its implementation of tiered storage for object storage. Apache jClouds is an open source library that brings cloud-native functionality to Java. The concept of offloaders is simply a set of configurations and rules for how to offload and where to offload the data (see [Figure 9-10](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#the_management_lifecycle_of_pulsar_tier)).

![The management lifecycle of Pulsar tiered storage. The configurations to external systems and authentication, scheduling, and unloading mechanisms are stored as metadata and executed when required.](../img/mapu_0910.png)

*Figure 9-10. The management lifecycle of Pulsar tiered storage. The configurations to external systems and authentication, scheduling, and unloading mechanisms are stored as metadata and executed when required.*



For object storage, Pulsar supports Amazon S3 as well as Google Cloud Storage. We’ll cover each in this section. For each object store we won’t cover creating a bucket; good tutorials for how to do that are available online, and I will share some of the tutorials I find helpful in the footnotes.

### Amazon S3

To set up object storage on Amazon S3, follow these steps:

- Create an [S3 bucket](https://oreil.ly/QFJPr).

- Ensure that your AWS credentials are accessible via the CLI.

- Edit the *conf/pulsar_env.sh* file to include AWS credentials and role ARNs, as listed in the following code snippet:

  ```
  export AWS_ACCESS_KEY_ID=<id> // AN AWS ACCESS Key
  export AWS_SECRET_ACCESS_KEY=<Key> // AN AWS ACCESS KEY SECRET
  PULSAR_EXTRA_OPTS="${PULSAR_EXTRA_OPTS} ${PULSAR_MEM} ${PULSAR_GC} 
  -Daws.accessKeyId=<id> -Daws.secretKey=<key> 
  -Dio.netty.leakDetectionLevel=disabled
  -Dio.netty.recycler.maxCapacity.default=1000 
  -Dio.netty.recycler.linkCapacity=1024"
  s3ManagedLedgerOffloadRole=<aws role arn for your role> // The Role ARN
  s3ManagedLedgerOffloadRoleSessionName=session-name // A Session name
  ```

- Edit your *broker.conf* file using the values in [Table 9-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#broker_values_for_the_managed_ledger_of).

- Start the broker.

| Configuration value                           | Description                                                  | Required                | Default value/examples |
| :-------------------------------------------- | :----------------------------------------------------------- | :---------------------- | :--------------------- |
| `managedLedgerOffloadDriver`                  | Offloader driver name, which is case insensitive             | Yes                     | `aws-s3`               |
| `offloadersDirectory`                         | Offloader directory                                          | Yes                     | `offloaders`           |
| `s3ManagedLedgerOffloadBucket`                | Bucket                                                       | Yes                     | `pulsar-topic-offload` |
| `s3ManagedLedgerOffloadRegion`                | Bucket region                                                | No                      | `eu–west-3`            |
| `s3ManagedLedgerOffloadServiceEndpoint`       | `s3ManagedLedgerOffloadServiceEndpoint=https://s3.YOUR_REGION.amazonaws.com` | Only if `Region` is set | NA                     |
| `s3ManagedLedgerOffloadReadBufferSizeInBytes` | Size of read block                                           | No                      | 1 MB                   |
| `s3ManagedLedgerOffloadMaxBlockSizeInBytes`   | Size of write block                                          | No                      | 64 MB                  |
| `managedLedgerMinLedgerRollover⁠T⁠i⁠m⁠eM⁠i⁠n⁠u⁠t⁠e⁠s`   | Minimum time in minutes between ledger rollover for a topic  | No                      | 2                      |
| `managedLedgerMaxEntriesPerLedger`            | Maximum number of entries to append to a ledger before triggering a rollover | No                      | 500                    |
| `s3ManagedLedgerOffloadReadBufferSizeInBytes` | Block size for each individual read when reading back data from AWS S3 | No                      | 1 MB                   |
| `s3ManagedLedgerOffloadMaxBlockSizeInBytes`   | Maximum size of a “part” sent during a multipart upload to AWS S3 (cannot be smaller than 5 MB) | No                      | 64 MB                  |

In the preceding steps, the configuration is set for all topics across all namespaces. You can override this and set an offload policy with the Pulsar CLI:

```
bin/pulsar-admin namespaces set-offload-threshold --size 15M tenant-a/namespace-a
```

You can also do this with the Pulsar Admin API (see [Table 9-4](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#broker_values_for_the_managed_ledger_of) for the appropriate values):

```
POST /https://pulsar.apache.org/admin/v2/namespaces/
	{tenant}/{namespace}/offloadPolicies
Body 
{
  "offloadersDirectory": "string",
  "managedLedgerOffloadDriver": "string",
  "managedLedgerOffloadMaxThreads": 0,
  "managedLedgerOffloadPrefetchRounds": 0,
  "managedLedgerOffloadThresholdInBytes": 0,
  "managedLedgerOffloadDeletionLagInMillis": 0,
  "managedLedgerOffloadedReadPriority": "BOOKKEEPER_FIRST",
  "s3ManagedLedgerOffloadRegion": "string",
  "s3ManagedLedgerOffloadBucket": "string",
  "s3ManagedLedgerOffloadServiceEndpoint": "string",
  "s3ManagedLedgerOffloadMaxBlockSizeInBytes": 0,
  "s3ManagedLedgerOffloadReadBufferSizeInBytes": 0,
  "s3ManagedLedgerOffloadCredentialId": "string",
  "s3ManagedLedgerOffloadCredentialSecret": "string",
  "s3ManagedLedgerOffloadRole": "string",
  "s3ManagedLedgerOffloadRoleSessionName": "string",
  "managedLedgerOffloadBucket": "string",
  "managedLedgerOffloadRegion": "string",
  "managedLedgerOffloadServiceEndpoint": "string",
  "managedLedgerOffloadMaxBlockSizeInBytes": 0,
  "managedLedgerOffloadReadBufferSizeInBytes": 0,
  "s3Driver": true
}
```

You can also run a manually triggered offload with the following Pulsar CLI command:

```
bin/pulsar-admin topics offload --size-threshold 15M tenant-a/my-namespace/topicABC
```

Here’s a complete example of settings within a *broker.conf* file for AWS:

```
managedLedgerOffloadDriver=aws-s3
offloadersDirectory=offloaders
s3ManagedLedgerOffloadRegion=us-west-2
s3ManagedLedgerOffloadServiceEndpoint=https://s3.us-west-2.amazonaws.com
```

### Google Cloud Storage

To set up object storage on Google Cloud Storage, follow these steps:

- Create a [bucket](https://oreil.ly/W6T2o) in Google Cloud Storage.
- Create a [service account](https://oreil.ly/aMz7o).
- Edit your *broker.conf* file with the values shown in [Table 9-5](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#values_for_broker_configuration_with_go).
- Start the broker.

A service account is a way to enable API access to Google Cloud products. When access is enabled, the user can get an authentication key from Google to allow other products to use the Google Cloud service via a Google Cloud API. In our case, we want to create a service account for OAuth access to Google Cloud. I’ve linked to some [documentation from Google](https://oreil.ly/yKsUf) on how to set this up.

We need to place the keyfile somewhere on the broker in order to allow Pulsar to authenticate with Google Cloud. A keyfile looks something like this:

```
{
  "type": "service_account",
  "project_id": "project-id",
  "private_key_id": "key-id",
  "private_key":
    "-----BEGIN PRIVATE KEY-----\nprivate-key\n-----END PRIVATE KEY-----\n",
  "client_email": "service-account-email",
  "client_id": "client-id",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url":
    https://www.googleapis.com/robot/v1/metadata/x509/service-account-email
}
```

Once we have the keyfile and place it in our Pulsar broker, we edit our *broker.conf* to include the location to that keyfile:

```
gcsManagedLedgerOffloadServiceAccountKeyFile="data/myproj.json"
```

| Configuration value                            | Description                                                  | Required | Default/Example                                       |
| :--------------------------------------------- | :----------------------------------------------------------- | :------- | :---------------------------------------------------- |
| `managedLedgerOffloadDriver`                   | Offloader driver name                                        | Yes      | `google-cloud-storage`                                |
| `offloadersDirectory`                          | Offloader directory                                          | Yes      | `offloaders`                                          |
| `gcsManagedLedgerOffloadBucket`                | Bucket                                                       | Yes      | `pulsar-topic-offload`                                |
| `gcsManagedLedgerOffloadRegion`                | Bucket region                                                | Yes      | `us-a–2`                                              |
| `gcsManagedLedgerOffloadServiceAccountKeyFile` | Authentication                                               | Yes      | `Users/user-name/Downloads/project-804d5e6a6f33.json` |
| `gcsManagedLedgerOffloadReadB⁠u⁠f⁠f⁠e⁠rS⁠i⁠z⁠e⁠InBytes` | Size of block read                                           | No       | NA                                                    |
| `gcsManagedLedgerOffloadMaxBlockSizeInBytes`   | Size of block write                                          | No       | NA                                                    |
| `managedLedgerMinLedgerRolloverT⁠i⁠m⁠eM⁠i⁠n⁠u⁠t⁠e⁠s`    | Minimum time between ledger rollover for a topic             | No       | NA                                                    |
| `managedLedgerMaxEntriesPerLedger`             | The maximum number of entries to append to a ledger before triggering a rollover | No       | 5,000                                                 |
| `gcsManagedLedgerOffloadReadB⁠u⁠f⁠f⁠e⁠rS⁠i⁠z⁠eInBytes` | Block size for each individual read when reading back data from GCS | No       | 1 MB                                                  |
| `gcsManagedLedgerOffloadMaxBlockSizeInBytes`   | Maximum size of a “part” sent during a multipart upload to GCS | No       | 64 MB                                                 |

In the preceding steps, the configuration is set for all topics across all namespaces. You can override this and set an offload policy with the Pulsar CLI:

```
bin/pulsar-admin namespaces set-offload-threshold --size 15M tenant-a/namespace-a
```

Or the Pulsar Admin API:

```
POST /https://pulsar.apache.org/admin/v2/namespaces/{tenant}/{namespace}/offloadPolicies
Body 
{
  "offloadersDirectory": "string",
  "managedLedgerOffloadDriver": "string",
  "managedLedgerOffloadMaxThreads": 0,
  "managedLedgerOffloadPrefetchRounds": 0,
  "managedLedgerOffloadThresholdInBytes": 0,
  "managedLedgerOffloadDeletionLagInMillis": 0,
  "managedLedgerOffloadedReadPriority": "BOOKKEEPER_FIRST",
  "gcsManagedLedgerOffloadRegion": "string",
  "gcsManagedLedgerOffloadBucket": "string",
  "gcsManagedLedgerOffloadMaxBlockSizeInBytes": 0,
  "gcsManagedLedgerOffloadReadBufferSizeInBytes": 0,
  "gcsManagedLedgerOffloadServiceAccountKeyFile": "string",
  "managedLedgerOffloadBucket": "string",
  "managedLedgerOffloadRegion": "string",
  "managedLedgerOffloadServiceEndpoint": "string",
  "managedLedgerOffloadMaxBlockSizeInBytes": 0,
  "managedLedgerOffloadReadBufferSizeInBytes": 0,
  "gcsDriver": true
}
```

You can also run a manually triggered offload with the following Pulsar CLI command:

```
bin/pulsar-admin topics offload --size-threshold 15M tenant-a/my-namespace/topicABC
```

Here is a complete example of settings within a *broker.conf* file for GCP:

```
managedLedgerOffloadDriver=google-cloud-storage
offloadersDirectory=offloaders
gcsManagedLedgerOffloadBucket=my-offload-bucket
gcsManagedLedgerOffloadServiceAccountKeyFile=/my/file/path/key.json
```

Earlier in this chapter we talked about other cloud storage platforms such as Azure Blob Storage and IBM Object Storage. Pulsar can support these platforms in two ways. If they have an S3-compatible API, you can use the same S3 configurations we discussed. If not, it’s a matter of allowing Apache jClouds to support them and then making some changes to Pulsar to allow them as well.

# Retrieving Offloaded Data

Now that you know how to move data off the Apache BookKeeper tier and into a more cost-efficient storage mechanism, and the advantages of doing so, let’s focus on how to get offloaded data back into Pulsar.

Offloaded data are objects written to the file or object storage. There are three ways to interact with offloaded data:

- Directly through the object store
- Repopulating the topic with offloaded data
- Utilizing the Pulsar client

## Interacting with Object Store Data

Earlier in this chapter, we talked about the CQRS pattern and why, for some applications utilizing this pattern, we might want to store data using Pulsar clients and retrieve it using an object store (see [Figure 9-11](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#an_example_of_how_a_sql_engine_could_be)).

![An example of how a SQL engine could be used to retrieve BookKeeper segments from object storage. The SQL engine would need to be able to parse BookKeeper files.](../img/mapu_0911.png)

*Figure 9-11. An example of how a SQL engine could be used to retrieve BookKeeper segments from object storage. The SQL engine would need to be able to parse BookKeeper files.*



It’s worth reiterating that this approach has some limitations. First, data that is offloaded from BookKeeper to object storage is not necessarily human readable or in common machine-readable formats like CSV, Parquet, or JSON. It’s possible to build a parser to allow for a SQL interface over the journal files, but one does not exist off the shelf.

## Repopulating Topics

Pulsar clients can automatically read offloaded data (we will discuss this in the next subsection). However, there are some reasons why you may not want to enable the ability to read from offloaded data automatically and may want fine-grained control over which data a consumer can retrieve. In these cases, instead of automatically reading from offloaded topics, we may load data on a new topic and point the consumer to that topic for older data (see [Figure 9-12](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#a_producer_can_retrieve_data_from_bookk)).

![A producer can retrieve data from BookKeeper and create a new topic for consumers with special permission to consume the data.](../img/mapu_0912.png)

*Figure 9-12. A producer can retrieve data from BookKeeper and create a new topic for consumers with special permission to consume the data.*



This pattern may seem cumbersome and error prone on the surface, and while it can be, there are some excellent reasons to adopt it. For example, some companies, such as those in the financial and medical arenas, must keep records for up to 10 years. As a result, there is some risk to allow ingestion of old records, and an organization may decide to disable retrieval of old records. A procedure that restores retrieved data into a new topic for audit could be safer.

For this use case, you can use an object storage client library to read data from the bucket, parse the ledger into messages, and then write the messages out to Pulsar. This is a nontrivial amount of work, but it is doable if you have a specific use case. An alternative would be to build a Pulsar IO connector that had an object store source connector, and then parse the ledgers stored in S3 and write them to a Pulsar topic.

## Utilizing Pulsar Client

Pulsar topics and the location where cursor data is tracked are stored in the managed ledger. The managed ledger tells the Pulsar broker not only which ledger to retrieve for a consumer, but also where that ledger is stored (see [Figure 9-13](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#pulsar_brokers_use_the_managed_ledger_c)). As a consumer, it’s not necessary to worry about where the data is stored; the Pulsar broker will retrieve the data as needed from wherever it may be (see [Figure 9-14](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/ch09.html#when_consuming_datacomma_the_pulsar_bro)). This is the easiest way to interact with offloaded data in Pulsar, and it provides a transparent way for the consumer to get the data it needs.

![Pulsar brokers use the managed ledger cache to work as a tail log. BookKeeper is where data is stored when it’s required for the Pulsar cluster. Object storage (tiered storage) is where any long-lived data should go.](../img/mapu_0913.png)

*Figure 9-13. Pulsar brokers use the managed ledger cache to work as a tail log. BookKeeper is where data is stored when it’s required for the Pulsar cluster. Object storage (tiered storage) is where any long-lived data should go.*



![When consuming data, the Pulsar broker interacts with ZooKeeper and the managed ledger tells the broker where data is stored. It can retrieve data from BookKeeper or from object storage. Either way is transparent to the consumer.](../img/mapu_0914.png)

*Figure 9-14. When consuming data, the Pulsar broker interacts with ZooKeeper and the managed ledger tells the broker where data is stored. It can retrieve data from BookKeeper or from object storage. Either way is transparent to the consumer.*



For an implementation, suppose the earliest offsets in a Pulsar topic are offloaded and newer data is still stored in BookKeeper. We simply need to start our consuming at the earliest offset, and the Pulsar broker will handle retrieving data from the offloaded space:

```
import org.apache.pulsar.client.api.MessageId;
import org.apache.pulsar.client.api.Reader;

Reader<byte[]> reader = pulsarClient.newReader()
    .topic("read-from–topic")
    .startMessageId(MessageId.earliest) // get data at earliest offset
    .create();

while (true) {
    Message message = reader.readNext();

    // Get messages after this point
}
```

# Summary

In this chapter you learned about Apache Pulsar tiered storage. Specifically, you learned how it’s more than a cost-savings mechanism; it’s also a mechanism to enable new types of interactions with Apache Pulsar. You also learned that the way Pulsar stores data in BookKeeper creates a sensible link between an object store and Pulsar. Now we’ll move on to learning how to query Pulsar topics with the SQL language.

