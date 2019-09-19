# Example Configurations for Fluentd

## Inputs

### File Input

One of the most common types of log input is tailing a file. The in_tail input plugin allows you to read from a text log file as though you were running the _tail -f_ command. Full documentation on this plugin can be found [here](https://docs.fluentd.org/input/tail).

```
<source>
  @type tail
  <parse>
    @type none
  </parse>
  path /home/logs/*
  tag backend.application
</source>
```

In this tail example, we are declaring that the logs should not be parsed by seeting _@type none_. We are also adding a tag that will control routing. By setting _tag backend.application_ we can specify filter and match blocks that will only process the logs from this one source. More details on how routing works in Fluentd can be found [here](https://docs.fluentd.org/configuration/routing-examples). 

### Adding Parsing

Sometimes you will have logs which you wish to parse. There is a set of built-in parsers listed [here](https://docs.fluentd.org/parser#list-of-built-in-parsers) which can be applied. Some of the parsers like the _nginx_ parser understand a common log format and can parse it "automatically." Others like the _regexp_ parser are used to declare custom parsing logic. There is also a very commonly used 3rd party parser for [grok](https://github.com/fluent/fluent-plugin-grok-parser) that provides a set of regex macros to simplify parsing.

This next example is showing how we could parse a standard NGINX log we get from file using the in_tail plugin. 

```
<source>
  @type tail
  <parse>
    @type nginx
  </parse>
  path /var/log/nginx/error.log
  tag nginx.error
</source>
```

Notice that we have chosen to tag these logs as _nginx.error_ to help route them to a specific output and filter plugin after. If we wanted to apply custom parsing the _grok_ filter would be an excellent way of doing it. In this next example, a series of _grok patterns_ are used. The first pattern is _%{SYSLOGTIMESTAMP:timestamp}_ which pulls out a timestamp assuming the standard syslog timestamp format is used. The next pattern grabs the log level and the final one grabs the remaining unnmatched txt. Each substring matched becomes an attribute in the log event stored in New Relic. This makes it possible to do more advanced monitoring and alerting later by using those attributes to filter, search and facet.

```
<source>
  @type tail
  <parse>
    @type grok
    <grok>
      pattern %{SYSLOGTIMESTAMP:timestamp} %{LOGLEVEL:loglevel}: %{GREEDYDATA:message}
    </grok>
  </parse>
  path /home/log/test.log
  tag custom.application
</source>
```

### Multiline Logs

Some logs have single entries which span multiple lines. Typically one log entry is the equivalent of one log line; but what if you have a stack trace or other long message which is made up of multiple lines but is logically all one piece? In that case you can use a multiline parser with a regex that indicates where to start a new log entry. A common start would be a timestamp; whenever the line begins with a timestamp treat that as the start of a new log entry. If the next line begins with something else, continue appending it to the previous log entry. 

```
 <filter applog.test>
    @type parser
    <parse>
      @type multiline_grok
      grok_failure_key grokfailure
      multiline_start_regex ^abc
      <grok>
        pattern %{GREEDYDATA:message}
      </grok>
    </parse>
  </filter>
```
<filter backend.application>
  @type record_transformer
  <record>
    service_name backend.application
  </record>
</filter>

<source>
  @type syslog
  port 5140
  tag culmone
</source>




```
<match **>
  @type newrelic
  api_key 7JR6UXcGHjErDJ7w4uVOqtbN-u1vI2VD
</match>
```

https://docs.fluentd.org/parser
