# Connections

A connection is a configuration for syncing data between a source and a destination. To setup a connection, a user must configure things such as:

* Sync schedule: when to trigger a sync of the data.
* Namespace and stream names in the destination: where the data will end up being written.
* A catalog selection: which [streams and fields](../catalog.md) to replicate from the source.
* Sync mode: how each stream should be replicated (read & write behaviors).

## Sync schedules

Syncs will be triggered by either:

* A manual request \(i.e: clicking the "Sync Now" button in the UI\)
* A schedule

When a scheduled connection is first created, a sync is executed as soon as possible. After that, a sync is run once the time since the last sync \(whether it was triggered manually or due to a schedule\) has exceeded the schedule interval. For example, consider the following illustrative scenario:

* **October 1st, 2pm**, a user sets up a connection to sync data every 24 hours.
* **October 1st, 2:01pm**: sync job runs
* **October 2nd, 2:01pm:** 24 hours have passed since the last sync, so a sync is triggered.
* **October 2nd, 5pm**: The user manually triggers a sync from the UI
* **October 3rd, 2:01pm:** since the last sync was less than 24 hours ago, no sync is run
* **October 3rd, 5:01pm:** It has been more than 24 hours since the last sync, so a sync is run

## Destination namespace

The location of where a connection replication will store data is referenced as the destination namespace. The destination connectors should create and write records (for both raw and normalized tables) in the specified namespace which should be configurable in the UI thanks to the Namespace Configuration field (or NamespaceDefinition in the API).

The available options are:

### - Mirror source structure

Some sources (such as databases based on JDBC for example) are providing namespace informations from which a stream has been extracted from. Whenever a source is able to fill this field in the catalog.json file, the destination will try to reproduce exactly the same namespace when this configuraton is set.
For sources or streams where the source namespace is not known, the behavior will fall back to the "Destination Connector settings".

### - Destination connector settings

All stream will be replicated and store in the default namespace defined on the destination settings page.
In the destinations, namespace refers to:

| Destination Connector | Namespace setting |
| :--- | :--- |
| BigQuery | my_schema |
| MSSQL | schema |
| MySql | database |
| Postgres | schema |
| Oracle | schema |
| Redshift | schema |
| Snowflake | schema |

### - Custom format

When replicating multiple sources into the same destination, conflicts on tables being overwritten by syncs can occur.

For example, a Github source can be replicated into a "github" schema.
But if we have multiple connections to different GitHub repositories (similar in multi-tenant scenarios):

- we'd probably wish to keep the same table names (to keep consistent queries downstream)
- but store them in different namespaces (to avoid mixing data from different "tenants")

To solve this, we can either:

- use a specific namespace for each connection, thus this option of custom format.
- or, use prefix to stream names as described below.

Note that we can use a template format string using variables that will be resolved during replication as follow:

- `${SOURCE_NAMESPACE}`: will be replaced by the namespace provided by the source if available

### Examples

Here are examples of replication configurations between a Postgres Source and Snowflake Destination (with settings of schema = "my_schema"):

| Namespace Configuration | Source Namespace | Source Table Name | Destination Namespace | Destination Table Name |
| :--- | :--- | :--- | :--- | :--- |
| Mirror source structure | public | my_table | public | my_table |
| Mirror source structure | | my_table | my_schema | my_table |
| Destination connector settings | public | my_table | my_schema | my_table |
| Destination connector settings | | my_table | my_schema | my_table |
| Custom format = "custom" | public | my_table | custom | my_table |
| Custom format = "${SOURCE_NAMESPACE}" | public | my_table | public | my_table |
| Custom format = "my_${SOURCE_NAMESPACE}_schema" | public | my_table | my_public_schema | my_table |
| Custom format = "   " | public | my_table | my_schema | my_table |

## Destination stream name

### Prefix stream name

Stream names refers to table names in a typical RDBMS. But it can also be the name of an API endpoint etc. Similarly to the namespace, stream names can be configured to diverge from their names in the source with a "prefix" field. The prefix is prepended to the source stream name in the destination.

## Stream by stream customization

All the customization of namespace and stream names described above will be equally applied to all streams selected for replication in a catalog per connection. If you need more granular customization, stream by stream, for example, or with different logic rules, then you could follow the tutorial on [customizing transformations with dbt](../../tutorials/transformation-and-normalization/transformations-with-dbt.md).

## Sync modes

A sync mode governs how Airbyte reads from a source and writes to a destination. Airbyte provides different sync modes to account for various use cases. To minimize confusion, a mode's behavior is reflected in its name. The easiest way to understand Airbyte's sync modes is to understand how the modes are named.

1.  The first part of the name denotes how the source connector reads data from the source:

* Incremental: Read records added to the source since the last sync job. (The first sync using Incremental is equivalent to a Full Refresh)
  * Method 1: Using a cursor. Generally supported by all sources.
  * Method 2: Using change data capture. Only supported by some sources. See [CDC](../cdc.md) for more info.
* Full Refresh: Read everything in the source.

2. The second part of the sync mode name denotes how the destination connector writes data. This is not affected by how the source connector produced the data:

* Overwrite: Overwrite by first deleting existing data in the destination.
* Append: Write by adding data to existing tables in the destination.
* Deduped History: Write by first adding data to existing tables in the destination to keep a history of changes. The final table is produced by de-duplicating the intermediate ones using a primary key.

A sync mode is therefore, a combination of a source and destination mode together. The UI exposes the following options, whenever both source and destination connectors are capable to support it for the corresponding stream:
* [Full Refresh Overwrite](full-refresh-overwrite.md): Sync the whole stream and replace data in destination by overwriting it.
* [Full Refresh Append](full-refresh-append.md): Sync the whole stream and append data in destination.
* [Incremental Append](incremental-append.md): Sync new records from stream and append data in destination.
* [Incremental Deduped History](incremental-deduped-history.md): Sync new records from stream and append data in destination, also provides a de-duplicated view mirroring the state of the stream in the source.
