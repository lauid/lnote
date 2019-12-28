---
title: nginx-logstash
date: 2019-12-28 17:07:55
tags: nginx,elk
---

#### nginx日志格式定义

```
  log_format glog '$remote_addr [$time_local] $request_method $scheme $http_host "$request_uri" '
                  '$status $body_bytes_sent "$http_referer" "$http_user_agent" $request_time '
                          '$hostname "$request_body" $req_id';

```

<!--more-->

lua 定义请求id

```
set_by_lua_block $req_id {
	if ngx.var.http_x_nl_request_id == nil then
		return io.open("/proc/sys/kernel/random/uuid"):read()
	else
		return ngx.var.http_x_nl_request_id
	end
}
```

logstash配置
```
input {
	stdin {
	}
}

filter {
	grok {
		match => { "message" => "%{IPORHOST:clientip} \[%{HTTPDATE:timestamp}\] %{WORD:verb} %{URIPROTO:scheme} %{URIHOST:domain} \"%{URIPATH:uri_path}(?:%{URIPARAM:uri_args})?\" %{NUMBER:response} (?:%{NUMBER:bytes}|-) (?:\"(?:%{URI:referrer}|-)\"|%{QS:referrer}) %{QS:agent} %{BASE10NUM:request_duration} %{HOSTNAME:hostname} \"%{DATA:post_data}\" %{UUID:uuid}"}
	}
	date {
		match => [ "timestamp", "dd/MMM/YYYY:HH:mm:ss Z"]  
		target => "@timestamp"    
		remove_field => ["timestamp"]
	}
	mutate {
		split => ["uri_args", "?"]
	}
	mutate {
		join => ["uri_args", ""]
	}
	kv {
            source => "uri_args"
            field_split => "&"
            target => "args"
	}

	urldecode{
		field=>[post_data]
	}

	useragent {
		source => "agent"
		target => "ua"
	}

	geoip {
		source => "clientip"
	}
}
output {
	elasticsearch {
		hosts => [ "192.168.1.103:9200" ]
		index => "logstash-test1-%{+YYYY.MM.dd}"
    		document_id => "%{uuid}"
		template_overwrite => true
	}
	stdout { codec => rubydebug }
}
```

logstash-7.0.1/bin/logstash -f logstash-7.0.1/config/conf.d/nginx.conf --config.reload.automatic
