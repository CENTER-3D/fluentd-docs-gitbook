# unix

![](../.gitbook/assets/unix.png)

The `in_unix` Input plugin enables Fluentd to retrieve records from the Unix Domain Socket. The wire protocol is the same as [in\_forward](forward.md), but the transport layer is different.

## Example Configuration

`in_unix` is included in Fluentd's core. No additional installation process is required.

```text
<source>
  @type unix
  path /path/to/socket.sock
</source>
```

Please see the [Config File](../configuration/config-file.md) article for the basic structure and syntax of the configuration file.

## Parameters

### type \(required\)

The value must be `unix`.

### path \(required\)

The path to your Unix Domain Socket.

### backlog

The backlog of Unix Domain Socket. The default is `1024`.

#### log\_level option

The `log_level` option allows the user to set different levels of logging for each plugin. The supported log levels are: `fatal`, `error`, `warn`, `info`, `debug`, and `trace`.

Please see the [logging article](../deployment/logging.md) for further details.

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open). [Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation \(CNCF\)](https://cncf.io/). All components are available under the Apache 2 License.

