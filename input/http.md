# http

![](../.gitbook/assets/http.png)

The `in_http` Input plugin enables Fluentd to retrieve records from HTTP POST. The URL path becomes the `tag` of the Fluentd event log and the POSTed body element becomes the record itself.

## Example Configuration

`in_http` is included in Fluentd's core. No additional installation process is required.

```text
<source>
  @type http
  port 8888
  bind 0.0.0.0
  body_size_limit 32m
  keepalive_timeout 10s
</source>
```

Please see the [Config File](../configuration/config-file.md) article for the basic structure and syntax of the configuration file.

### Example Usage

The example below posts a record using the `curl` command.

```text
$ curl -X POST -d 'json={"action":"login","user":2}'
  http://localhost:8888/test.tag.here;
```

## Parameters

### type \(required\)

The value must be `http`.

### port

The port to listen to. Default Value = 9880

### bind

The bind address to listen to. Default Value = 0.0.0.0 \(all addresses\)

### body\_size\_limit

The size limit of the POSTed element. Default Value = 32MB

### keepalive\_timeout

The timeout limit for keeping the connection alive. Default Value = 10 seconds

### add\_http\_headers

Add `HTTP_` prefix headers to the record. The default is `false`

### add\_remote\_addr

Add `REMOTE_ADDR` field to the record. The value of `REMOTE_ADDR` is the client's address. The default is `false`

If your system set multiple `X-Forwarded-For` headers in the request, `in_http` uses first one. For example:

```text
X-Forwarded-For: host1, host2
X-Forwarded-For: host3
```

If send above multiple headers, `REMOTE_ADDR` value is `host1`.

### cors\_allow\_origins

White list domains for CORS. Default is no check.

If you set `["domain1", "domain2"]` to `cors_allow_origins`, `in_http` returns `403` to access from othe domains.

### format

The format of the HTTP body. The default is `default`.

* default

Accept records using `json=` / `msgpack=` style.

* regexp

Specify body format by regular expression.

```text
format /^(?<field1>\d+):(?<field2>\w+)$/
```

If you execute following command:

```text
$ curl -X POST -d '123456:awesome' "http://localhost:8888/test.tag.here"
```

then got parsed result like below:

```text
{"field1":"123456","field2":"awesome}
```

`json`, `ltsv`, `tsv`, `csv` and `none` are also supported. Check [parser plugin overview](../parser/) for more details.

#### log\_level option

The `log_level` option allows the user to set different levels of logging for each plugin. The supported log levels are: `fatal`, `error`, `warn`, `info`, `debug`, and `trace`.

Please see the [logging article](../deployment/logging.md) for further details.

## Additional Features

### time query parameter

If you want to pass the event time from your application, please use the `time` query parameter.

```text
$ curl -X POST -d 'json={"action":"login","user":2}'
  "http://localhost:8888/test.tag.here?time=1392021185"
```

### Batch mode

If you use `default` format, then you can send array type of json / msgpack to in\_http.

```text
$ curl -X POST -d 'json=[{"action":"login","user":2,"time":1392021185},{"action":"logout","user":2,"time":1392027356}]'
  http://localhost:8888/test.tag.here;
```

This improves the input performance by reducing HTTP access. Non `default` format doesn't support batch mode yet. Here is a simple bechmark result on MacBook Pro with ruby 2.3:

| json | msgpack | msgpack array\(10 items\) |
| :--- | :--- | :--- |
| 2100 events/sec | 2400 events/sec | 10000 events/sec |

Tested configuration and ruby script is [here](https://gist.github.com/repeatedly/672ac73abf7cbcb629aaec791838cf6d).

## FAQ

### Why in\_http removes '+' from my log?

This is HTTP spec, not fluentd problem. You need to encode your payload properly or use multipart request. Here is ruby example:

```text
# OK
URI.encode_www_form({json: {"message" => "foo+bar"}.to_json})

# NG
"json=#{{"message" => "foo+bar"}.to_json}"
```

curl command example:

```text
# OK
curl -X POST -H 'Content-Type: multipart/form-data' -F 'json={"message":"foo+bar"}' http://localhost:8888/test.tag.here

# NG
curl -X POST -F 'json={"message":"foo+bar"}' http://localhost:8888/test.tag.here
```

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open). [Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation \(CNCF\)](https://cncf.io/). All components are available under the Apache 2 License.

