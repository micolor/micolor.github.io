# ELK日志系统搭建

## ELK

### 安装

docker-compose配置参考

```yml
version: "3.7"
services:
  elasticsearch:
    container_name: elasticsearch
    image: elasticsearch:7.7.0
    ports:
      - 9200:9200
    networks:
      - elk-net
    environment:
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
      - cluster.name=elasticsearch
      - discovery.type=single-node
    volumes:
      - ./elasticsearch/data:/usr/share/elasticsearch/data
      - ./elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
  kibana:
    image: kibana:7.7.0
    container_name: kibana
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    ports:
      - 5601:5601
    networks:
      - elk-net
    volumes:
      - ./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml
  logstash:
    image: logstash:7.7.0
    container_name: logstash
    restart: always
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
      - ./logstash/patterns:/usr/local/logstash/patterns
    depends_on:
      - elasticsearch
    ports:
      - 5044:5044
    networks:
      - elk-net
networks:
  elk-net:
```

### elasticsearch 配置

elasticsearch.yml参考

```yml
cluster.name: "docker-cluster"
network.host: 0.0.0.0
action.auto_create_index: true
#
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
#
xpack.ml.enabled: false
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
```

### logstash 配置

```json
input {
    # beats {
    #     port => 5044
    #     codec => json    
    # }
  redis {
    data_type => "list"
    key => "filebeat"
    host => "redis.jguocloud.com"
    port => "6379"
    db => "254"
    password => "YuanXiao@1314"
  }
}
filter {
    if [fields][type] == "spring" {
      grok {
          match => ["message", '%{TIMESTAMP_ISO8601:time}\s* \s*%{NOTSPACE:thread-id}\s* \s*%{LOGLEVEL:level}\s* \s*%{JAVACLASS:class}\s* \- \s*%{JAVALOGMESSAGE:logmsg}\s*']
      }
    }
    if [fields][type] == "nginx-access" {
      # 112.39.68.184 - - [13/Mar/2022:00:59:48 +0800] wx.yiyuanjianshi.com POST "/interfaces/api/eduCourseKpoint/list" "-" 200 26583 "https://servicewechat.com/wx48813774c10830eb/35/page-frame.html" 200 127.0.0.1:8083 0.132 0.132 "Mozilla/5.0 (iPhone; CPU iPhone OS 11_4 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Mobile/15F79 MicroMessenger/8.0.15(0x18000f2e) NetType/WIFI Language/zh_CN" "-"
      grok {
          match => ["message","%{IPORHOST:remote_user} - (%{USERNAME:remote_user}|-) \[%{HTTPDATE:time_local}\] (%{HOSTNAME:http_host}|-) (%{WORD:request_method}|-) \"(%{URIPATHPARAM:uri}|-|)\" \"(%{GREEDYDATA:query_string}|-)\" %{NUMBER:http_status} (?:%{BASE10NUM:body_bytes_sent}|-) \"(%{GREEDYDATA:http_referer}|-)\" %{NUMBER:upstream_status} (?:%{HOSTPORT:upstream_addr}|-) (%{BASE16FLOAT:request_time}|-) (%{NUMBER:upstream_response_time}|-) \"(%{GREEDYDATA:user_agent}|-)\" \"(%{GREEDYDATA:x_forword_for}|-)\""]
      }
      date {
          match => [ "time_local" , "dd/MMM/YYYY:HH:mm:ss Z" ]
      }
      mutate {
          remove_field => ["message"]
      }
    }
}
output{
    elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "%{[tags]}-%{+YYYY.MM.dd}"
      user => "elastic"
	    password => "1qaz2ys"
    }
    stdout { codec => rubydebug }
}
```

### kibana 配置

kibana.yml参考

```
#
# ** THIS IS AN AUTO-GENERATED FILE **
#
# Default Kibana configuration for docker target
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://elasticsearch:9200" ]
elasticsearch.requestTimeout: 60000
monitoring.ui.container.elasticsearch.enabled: true
kibana.index: ".kibana"
i18n.locale: "zh-CN"
elasticsearch.username: "elastic"
elasticsearch.password: "1qaz2ys"
xpack.reporting.encryptionKey: "a_random_string"
xpack.security.encryptionKey: "something_at_least_32_characters"
```

### elasticsearch 测试

```shell
#访问
curl localhost:9200 -u elastic:{password}
#修改密码
curl -XPOST -u elastic "localhost:9200/_security/user/elastic/_password" -H 'Content-Type: application/json' -d'{"password" : "abcd1234"}'
```



## FILEBEAT

### 安装

docker-compose配置参考

```yml
version: "3.7"
services:
  filebeat:
    image: docker.elastic.co/beats/filebeat:7.1.1
    container_name: filebeat
    volumes:
      - /var/log/nginx:/var/log/nginx/
      - /home/yyjs/interfaces_a:/var/log/interfacesa
      - /home/yyjs/interfaces_b:/var/log/interfacesb
      - ./filebeat/conf/filebeat.yml:/usr/share/filebeat/filebeat.yml
```

### 配置

filebeat.yml参考

```yml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: true
processors:
- add_cloud_metadata: ~

filebeat.inputs:
- type: log
  enabled: true
  close_inactive: 15m
  paths:
    - /var/log/nginx/access.log
  fields:
    type: "nginx-access"
  tags: nginx-access
- type: log
  enabled: true
  close_inactive: 15m
  paths:
    - /var/log/nginx/error.log
  fields:
    type: "nginx-error"
  tags: nginx-error
- type: log
  enabled: true
  close_inactive: 15m
  paths:
    - /var/log/interfacesa/*.log
    - /var/log/interfacesb/*.log
  fields:
    type: "spring"
  multiline.pattern: ^[0-9]{4}-[0-9]{2}-[0-9]{2}
  multiline.negate: true
  multiline.match: after
  tags: spring-interface

output.redis:
  hosts: ["redis.jguocloud.com:6379"]
  password: "YuanXiao@1314"
  key: "filebeat"
  db: 254
  timeout: 45           
```

## Nginx grok配置

### nginx自定访问义日志

修改nginx.conf

```json
log_format  main  '$remote_addr - $remote_user [$time_local] $http_host $request_method "$uri" "$query_string" '
'$status $body_bytes_sent "$http_referer" $upstream_status $upstream_addr $request_time $upstream_response_time '
'"$http_user_agent" "$http_x_forwarded_for"' ;
access_log  /var/log/nginx/access.log  main;
```

匹配规则

```
"%{IPORHOST:remote_user} - (%{USERNAME:remote_user}|-) \[%{HTTPDATE:time_local}\] (%{HOSTNAME:http_host}|-) (%{WORD:request_method}|-) \"(%{URIPATHPARAM:uri}|-|)\" \"(%{GREEDYDATA:query_string}|-)\" %{NUMBER:http_status} (?:%{BASE10NUM:body_bytes_sent}|-) \"(%{GREEDYDATA:http_referer}|-)\" %{NUMBER:upstream_status} (?:%{HOSTPORT:upstream_addr}|-) (%{BASE16FLOAT:request_time}|-) (%{NUMBER:upstream_response_time}|-) \"(%{GREEDYDATA:user_agent}|-)\" \"(%{GREEDYDATA:x_forword_for}|-)\""
```

