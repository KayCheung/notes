<pre>
input {
  beats {
    port => 5044
  }
}


# 14:06:15.757 [main] INFO  s.d.s.w.r.o.CachingOperationNameGenerator - Generating unique operation named: getListUsingPOST_2
# 自定义：(?&lt;属性名称&gt;正则表达式)
filter {
	if [fields][appid] == "pointmall" {
		grok {
			match => {"message" => "(?&lt;logtime&gt;([0-9]{4}\-[0-9]{2}\-[0-9]{2}\s+[0-9]{2}:[0-9]{2}:[0-9]{2}+\.+[0-9]{3})) \[(?&lt;thread&gt;.*)\] %{LOGLEVEL:logLevel}  %{NOTSPACE:logger} - %{GREEDYDATA:msginfo}"}
		}
		date {
			match => ["logtime", "yyyy-MM-dd HH:mm:ss.SSS"]
			target => "@timestamp"
		}
		mutate{
			remove_field =>["message"]
		}
	}
}

output {
	if [fields][appid] == "pointmall" {
		file {
			path => "E:\\logstatsh.log"
		}
	}
}
</pre>
