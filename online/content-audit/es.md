```
PUT content-audit-operation-record
 {
   
    "mappings" : {
      "properties" : {
        "@timestamp" : {
          "type" : "date"
        },
        "@version" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "step" : {
          "type" : "long"
        }, "business_code" : {
          "type" : "keyword"
        },
        "business_data_id" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "status" : {
          "type" : "long"
        },
        "admin_uid" : {
          "type" : "long"
        } ,
        "sso_id" : {
         "type" : "keyword"
        },"sso_log_id" : {
          "type" : "keyword"
        },
        "admin_username" : {
           "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        },
        "create_time" : {
          "type" : "date",
          "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
           
        }
      }
    } 
}

PUT _template/content-audit
{
  "index_patterns": ["content-audit-*"],
  "settings": {
    "index": {
      "number_of_shards": "3",
      "number_of_replicas": "1"
    }
  },
  "mappings": {
    "properties": {
      "create_time": {
        "type": "date",
        "format": "strict_date_optional_time||epoch_millis||yyyy-MM-dd HH:mm:ss"
      }, "business_code" : {
          "type" : "keyword"
        }, "sso_id" : {
          "type" : "keyword"
        }, "sso_log_id" : {
          "type" : "keyword"
        }, "suggestion" : {
           "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }, "business_data_id" : {
           "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }, "keywords" : {
           "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }, "label" : {
           "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }, "content" : {
           "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
    }
  },
  "aliases": {}
}
```


