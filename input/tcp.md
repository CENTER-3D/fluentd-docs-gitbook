# tcp

![](../.gitbook/assets/tcp.png)

The `in_tcp` Input plugin enables Fluentd to accept TCP payload.

Don't use this plugin for receiving logs from client libraries. Use `in_forward` for such cases.

## Example Configuration

`in_tcp` is included in Fluentd's core. No additional installation process is required.

```text
<source>
  @type tcp
  tag tcp.events # required
  format /^(?<field1>\d+):(?<field2>\w+)$/ # required
  port 20001 # optional. 5170 by default
  bind 0.0.0.0 # optional. 0.0.0.0 by default
  delimiter \n # optional. \n (newline) by default
</source>
```

Please see the [Config File](../configuration/config-file.md) article for the basic structure and syntax of the configuration file.

We\'ve observed the drastic performance improvements on Linux, with proper kernel parameter settings. If you have high-volume TCP traffic, please make sure to follow the instruction described at [Before Installing Fluentd](../articles/before-install.md).

## Parameters

### type \(required\)

The value must be `tcp`.

### tag \(required\)

tag of output events.

### port

The port to listen to. Default Value = 5170

### bind

The bind address to listen to. Default Value = 0.0.0.0

### delimiter

The payload is read up to this character. By default, it is "\n".

### source\_hostname\_key

The field name of the client's hostname. If set the value, the client's hostname will be set to its key. The default is nil \(no adding hostname\).

If you set following configuration:

```text
source_hostname_key client_host
```

then the client's hostname is set to `client_host` field.

```text
{
    ...
    "foo": "bar",
    "client_host": "client.hostname.org"
}
```

### format \(required\)

The format of the TCP payload. Here is the example by regular expression.

```text
format /^(?<field1>\d+):(?<field2>\w+)$/
```

If you execute following command:

```text
$ echo '123456:awesome' | netcat 0.0.0.0 5170
```

then got parsed result like below:

```text
{"field1":"123456","field2":"awesome}
```

`in_tcp` uses parser plugin to parse the payload. See [parser article](../parser/) for more detail.

#### log\_level option

The `log_level` option allows the user to set different levels of logging for each plugin. The supported log levels are: `fatal`, `error`, `warn`, `info`, `debug`, and `trace`.

Please see the [logging article](../deployment/logging.md) for further details.

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open). [Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation \(CNCF\)](https://cncf.io/). All components are available under the Apache 2 License.

