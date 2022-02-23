调试grok
```
POST /_ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
        "grok": {
          "field": "message",
          "patterns": [
            """\[%{EVENTDATE:log.timestamp}\]%{SPACE}\[%{LOGLEVEL:log.level}\]%{SPACE}\[%{NOTSPACE:log.package}\]%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{LOGLEVEL:log.level}%{SPACE}%{EVENTDATE2:log.timestamp}%{SPACE}%{NOTSPACE:log.package}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{LOGLEVEL:log.level}%{SPACE}\[,%{WORD:log.traeid},%{WORD:log.spanid},%{WORD}\]%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{LOGLEVEL:log.level}%{SPACE}\[%{NOTSPACE:log.package}\]%{SPACE}%{GREEDYDATA:log.msg}""",
            """\[%{WORD:log.package}\]%{SPACE}\[%{LOGLEVEL:log.level}\]%{SPACE}%{EVENTDATE:log.timestamp}%{SPACE}%{GREEDYDATA:log.msg}""",
            """time=\"%{EVENTDATE:log.timestamp}Z\"%{SPACE}level=%{LOGLEVEL:log.level}%{SPACE}%{GREEDYDATA:log.msg}""",
            """time=\"%{EVENTDATE:log.timestamp}Z\"%{SPACE}level=%{LOGLEVEL:log.level}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{LOGLEVEL:log.level}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{GREEDYDATA:log.msg}"""
            
          ],"pattern_definitions": {
            "EVENTDATE": "%{YEAR}[/-]%{MONTHNUM}[/-]%{MONTHDAY}[T ]%{TIME}",
            "EVENTDATE2": "%{MONTHNUM}[/-]%{MONTHDAY}[T -]%{TIME}"
          
          }
        }
        
      } ,{"date" : {
          "target_field" : "@timestamp",
          "formats" : [
            "yyyy/MM/dd HH:mm:ss.SSS",
            "yyyy/MM/dd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy-MM-dd HH:mm:ss.SSS",
            "yyyy-MM-dd HH:mm:ss.SSS"
          ],
        
           "timezone" : "Asia/Shanghai",
          "on_failure" : [
            {
              "append" : {
                "field" : "error.message",
                "value" : "{{ _ingest.on_failure_message }}"
              }
            }
          ],
            "ignore_failure": true ,          
          "field" : "log.timestamp"
        }}, {
        "set" : {
          "field" : "log.kind",
          "value" : "event"
        }
      }
    
  ]
  },
  "docs": [
    {
      "_source": {
        "message": "time=\"2021-10-20T08:45:11+08:00\" level=info msg=\"GOENV=>test\" source=\"conf.go:28\""
      }
    }
  ]
}
```

```

PUT _ingest/pipeline/go_log_pipline
{
  "description" : "parsing go project log",
  "processors" : [
      {
        "grok": {
          "field": "message",
          "patterns": [
            """\[%{EVENTDATE:log.timestamp}\]%{SPACE}\[%{LOGLEVEL:log.level}\]%{SPACE}\[%{NOTSPACE:log.package}\]%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{LOGLEVEL:log.level}%{SPACE}%{EVENTDATE2:log.timestamp}%{SPACE}%{NOTSPACE:log.package}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{LOGLEVEL:log.level}%{SPACE}\[,%{WORD:log.traeid},%{WORD:log.spanid},%{WORD}\]%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{LOGLEVEL:log.level}%{SPACE}\[%{NOTSPACE:log.package}\]%{SPACE}%{GREEDYDATA:log.msg}""",
            """\[%{WORD:log.package}\]%{SPACE}\[%{LOGLEVEL:log.level}\]%{SPACE}%{EVENTDATE:log.timestamp}%{SPACE}%{GREEDYDATA:log.msg}""",
            """time=\"%{EVENTDATE:log.timestamp}Z\"%{SPACE}level=%{LOGLEVEL:log.level}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{LOGLEVEL:log.level}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{EVENTDATE:log.timestamp}%{SPACE}%{GREEDYDATA:log.msg}""",
            """%{GREEDYDATA:log.msg}"""
            
          ],"pattern_definitions": {
            "EVENTDATE": "%{YEAR}[/-]%{MONTHNUM}[/-]%{MONTHDAY}[T ]%{TIME}",
            "EVENTDATE2": "%{MONTHNUM}[/-]%{MONTHDAY}[T -]%{TIME}"
          
          }
        }
        
      } ,{"date" : {
          "target_field" : "@timestamp",
          "formats" : [
            "yyyy/MM/dd HH:mm:ss.SSS",
            "yyyy/MM/dd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy-MM-dd HH:mm:ss.SSS",
            "yyyy-MM-dd HH:mm:ss.SSS"
          ],
        
           "timezone" : "Asia/Shanghai",
          "on_failure" : [
            {
              "append" : {
                "field" : "error.message",
                "value" : "{{ _ingest.on_failure_message }}"
              }
            }
          ],
            "ignore_failure": true ,          
          "field" : "log.timestamp"
        }} ,{"date" : {
          "target_field" : "log.timestamp",
          "formats" : [
            "yyyy/MM/dd HH:mm:ss.SSS",
            "MM/dd-HH:mm:ss.SSS",
            "yyyy/MM/dd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy-MM-dd HH:mm:ss.SSS",
            "yyyy-MM-dd HH:mm:ss"
          ],
           "output_format":"yyyy/MM/dd HH:mm:ss",
           "timezone" : "Asia/Shanghai",
          "on_failure" : [
            {
              "append" : {
                "field" : "error.message",
                "value" : "{{ _ingest.on_failure_message }}"
              }
            }
          ],
            "ignore_failure": true ,          
          "field" : "log.timestamp"
        }}, {
        "set" : {
          "field" : "log.kind",
          "value" : "event"
        }
      },
      {
        "remove" : {
          "field" : "fields.index"
        }
      }
    
  ]
}


```
```

PUT _ingest/pipeline/php_log_pipline
{
  "description" : "parsing php project log",
  "processors" : [
      {
        "grok": {
          "field": "message",
          "patterns": [
            """%{IP} - - \[%{EVENTDATE3:log.timestamp} \+0000\]%{GREEDYDATA:log.msg}""",
            """\[%{EVENTDATE3:log.timestamp}\]%{GREEDYDATA:log.msg}""",
            """%{GREEDYDATA:log.msg}"""
            
          ],"pattern_definitions": {
            "EVENTDATE": "%{YEAR}[/-]%{MONTHNUM}[/-]%{MONTHDAY}[T ]%{TIME}",
            "EVENTDATE2": "%{MONTHNUM}[/-]%{MONTHDAY}[T -]%{TIME}",
            "EVENTDATE3": "%{MONTHDAY}[/-]%{MONTH}[/-]%{YEAR}[: ]%{TIME}"
          
          }
        }
        
      } ,{"date" : {
          "target_field" : "@timestamp",
          "formats" : [
            "yyyy/MM/dd HH:mm:ss.SSS",
            "yyyy/MM/dd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy-MM-dd HH:mm:ss.SSS",
            "yyyy-MM-dd HH:mm:ss.SSS"
          ],
           "output_format":"yyyy-MM-dd'T'HH:mm:ss.SSSZ",
           "timezone" : "Asia/Shanghai",
          "on_failure" : [
            {
              "append" : {
                "field" : "error.message",
                "value" : "{{ _ingest.on_failure_message }}"
              }
            }
          ],
            "ignore_failure": true ,          
          "field" : "log.timestamp"
        }},
      {
        "remove" : {
          "field" : "fields.index"
        }
      }
    
  ]
}
```
```
PUT _ingest/pipeline/vue_log_pipline
{
  "description" : "parsing vue project log",
  "processors" : [
      {
        "grok": {
          "field": "message",
          "patterns": [
            """%{GREEDYDATA:log.msg}"""
            
          ],"pattern_definitions": {
            "EVENTDATE": "%{YEAR}[/-]%{MONTHNUM}[/-]%{MONTHDAY}[T ]%{TIME}",
            "EVENTDATE2": "%{MONTHNUM}[/-]%{MONTHDAY}[T -]%{TIME}"
          
          }
        }
        
      } ,{"date" : {
          "target_field" : "@timestamp",
          "formats" : [
            "yyyy/MM/dd HH:mm:ss.SSS",
            "yyyy/MM/dd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy-MM-dd HH:mm:ss.SSS",
            "yyyy-MM-dd HH:mm:ss.SSS"
          ],
        
           "timezone" : "Asia/Shanghai",
          "on_failure" : [
            {
              "append" : {
                "field" : "error.message",
                "value" : "{{ _ingest.on_failure_message }}"
              }
            }
          ],
            "ignore_failure": true ,          
          "field" : "log.timestamp"
        }}, {
        "set" : {
          "field" : "log.kind",
          "value" : "event"
        }
      },
      {
        "remove" : {
          "field" : "fields.index"
        }
      }
    
  ]
}
```

event_log_pipeline
```kibana
PUT _ingest/pipeline/event_log_pipline
{
  "description" : "parsing k8s event log",
 "processors": [
      {
        "grok": {
          "field": "message",
          "patterns": [
            """I%{EVENTDATE3:log.timestamp}%{PACK:log.package}%{GREEDYDATA:log.msg}""", 
            """%{GREEDYDATA:log.msg}"""
            
          ],"pattern_definitions": {
            "PACK": "%{SPACE}%{GREEDYDATA}42]%{SPACE}",
            "EVENTDATE": "%{YEAR}[/-]%{MONTHNUM}[/-]%{MONTHDAY}[T ]%{TIME}",
            "EVENTDATE2": "%{MONTHNUM}[/-]%{MONTHDAY}[T -]%{TIME}",
            "EVENTDATE3": "%{MONTHNUM}%{MONTHDAY}[T -:]%{TIME}"
          }
        }
        
      } ,{
    "date": {
        "target_field": "@timestamp",
        "formats": [
            "MMdd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss.SSS",
            "yyyy/MM/dd HH:mm:ss.SSSSSS",
            "yyyy/MM/dd HH:mm:ss",
            "yyyy-MM-dd HH:mm:ss.SSS",
            "yyyy-MM-dd HH:mm:ss.SSS"
        ],
        "timezone": "Asia/Shanghai",
        "on_failure": [
            {
                "append": {
                    "field": "error.message",
                    "value": "{{ _ingest.on_failure_message }}"
                }
            }
        ],
        "ignore_failure": true,
        "field": "log.timestamp"
    }
}, {
        "set" : {
          "field" : "log.kind",
          "value" : "event"
        }
      },{
  "json" : {
    "field" : "log.msg",
    "target_field" : "event"
  }
} ,{
    "date": {
        "target_field": "event.event.firstTimestamp",
        "formats": [
          "date_time_no_millis"
        ],
        "timezone": "Asia/Shanghai",
        "on_failure": [
            {
                "append": {
                    "field": "error.message",
                    "value": "{{ _ingest.on_failure_message }}"
                }
            }
        ],
        "ignore_failure": false,
        "field": "event.event.firstTimestamp"
    }
}
    
  ]
}

```
traefik pipeline
```

PUT _ingest/pipeline/traefik_log_pipline
{
  "description" : "traefik log ",
    "processors": [
    {
  "json" : {
    "field" : "message",
    "target_field" : "log"
  }
} ,{
    "date": {
        "target_field": "@timestamp",
        "formats": [
           "date_time_no_millis"
        ],
        "timezone": "Asia/Shanghai",
        "on_failure": [
            {
                "append": {
                    "field": "error.message",
                    "value": "{{ _ingest.on_failure_message }}"
                }
            }
        ],
        "ignore_failure": true,
        "field": "log.time"
    }
}, {
        "set" : {
          "field" : "log.kind",
          "value" : "event"
        }
      }
    
  ]
}
 
```
