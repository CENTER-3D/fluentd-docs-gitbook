# Routing Examples

This article shows typical routing examples.

## Simple Input -&gt; Filter -&gt; Output

```text
<source>
  @type forward
</source>

<filter app.**>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match app.**>
  @type file
  # ...
</match>
```

### Two input cases

```text
<source>
  @type forward
</source>

<source>
  @type tail
  tag system.logs
  # ...
</source>

<filter app.**>
  @type record_transformer
  <record>
    hostname "#{Socket.gethostname}"
  </record>
</filter>

<match {app.**,system.logs}>
  @type file
  # ...
</match>
```

If you want to separate data pipeline for each sources, use Label.

## Input -&gt; Filter -&gt; Output with Label

Label reduces complex tag handling by separating data pipeline.

```text
<source>
  @type forward
</source>

<source>
  @type dstat
  @label @METRICS # dstat events are routed to <label @METRICS>
  # ...
</source>

<filter app.**>
  @type record_transformer
  <record>
    # ...
  </record>
</filter>

<match app.**>
  @type file
  # ...
</match>

<label @METRICS>
  <match **>
    @type elasticsearch
    # ...
  </match>
</label>
```

## Re-route event by tag

Use [fluent-plugin-route](https://github.com/tagomoris/fluent-plugin-route) plugin. `route` plugin rewrites tag and re-emit events to other match or Label.

```text
<match worker.**>
  @type route
  remove_tag_prefix worker
  add_tag_prefix metrics.event

  <route **>
    copy # For fall-through. Without copy, routing is stopped here. 
  </route>
  <route **>
    copy
    @label @BACKUP
  </route>
</match>

<match metrics.event.**>
  @type stdout
</match>

<label @BACKUP>
  <match metrics.event.**>
    @type file
    path /var/log/fluent/bakcup
  </match>
</label>
```

## Re-route event by record content

Use [fluent-plugin-rewrite-tag-filter](https://github.com/fluent/fluent-plugin-rewrite-tag-filter).

```text
<source>
  @type forward
</source>

# event example: app.logs {"message":"[info]: ..."}
<match app.**>
  @type rewrite_tag_filter
  rewriterule1 message ^\[(\w+)\] $1.${tag}
</match>

# send mail when receives alert level logs
<match alert.app.**>
  @type mail
  # ...
</match>

# other logs are stored into file
<match *.app.**>
  @type file
  # ...
</match>
```

See also [out\_rewrite\_tag\_filter]() article.

## Re-route event to other Label

Use [out\_relabel]() plugin. `relabel` plugin simply emits events to Label. No tag rewrite.

```text
<source>
  @type forward
</source>

<match app.**>
  @type copy
  <store>
    @type forward
    # ...
  </store>
  <store>
    @type relabel
    @label @NOTIFICATION
  </store>
</match>

<label @NOTIFICATION>
  <filter app.**>
    @type grep
    regexp1 message ERROR
  </filter>

  <match app.**>
    @type mail
  </match>
</label>
```

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open). [Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation \(CNCF\)](https://cncf.io/). All components are available under the Apache 2 License.

