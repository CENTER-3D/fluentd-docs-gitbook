# Common Log Formats

This page is a glossary of common log formats that can be parsed with the [Tail input plugin]().

* Apache Access Log

  Use `format apache2` as shown below:

  ```text
    <source>
        @type tail
        format apache2
        tag apache.access
        path /var/log/apache2/access.log
    </source>
  ```

* Apache Error Log

  Use a regular expression. See the `format` field in the following sample configuration.

  ```text
    <source>
        @type tail
        format /^\[[^ ]* (?<time>[^\]]*)\] \[(?<level>[^\]]*)\] \[pid (?<pid>[^\]]*)\] \[client (?<client>[^\]]*)\] (?<message>.*)$/
        tag apache.error
        path /var/log/apache2/error.log
    </source>
  ```

  Depending on your particular error log format, you may need to adjust the regular expression above. You can test your format using [fluentd-ui's in\_tail editor](../deployment/fluentd-ui.md#intail-setting) or [Fluentular](http://fluentular.herokuapp.com).

* Maillog

  Use a regular expression. See the `format` field in the following sample configuration.

  ```text
    <source>
        @type tail
        format /^(?<time>[^ ]+) (?<host>[^ ]+) (?<process>[^:]+): (?<message>((?<key>[^ :]+)[ :])? ?((to|from)=<(?<address>[^>]+)>)?.*)$/
        tag postfix.maillog
        path /var/log/maillog
    </source>
  ```

* Nginx Access Log

  Use `format nginx` as shown below:

  ```text
    <source>
        @type tail
        format nginx
        tag nginx.access
        path /var/log/nginx/access.log
    </source>
  ```

* Nginx Error Log

  Use the `format*` and `multiline_flush_interval` fields in the following sample configuration. Applications running under Nginx can output multi-line errors including stack traces, so the multiline mode is a good fit.

  ```text
    <source>
        @type tail
        tag nginx.error
        path /var/log/nginx/error.log

        format multiline
        format_firstline /^\d{4}/\d{2}/\d{2} \d{2}:\d{2}:\d{2} \[\w+\] (?<pid>\d+).(?<tid>\d+): /
        format1 /^(?<time>\d{4}/\d{2}/\d{2} \d{2}:\d{2}:\d{2}) \[(?<log_level>\w+)\] (?<pid>\d+).(?<tid>\d+): (?<message>.*)/
        multiline_flush_interval 3s
    </source>
  ```

  If you know your error log will only contain single lines, you can use the below simpler configuration with just a `format`.

  ```text
    <source>
        @type tail
        format /^(?<time>\d{4}/\d{2}/\d{2} \d{2}:\d{2}:\d{2}) \[(?<log_level>\w+)\] (?<pid>\d+).(?<tid>\d+): (?<message>.*)$/
        tag nginx.error
        path /var/log/nginx/error.log
    </source>
  ```

* GlusterFS Logs

  Use the [GlusterFS input plugin.](collect-glusterfs-logs.md)

## Do you not see what you are looking for?

Give us a shout on [GitHub](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open), [Twitter](http://twitter.com/fluentd) or [the mailing list](https://groups.google.com/forum/#!forum/fluentd). Better yet, we always welcome a [pull request](https://github.com/fluent/fluentd-docs-gitbook/pulls)!

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open). [Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation \(CNCF\)](https://cncf.io/). All components are available under the Apache 2 License.

