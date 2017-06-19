---
layout: post
title: Elastalert + Slack 로그 알람 구축
---

흔히 ELK라고 부르는 [elastic](https://www.elastic.co/kr/)사의 제품 스택 ElasticSearch + LogStash + Kibaba 조합으로 로그 수집을 구축하면, 브라우저에서 Kibina를 통해 원하는 시간대의 데이터를 쿼리하고 시각화할 수 있고, 시각화한 데이터를 이용해 대시보드를 꾸미고 모니터링이 가능합니다.
하지만 하루 종일 대시보드를 보고 있을 수는 없기에 로그에서 특별한 상황이 발생하였을 때 즉각 알람을 받을 수 있는 시스템이 필요했습니다.
관련 정보를 찾던 중 데일리 호텔 미디엄에서 [Elastalert 로그 알람 구축](https://dailyhotel.io/elastalert%EB%A1%9C-%EB%A1%9C%EA%B7%B8-%EC%95%8C%EB%9E%8C-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-a5c770a86002#.zdszs5l71)이라는 포스트를 발견했습니다.

링크를 따라가 [Yelp 블로그](https://engineeringblog.yelp.com/2015/10/elastalert-alerting-at-scale-with-elasticsearch.html)에서 Elastalert에 대한 소개와 사용법을 확인했습니다.


## Elastalert
>ElastAlert is a simple framework for alerting on anomalies, spikes, or other patterns of interest from data in Elasticsearch. At Yelp, we use Elasticsearch, Logstash and Kibana for managing our ever increasing amount of data and logs. Kibana is great for visualizing and querying data, but we quickly realized that it needed a companion tool for alerting on inconsistencies in our data. Out of this need, ElastAlert was created. If you have data being written into Elasticsearch in near real time and want to be alerted when that data matches certain patterns, ElastAlert is the tool for you.

![elastalert](https://engineeringblog.yelp.com/images/posts/elastalert-part-two/elastalert.png)

Elastalert는 Elasticsearch에 있는 데이터로부터 비정상, 스파이크 혹은 다른 관심있는 패턴을 경고해주는 프레임워크입니다.
[Yelp](https://www.yelp.com/sf) 에서도 ELK로 로그를 수집/관리하고 있고 Kibana로 데이터를 쿼리하고 시각화 하고 있지만 데이터를 통한 알람 기능이 필요하여 직접 만들어 오픈소스로 [공개](https://github.com/Yelp/elastalert)했다. 99.9% 파이썬2 코드입니다.

stand alone 툴이라 설치 및 사용이 매우 간편하고, 
[문서화](http://elastalert.readthedocs.io/en/latest/elastalert.html)가 잘 되어 있습니다. 
YAML 포맷으로 간단하게 설정하고 룰만 지정하면 바로 동작합니다. 
알람이 필요한 대부분의 경우를 커버하는 알람 타입을 지원합니다.
- “Match where there are X events in Y time” (frequency type)
- “Match when the rate of events increases or decreases” (spike type)
- “Match when there are less than X events in Y time” (flatline type)
- “Match when a certain field matches a blacklist/whitelist” (blacklist and whitelist type)
- “Match on any event matching a given filter” (any type)
- “Match when a field has two different values within some time” (changetype)
- “Match when a never before seen term appears in a field” (new_term type)
- “Match when the number of unique values for a field is above or below a threshold (cardinality type)

다양한 경고 방법을 지원한다. 이메일은 기본이고 우리가 필요한 slack도 되며 심지어 알람이 발생할 때 자동으로 JIRA 테스트 생성도 가능합니다.
- command
- email
- Slack
- HipChat
- SNS
- Telegram
- JIRA
- ... 

## 설치

```
$ pip install elastalert
```

제가 설치한 서버는 python 2,7, 3.5 두 가지가 설치되어 있고, elastalert는 python2, 연동하려는 elasticsearch가  5.x 버전이라 약간 섬세함이 필요했습니다.

```
$ pip-2.7 install elasicsearch>=5.0.0
$ pip-2.7 install elastalert
```


## 설정
GitHub repo에 기본 설정 파일 [샘플](https://github.com/Yelp/elastalert/blob/master/config.yaml.example)이 올라가있습니다.

다운 받아서 필요에 맞게 수정합니다.

```bash
$ wget https://github.com/Yelp/elastalert/blob/master/config.yaml.example
$ vim config.yaml.example
```

#### config.yaml

```yaml
# This is the folder that contains the rule yaml files
# Any .yaml file will be loaded as a rule
rules_folder: example_rules

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
es_host: elasticsearch.example.com

# The Elasticsearch port
es_port: 9200

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
 
# The index on es_host which is used for metadata storage
# This can be a unmapped index, but it is recommended that you run
# elastalert-create-index to set a mapping
writeback_index: elastalert_status
 
# If an alert fails for some reason, ElastAlert will retry
# sending the alert until this time period has elapsed
alert_time_limit:
  days: 2
```



## Rule 설정
룰 또한 GitHub repo에 샘플이 다양하게 올라가있습니다. 마음에 드는 파일 아무거나 가져와서 필요에 맞게 수정했습니다.

첫 번째로 등록하려는 룰은
- loglevel이 ERROR인 로그가 (query string)
- 발생하기만 하면 (Any)
- slack으로 경고 (slack)
하는 룰이다.

다음과 같이 설정하였다.

#### error.yaml
```yaml
 # Alert when the rate of events exceeds a threshold
 
# (Required)
# Rule name, must be unique
name: ERROR Log
 
# (Required)
# Type of alert.
# the frequency rule type alerts when num_events events occur with timeframe time
# type: frequency
type: any
 
# (Required)
# Index to search, wildcard supported
index: logstash-*
 
# use_strftime_index: true
 
# (Required, frequency specific)
# Alert when this many documents matching the query occur within a timeframe
# num_events: 1
 
# (Required, frequency specific)
# num_events must occur within this amount of time to trigger an alert
# timeframe:
#     hours: 1
 
# (Required)
# A list of Elasticsearch filters used for find events
# These filters are joined with AND and nested in a filtered query
# For more info: http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/query-dsl.html
filter:
- query:
    query_string:
        query: "loglevel: ERROR"
 
# (Required)
# The alert is use when a match is found
alert:
- "slack":
    slack_webhook_url: "https://hooks.slack.com/services/xxxxxxxxxxxxxxxxxxxxxxxxx"
    slack_username_override: 'username'
    slack_channel_override: '#channel_to_send'
    slack_emoji_override: ':emoji:'
```


## Slack 알람 설정
slack으로 알람을 보내려면 해당 slack Team의 WebHook URL을 설정해야 합니다.
Incoming Webhook 기능은 지정된 URL로 http POST 메시지를 보내서 특정 채널로 메시지를 보낼 수 있는 기능입니다.
Slack에서 미리 Incoming WebHook 기능을 연동해두고 해당 Hook의 URL을 확인하여 설정합니다.
https://api.slack.com/incoming-webhooks#sending_messages


WebHook URL을 확인하면 위의 룰 파일에서 slack부분을 설정합니다.
webhook 설정 자체에 username, emoji, channel 등이 설정되어 있지만 elastalert에서 보낼 때 이 설정을 override할 수도 있습니다.
```yaml
slack_webhook_url: "https://hooks.slack.com/services/xxxxxxxxxxxxxxxxxxxxxxxxx"
slack_username_override: 'username'
slack_channel_override: '#channel_to_send'
slack_emoji_override: ':emoji:'
```


## 시험 가동
설정과 룰 셋팅이 끝났다면 잘 설정 되었는지 확인해볼 수 있다. 실제로 알람을 발생시키지는 않고 잘 되는지 테스트만할 수 있는 툴을 제공하고 있습니다.
```bash
$ elastalert-test-rule error.yaml
```

## 결과

잘 된다면 진짜로 실행합니다.
```bash
$ elastalert --verbose --rule error.yaml
```

지정된 조건을 만족하는 데이터가 발생하면 슬랙으로 알람이 오는 것을 확인할 수 있습니다.

![result](http://i.imgur.com/FV3hZXI.png)
