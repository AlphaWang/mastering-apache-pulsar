# Appendix B. Pulsar Admin CLI

Throughout the book, I’ve used the Pulsar Admin CLI to perform many tasks. I shared snippets of the commands but didn’t really dive into the bigger picture of the CLI. That was partly due to it not fitting into the flow of the book, but also due to knowing I would reserve this section of the book to talk about it. This appendix will quickly cover the Pulsar Admin CLI and go through all of the resources available on the CLI. It’s here to serve as a reference (see [Table B-1](https://learning.oreilly.com/library/view/mastering-apache-pulsar/9781492084891/app02.html#available_methods_on_the_pulsar_admin_c)).

For an exhaustive list of what is available in the Pulsar Admin CLI, you can consult the [official CLI reference](https://oreil.ly/4GOt9).

| Resource              | Purpose                                                      |
| :-------------------- | :----------------------------------------------------------- |
| `Brokers`             | List, delete, and edit brokers and metadata about brokers    |
| `Broker-stats`        | Retrieve broker statistics                                   |
| `Clusters`            | List, delete, and edit clusters and metadata about clusters  |
| `Functions`           | List, create, delete, and edit Pulsar Functions and metadata |
| `Namespaces`          | List, create, delete, and edit Pulsar namespaces             |
| `ns-isolation-policy` | Manage Pulsar namespace isolation policies                   |
| `Sources`             | List, create, delete, and edit Pulsar IO sources             |
| `Sinks`               | List, create, delete, and edit Pulsar IO sinks               |
| `Topics`              | List, create, delete, and edit Pulsar topics                 |
| `Tenants`             | List, create, delete, and edit Pulsar tenants                |
| `Resource-quotas`     | Manage Pulsar resource-quotas                                |
| `Schemas`             | List and manage schemas used in the schema registry          |
| `Packages`            | Manage packaged versions used throughout Pulsar              |

# CLI API

The CLI API is consistent across all the resources in the Pulsar Admin API. This makes it easy to remember and also easy to reason about if you don’t know the exact command for the CLI. The format is:

```
$ pulsar-admin <resource> <subcommand>
```

For each resource, you can find hints by asking for help on that specific resource:

```
$ pulsar-admin <resource> -h
```

Let’s look at a few examples of this in action.

# Examples

In this section, I’ll share a few commands for performing common tasks with the Pulsar CLI.

## Creating a Partitioned Topic

This command creates a partitioned topic with 100 partitions:

```
$ pulsar-admin topics create-partitioned-topic -p 100
```

## Creating a Pulsar IO Source

This command creates a Pulsar IO source. Options include picking a source and sharing configuration files with Pulsar:

```
$ pulsar-admin sources create options
```

## Creating a Pulsar IO Sink

This command creates a Pulsar IO sink. Options include picking a sink and sharing configuration files with Pulsar:

```
$ pulsar-admin sinks create options
```

## Uploading a Schema

This command uploads a schema to the schema registry with the schema definition provided by the file:

```
$ pulsar-admin schemas upload -f myschema.json
```

## Deleting a Schema

This command deletes a specific schema:

```
$ pulsar-admin schemas delete options
```

## Creating a Namespace

This command creates a namespace with specified options. Options include clusters in which to enable the new namespace:

```
$ pulsar-admin namespaces create options
```

## Deleting a Namespace

This command deletes a namespace with options:

```
$ pulsar-admin namespaces delete options
```

# Summary

As you can see, the Pulsar Admin CLI is a powerful tool. Similar to the REST API, we can perform all the necessary administrative tasks with the Pulsar Admin CLI. It’s important to remember that access to full CLI capabilities should be restricted to staff who have experience, and operations like deleting should be carefully considered.