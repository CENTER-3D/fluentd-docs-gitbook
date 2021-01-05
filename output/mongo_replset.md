# mongo\_replset

![](../.gitbook/assets/mongo_replset.png)

The `out_mongo_replset` Buffered Output plugin writes records into [MongoDB](http://mongodb.org/), the emerging document-oriented database system. This plugin is for users using ReplicaSet. If you are not using ReplicaSet, please see the [out\_mongo](mongo.md) article instead.

## Why Fluentd with MongoDB?

Fluentd enables your apps to insert records to MongoDB asynchronously with batch-insertion, unlike direct insertion of records from your apps. This has the following advantages:

1. less impact on application performance
2. higher MongoDB insertion throughput while maintaining JSON record

   structure

## Install

`out_mongo_replset` is included in td-agent by default. Fluentd gem users will need to install the fluent-plugin-mongo gem using the following command.

```text
$ fluent-gem install fluent-plugin-mongo
```

## Example Configuration

```text
# Single MongoDB
<match mongo.**>
  @type mongo_replset
  database fluentd
  collection test
  nodes localhost:27017,localhost:27018,localhost:27019

  # flush
  flush_interval 10s
</match>
```

Please see the [Store Apache Logs into MongoDB](../articles/apache-to-mongodb.md) article for real-world use cases.

Please see the [Config File](../configuration/config-file.md) article for the basic structure and syntax of the configuration file.

## Parameters

### type \(required\)

The value must be `mongo`.

### nodes \(required\)

The comma separated node strings \(e.g. host1:27017,host2:27017,host3:27017\).

### database \(required\)

The database name.

### collection \(required if not tag\_mapped\)

The collection name.

### capped

This option enables capped collection. This is always recommended because MongoDB is not suited to storing large amounts of historical data.

### capped\_size

Sets the capped collection size.

### user

The username to use for authentication.

### password

The password to use for authentication.

### tag\_mapped

This option will allow out\_mongo to use Fluentd's tag to determine the destination collection.

For example, if you generate records with tags 'mongo.foo', the records will be inserted into the `foo` collection within the `fluentd` database.

```text
<match mongo.*>
  @type mongo_replset
  database fluentd
  nodes localhost:27017,localhost:27018,localhost:27019

  # Set 'tag_mapped' if you want to use tag mapped mode.
  tag_mapped

  # If the tag is "mongo.foo", then the prefix "mongo." is removed.
  # The inserted collection name is "foo".
  remove_tag_prefix mongo.

  # This configuration is used if the tag is not found. The default is 'untagged'.
  collection misc
</match>
```

### name

The ReplicaSet name.

### read

The ReplicaSet read preference \(e.g. secondary, etc\).

### refresh\_mode

The ReplicaSet refresh mode \(e.g. sync, etc\).

### refresh\_interval

The ReplicaSet refresh interval.

### num\_retries

The ReplicaSet failover threshold. The default threshold is 60. If the retry count reaches this threshold, the plugin raises an exception.

## Buffered Output Parameters

For advanced usage, you can tune Fluentd's internal buffering mechanism with these parameters.

### buffer\_type

The buffer type is `memory` by default \([buf\_memory](../buffer/memory.md)\) for the ease of testing, however `file` \([buf\_file](../buffer/file.md)\) buffer type is always recommended for the production deployments. If you use `file` buffer type, `buffer_path` parameter is required.

### buffer\_queue\_limit, buffer\_chunk\_limit

The length of the chunk queue and the size of each chunk, respectively. Please see the [Buffer Plugin Overview](../buffer/) article for the basic buffer structure. The default values are 64 and 8m, respectively. The suffixes "k" \(KB\), "m" \(MB\), and "g" \(GB\) can be used for buffer\_chunk\_limit.

### flush\_interval

The interval between data flushes. The default is 60s. The suffixes "s" \(seconds\), "m" \(minutes\), and "h" \(hours\) can be used.

### flush\_at\_shutdown

If set to true, Fluentd waits for the buffer to flush at shutdown. By default, it is set to true for Memory Buffer and false for File Buffer.

### retry\_wait, max\_retry\_wait

The initial and maximum intervals between write retries. The default values are 1.0 seconds and unset \(no limit\). The interval doubles \(with +/-12.5% randomness\) every retry until `max_retry_wait` is reached.

Since td-agent will retry 17 times before giving up by default \(see the `retry_limit` parameter for details\), the sleep interval can be up to approximately 131072 seconds \(roughly 36 hours\) in the default configurations.

### retry\_limit, disable\_retry\_limit

The limit on the number of retries before buffered data is discarded, and an option to disable that limit \(if true, the value of `retry_limit` is ignored and there is no limit\). The default values are 17 and false \(not disabled\). If the limit is reached, buffered data is discarded and the retry interval is reset to its initial value \(`retry_wait`\).

### num\_threads

The number of threads to flush the buffer. This option can be used to parallelize writes into the output\(s\) designated by the output plugin. Increasing the number of threads improves the flush throughput to hide write / network latency. The default is 1.

### slow\_flush\_log\_threshold

The threshold for checking chunk flush performance. The default value is `20.0` seconds. Note that parameter type is `float`, not `time`.

If chunk flush takes longer time than this threshold, fluentd logs warning message like below:

```text
2016-12-19 12:00:00 +0000 [warn]: buffer flush took longer time than slow_flush_log_threshold: elapsed_time = 15.0031226690043695 slow_flush_log_threshold=10.0 plugin_id="foo"
```

#### log\_level option

The `log_level` option allows the user to set different levels of logging for each plugin. The supported log levels are: `fatal`, `error`, `warn`, `info`, `debug`, and `trace`.

Please see the [logging article](../deployment/logging.md) for further details.

## Further Readings

* [fluent-plugin-webhdfs mongo](https://github.com/fluent/fluent-plugin-mongo)

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open). [Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation \(CNCF\)](https://cncf.io/). All components are available under the Apache 2 License.

