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
