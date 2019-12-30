# elastalert ElasticSerach报警插件
## 1. 官方文档 py3版本-最新版
https://elastalert.readthedocs.io/en/latest/elastalert.html
## 2. 官方文档 py2版本-稳定版
https://elastalert.readthedocs.io/en/stable/running_elastalert.html#requirements
## 3. 概述
我们将ElastAlert设计为可靠，高度模块化，并且易于设置和配置。

它通过将Elasticsearch与两种类型的组件（规则类型和警报）结合使用。定期查询Elasticsearch，并将数据传递到规则类型，该规则类型确定何时找到匹配项。发生匹配时，将为该警报提供一个或多个警报，这些警报将根据匹配采取行动。

这是由一组规则配置的，每个规则定义一个查询，一个规则类型和一组警报。

ElastAlert包含几种具有常见监视范例的规则类型：

* “匹配Y时间有X个事件的地方”（`frequency`类型）
* “在事件发生率增加或减少时进行匹配”（`spike`类型）
* “ Y时间内少于X个事件时匹配”（`flatline`类型）
* “当某个字段与黑名单/白名单匹配时匹配”（`blacklist`并`whitelist`键入）
* “匹配与给定过滤器匹配的任何事件”（`any`类型）
* “当某个字段在一段时间内具有两个不同的值时进行匹配”（`change`类型）

当前，我们为以下警报类型提供了内置支持：
* Command
* Email
* JIRA
* OpsGenie
* SNS
* HipChat
* Slack
* Telegram
* GoogleChat
* Debug
* Stomp
* theHive

## 4. 配置说明
### 4.1 统一配置文件config.yaml（在elastalert根目录下config.yaml.example）
```yaml
# This is the folder that contains the rule yaml files
# Any .yaml file will be loaded as a rule
rules_folder: /data/elastalert/rules

# How often ElastAlert will query Elasticsearch
# The unit can be anything from weeks to seconds
run_every:
  minutes: 1

# ElastAlert will buffer results from the most recent
# period of time, in case some log sources are not in real time
buffer_time:
  minutes: 15

# The Elasticsearch hostname for metadata writeback
# Note that every rule can have its own Elasticsearch host
es_host: elk01

# The Elasticsearch port
es_port: 9200

# The AWS region to use. Set this when using AWS-managed elasticsearch
#aws_region: us-east-1

# The AWS profile to use. Use this if you are using an aws-cli profile.
# See http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html
# for details
#profile: test

# Optional URL prefix for Elasticsearch
#es_url_prefix: elasticsearch

# Connect with TLS to Elasticsearch
#use_ssl: True

# Verify TLS certificates
#verify_certs: True

# GET request with body is the default option for Elasticsearch.
# If it fails for some reason, you can pass 'GET', 'POST' or 'source'.
# See http://elasticsearch-py.readthedocs.io/en/master/connection.html?highlight=send_get_body_as#transport
# for details
#es_send_get_body_as: GET

# Option basic-auth username and password for Elasticsearch
#es_username: someusername
#es_password: somepassword

# Use SSL authentication with client certificates client_cert must be
# a pem file containing both cert and key for client
#verify_certs: True
#ca_certs: /path/to/cacert.pem
#client_cert: /path/to/client_cert.pem
#client_key: /path/to/client_key.key

# The index on es_host which is used for metadata storage
# This can be a unmapped index, but it is recommended that you run
# elastalert-create-index to set a mapping
writeback_index: elastalert_status
#writeback_alias: elastalert_alerts

# If an alert fails for some reason, ElastAlert will retry
# sending the alert until this time period has elapsed
alert_time_limit:
  days: 2

# Custom logging configuration
# If you want to setup your own logging configuration to log into
# files as well or to Logstash and/or modify log levels, use
# the configuration below and adjust to your needs.
# Note: if you run ElastAlert with --verbose/--debug, the log level of
# the "elastalert" logger is changed to INFO, if not already INFO/DEBUG.
#logging:
#  version: 1
#  incremental: false
#  disable_existing_loggers: false
#  formatters:
#    logline:
#      format: '%(asctime)s %(levelname)+8s %(name)+20s %(message)s'
#
#    handlers:
#      console:
#        class: logging.StreamHandler
#        formatter: logline
#        level: DEBUG
#        stream: ext://sys.stderr
#
#      file:
#        class : logging.FileHandler
#        formatter: logline
#        level: DEBUG
#        filename: elastalert.log
#
#    loggers:
#      elastalert:
#        level: WARN
#        handlers: []
#        propagate: true
#
#      elasticsearch:
#        level: WARN
#        handlers: []
#        propagate: true
#
#      elasticsearch.trace:
#        level: WARN
#        handlers: []
#        propagate: true
#
#      '':  # root logger
#        level: WARN
#          handlers:
#            - console
#            - file
#        propagate: false

```
* `rules_folder`： 是ElastAlert从中加载规则配置文件的位置。它将尝试加载文件夹中的每个.yaml文件。没有任何有效规则，ElastAlert将无法启动。随着此文件夹中文件的更改，ElastAlert还将加载新规则，停止运行丢失的规则并重新启动修改后的规则。在本教程中，我们将使用example_rules文件夹。

* `run_every`： 是ElastAlert多久查询一次Elasticsearch的时间。

* `buffer_time`： 是查询窗口的大小，从运行每个查询的时间开始向后延伸。对于其中`use_count_query`或`use_terms_query`设置为true的规则，将忽略此值。

* `es_host`： 是Elasticsearch群集的地址，ElastAlert将在其中存储有关其状态，查询运行，警报和错误的数据。每个规则还可以使用不同的Elasticsearch主机进行查询。

* `es_port`： 是对应的端口es_host。

* `use_ssl`： 可选的; 是否es_host使用TLS 连接；设置为True或False。

* `verify_certs`： 可选的; 是否验证TLS证书；设置为True或False。默认是True

* `client_cert`： 可选的; 用作客户端证书的PEM证书的路径

* `client_key`： 可选的; 用作客户端密钥的私钥文件的路径

* `ca_certs`： 可选的; 用于验证SSL连接的CA证书捆绑包的路径

* `es_username`： 可选的; 用于连接的basic-auth用户名es_host。

* `es_password`： 可选的; 用于连接的basic-auth密码es_host。

* `es_url_prefix`： 可选的; Elasticsearch端点的URL前缀。

* `es_send_get_body_as`： 可选的; 方法查询Elasticsearch - GET，POST或source。默认是GET

* `writeback_index`： ElastAlert将在其中存储数据的索引的名称。我们稍后将创建此索引。

* `alert_time_limit`： 是失败警报的重试窗口。
* `es_conn_timeout`： 可选的; 设置连接和读取的超时es_host; 默认为20。
* `rules_loader`： 可选的; 设置ElastAlert用来检索规则和哈希值的加载程序类。FileRulesLoader如果未设置，则默认为。

* `rules_folder`：包含规则配置文件的文件夹的名称。ElastAlert将加载此文件夹中的所有文件以及所有以.yaml结尾的子目录。如果此文件夹的内容更改，则ElastAlert将根据其各自的配置文件加载，重新加载或删除规则。（仅在使用时需要`FileRulesLoader`）。

* `scan_subdirectories`： 可选的; 设置ElastAlert是否应以递归方式放到规则目录- true或false。默认是true
* `max_query_size`：在单个查询中将从Elasticsearch下载的最大文档数。默认值为10,000，如果您希望接近该数字，请考虑使用`use_count_query`。如果达到此限制，则在设置时或直到处理所有结果之前，ElastAlert将 使用设置的页面大小滚动。`max_query_sizemax_scrolling_count`

* `max_scrolling_count`：要滚动的最大页面数。默认值为0，这表示滚动没有限制。例如，如果将此值设置为5并将`max_query_size`设置为，10000则50000最多将下载文档。

* `scroll_keepalive`：滚动上下文的最长时间（以时间单位表示）应保持活动状态。避免使用高值，因为它会浪费Elasticsearch中的资源，但请留意留出足够的时间来完成所有结果的处理。

* `max_aggregation`：要汇总在一起的最大警报数。如果`aggregation`设置了规则，则在时间范围内发生的所有警报将一起发送。默认值为10,000。

* `old_query_limit`：查询ElastAlert从最近运行的查询开始的最长间隔时间。ElastAlert启动时，对于每个规则，它将搜索`elastalert_metadata`最近运行的查询并从该时间开始，除非它早于`old_query_limit`，在这种情况下它将从当前时间开始。默认值为一周。

* `disable_rules_on_error`：如果为true，则ElastAlert将禁用引发未捕获（不是EAException）异常的规则。它将向上载回溯消息`elastalert_metadata`，如果`notify_email`已设置，则发送电子邮件通知。在重新启动ElastAlert或修改规则文件之前，将不再运行规则。默认为True。

* `show_disabled_rules`：如果为true，则ElastAlert在完成执行时显示禁用规则的列表。默认为True。

* `notify_email`：通知电子邮件将发送到的电子邮件地址或电子邮件地址列表。当前，只有未捕获的异常会发送通知电子邮件。所述从地址，SMTP主机，并回复到报头可以使用被设置`from_addr`，`smtp_host`和`email_reply_to`选项分别。默认情况下，不发送电子邮件。

* `from_addr`：用作电子邮件通知中发件人标题的地址。该值也将用于电子邮件警报，除非在规则配置中被覆盖。默认值为“ ElastAlert”。

* `smtp_host`：用于发送电子邮件通知的SMTP主机。该值也将用于电子邮件警报，除非在规则配置中被覆盖。默认值为“ localhost”。

* `email_reply_to`：这将设置电子邮件中的Reply-To标头。默认值为收件人地址。

* `aws_region`：这使ElastAlert在使用Amazon Elasticsearch Service时签署HTTP请求。它将使用实例角色密钥对请求进行签名。环境变量AWS_DEFAULT_REGION将覆盖此字段。

* `boto_profile`：已弃用！如果您不想使用实例角色密钥，则在对Amazon Elasticsearch Service的请求签名时使用的Boto配置文件。

* `profile`：如果您不想使用实例角色密钥，则在对Amazon Elasticsearch Service的请求签名时使用的AWS配置文件。环境变量AWS_DEFAULT_PROFILE将覆盖此字段。

* `replace_dots_in_field_names`：如果为True，则ElastAlert在将文档写入Elasticsearch之前用下划线替换字段名称中的任何点。默认值为False。Elasticsearch 2.0-2.3不支持字段名称中的点。

* `string_multi_field_name`：如果设置，则后缀用于Elasticsearch中字符串多字段的子字段。默认值为.rawElasticsearch 2和.keywordElasticsearch 5。

* `add_metadata_alert`：如果置位，警报将包括元数据中所描述的规则（category，description，owner和priority）; 设置为True或False。默认值为False。

* `skip_invalid`：如果为True，请跳过无效文件而不退出。
默认情况下，ElastAlert使用简单的基本日志记录配置将日志消息打印为标准错误。您可以INFO使用`--verbose`或`--debug`命令行选项将日志级别更改为消息。

如果您需要更复杂的日志记录配置，则可以在config文件中提供完整的日志记录配置。这样，您还可以将日志记录配置为文件，Logstash并调整日志记录格式。

有关详细信息，请参见末尾`config.yaml.example`的示例日志记录配置。


### 4.2 创建规则
每个规则都定义要执行的查询，触发匹配的参数以及每个匹配要触发的警报列表。我们将使用它example_rules/example_frequency.yaml作为模板：
```yaml
# Alert when the rate of events exceeds a threshold

# (Optional)
# Elasticsearch host
es_host: elk01

# (Optional)
# Elasticsearch port
es_port: 9200

# (OptionaL) Connect with SSL to Elasticsearch
#use_ssl: True

# (Optional) basic-auth username and password for Elasticsearch
#es_username: someusername
#es_password: somepassword

# (Required)
# Rule name, must be unique
name: hdfs_append_error

# (Required)
# Type of alert.
# the frequency rule type alerts when num_events events occur with timeframe time
type: frequency

# (Required)
# Index to search, wildcard supported
index: log-error-data-synchro-*

# (Required, frequency specific)
# Alert when this many documents matching the query occur within a timeframe
num_events: 1

# (Required, frequency specific)
# num_events must occur within this amount of time to trigger an alert
timeframe:
  hours: 1

# (Required)
# A list of Elasticsearch filters used for find events
# These filters are joined with AND and nested in a filtered query
# For more info: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl.html
filter:
- bool:
    must: 
    - match_phrase: 
        message: "哈哈哈哈"
      #must:
       # - match_phrase:
        #    stack_trace: "is not sufficiently replicated yet"
# (Required)
# The alert is use when a match is found
alert:
- "email"

# (required, email specific)
# a list of email addresses to send alerts to
email:
- "1111111@qq.com"
```
* `es_host`：并es_port应指向我们要查询的Elasticsearch集群。

* `name`：是此规则的唯一名称。如果两个规则共享相同的名称，则ElastAlert将不会启动。

* `type`：每个规则具有不同的类型，可能采用不同的参数。该frequency类型的意思是“当num_events出现多个警报时发出警报timeframe。”有关其他类型的信息，请参阅规则类型。

* `index`：要查询的索引的名称。如果您使用的是Logstash，则默认情况下索引将匹配"logstash-*"。

* `num_events`：此参数特定于frequency类型，并且是触发警报时的阈值。

* `timeframe`：是num_events必须发生的时间段。

* `filter`：是用于过滤结果的Elasticsearch过滤器列表。在这里，我们为具有some_fieldmatch的文档提供了一个单项过滤器some_value。有关更多信息，请参见为规则编写过滤器。如果不需要过滤器，则应将其指定为空列表：filter: []

* `alert`：是在每次比赛中运行的警报的列表。有关警报类型的更多信息，请参阅警报。电子邮件警报要求使用SMTP服务器发送邮件。默认情况下，它将尝试使用localhost。可以使用该smtp_host选项进行更改。

* `email`：是警报将发送到的地址列表。

还有许多其他可选的配置选项，请参阅[“ 常见配置选项”](https://elastalert.readthedocs.io/en/stable/ruletypes.html#commonconfig)  。  
`所有文档都必须有一个时间戳字段。ElastAlert将`@timestamp`默认尝试使用，但是可以使用`timestamp_field`选项更改。默认情况下，ElastAlert使用`ISO8601时间戳，尽管通过设置支持Unix时间戳`timestamp_type`。

本例子中，根据filter的规则去es搜索数据，如果在1个小时内大于1个以上的文档，则发送邮件给1111111@qq.com
### 4.3 测试规则
```shell
# 使用默认配置
elastalert-test-rule example_rules / example_frequency.yaml
# 使用指定配置文件
elastalert-test-rule --config <path-to-config-file> example_rules/example_frequency.yaml
```
配置首选项将按以下方式加载：  
* 在yaml文件中指定的配置。
* 配置文件中指定的配置（如果已指定）。
* 默认配置，使该工具运行。
### 4.4 运行ElastAlert
```shell
$ python elastalert/elastalert.py
```
运行ElastAlert时，有几个参数可用：

* `--config`：将指定要使用的配置文件。默认值为config.yaml。

* `--debug`：将在调试模式下运行ElastAlert。这将增加日志记录的详细程度，将所有警报更改为DebugAlerter，以打印警报并取消其常规操作，并跳过将搜索和警报元数据写回到Elasticsearch的过程。与–verbose不兼容。

* `--verbose`：将增加日志记录的详细程度，使您可以查看有关查询状态的信息。与–debug不兼容。

* `--start <timestamp>`：将强制ElastAlert从给定时间开始查询，而不是从当前时间开始查询。时间戳记应为ISO8601，例如 YYYY-MM-DDTHH:MM:SS（UTC）或带时区 YYYY-MM-DDTHH:MM:SS-08:00（PST）。请注意，如果查询的日期范围较大，则在该规则在整个时间段内完成查询之前，不会发送任何警报。要从当前时间强制查询，请使用“ NOW”。

* `--end <timestamp>`：将导致ElastAlert在指定的时间戳记停止查询。默认情况下，ElastAlert将定期查询直到当前为止。

* `--rule <rule.yaml>`：将只运行给定的规则。规则文件可以是完整的文件路径，也可以是rules_folder 其或其子目录中的文件名。

* `--silence <unit>=<number>`：将在一段时间内使给定规则的警报静音。必须使用指定规则 --rule。<单位>是天，周，小时，分钟或秒之一。<number>是整数。例如， 将在4小时内阻止noisy_rule生成任何警报。--rule noisy_rule.yaml --silence hours=4

* `--es_debug`：将为所有对Elasticsearch的查询启用日志记录。

* `--es_debug_trace <trace.log>`：将为所有对Elasticsearch的所有查询启用curl命令记录到指定的日志文件。--es_debug_trace传递给elasticsearch.py，该日志记录localhost：9200 而不是实际的es_host：es_port。

* `--end <timestamp>`：将强制ElastAlert在给定时间（而不是默认时间）之后停止查询到当前时间。这仅在独立运行时才有意义。时间戳记格式为YYYY-MM-DDTHH:MM:SS（UTC）或时区YYYY-MM-DDTHH:MM:SS-XX:00（UTC-XX）。

* `--pin_rules`：将根据其配置文件的更改阻止ElastAlert加载，重新加载或删除规则。
### 4.5 运行一个demo
```yaml
$ python -m elastalert.elastalert --verbose --config  /data/elastalert/config.yaml --rule /data/elastalert/rules/example_frequency.yaml
# or use the entry point: elastalert --verbose --rule ...
No handlers could be found for logger "Elasticsearch"
INFO:root:Queried rule Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 hits
INFO:Elasticsearch:POST http://elasticsearch.example.com:14900/elastalert_status/elastalert_status?op_type=create [status:201 request:0.025s]
INFO:root:Ran Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 query hits (0 already seen), 0 matches, 0 alerts sent
INFO:root:Sleeping for 297 seconds
```
ElastAlert使用python日志记录系统并将其--verbose设置为显示INFO级别的消息。指定要运行的规则，否则ElastAlert将尝试在example_rules文件夹中加载其他规则。--rule example_frequency.yaml

让我们分解一下响应，看看发生了什么。

Queried rule Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 hits

ElastAlert定期查询最近buffer_time（默认为45分钟）以查找与过滤器匹配的数据。在这里，我们看到它匹配了5个匹配。

POST http://elasticsearch.example.com:14900/elastalert_status/elastalert_status?op_type=create [status:201 request:0.025s]

此行显示ElastAlert将文档上传到elastalert_status索引，其中包含有关它刚进行的查询的信息。

Ran Example rule from 1-15 14:22 PST to 1-15 15:07 PST: 5 query hits (0 already seen), 0 matches, 0 alerts sent

该行表示ElastAlert已完成处理规则。在较长的时间段内，有时可能会运行多个查询，但它们的数据将一起处理。是从Elasticsearch下载的文档数，指的是在先前的重叠查询中已经计数并且将被忽略的文档，是输出的规则类型的匹配数，是实际发送的警报数。这可能由于诸如和或由于错误的选项而有所不同。`query hits` `already seen` `matches` `alerts sent` `matches` `realert` `aggregation`

`Sleeping for 297 seconds`

默认`run_every值`为5分钟，这意味着ElastAlert将休眠，直到从最后一个周期开始经过5分钟为止，然后再对每个规则运行查询，时间范围向前移动5分钟。

假设在接下来的297秒内，Elasticsearch又添加了46个匹配的文档：
```
INFO:root:Queried rule Example rule from 1-15 14:27 PST to 1-15 15:12 PST: 51 hits
...
INFO:root:Sent email to ['elastalert@example.com']
...
INFO:root:Ran Example rule from 1-15 14:27 PST to 1-15 15:12 PST: 51 query hits, 1 matches, 1 alerts sent
```
电子邮件的正文将包含以下内容：
```
Example rule

At least 50 events occurred between 1-15 11:12 PST and 1-15 15:12 PST

@timestamp: 2015-01-15T15:12:00-08:00
```
如果发生错误，例如无法访问的SMTP服务器，您可能会看到：

`ERROR:root:Error while running alert email: Error connecting to SMTP host: [Errno 61] Connection refused`

请注意，如果停止ElastAlert，然后稍后再次运行它，它将查找`elastalert_status`并在上一次查询的结束时间开始查询。如果重新启动ElastAlert，这可以防止重复或跳过警报。

通过使用`--debug`标志代替`--verbose`，将记录电子邮件的正文，并且将不发送电子邮件。此外，查询将不会保存到`elastalert_status`。

### 4.6 配置邮箱
#### 4.6.1 在config中配置
```
from_addr: sziov20188888@163.com
smtp_host: smtp.163.com
smtp_auth_file: /data/elastalert/smtpconfig/smtp_auth_file.yaml
smtp_port: 25
```
#### 4.6.1 smtp帐号密码配置
```
user: "xxxxxxxxxx@163.com"
password: "yyyyyy"
```
#### 4.6.1 config或者rule中配置
```
alert:
- "email"

# (required, email specific)
# a list of email addresses to send alerts to
email:
- "11111@qq.com"
```
