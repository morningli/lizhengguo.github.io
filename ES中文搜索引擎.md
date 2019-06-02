# ES中文搜索引擎

## 索引创建
	
	PUT /_template/template_reader
	{
	    "index_patterns": [
	        "reader_idx_*" //包含"reader_idx_"前缀的索引均会受到该模板影响
	    ],
	    "order": 2, //因为已经有了个优先级为1的模板
	    "settings": {
	            "analysis" : {
					"analyzer" : {
						"pinyin_analyzer" : {
							"tokenizer" : "my_pinyin"
						}
					},
					"tokenizer" : {
						"my_pinyin" : {
							"type" : "pinyin",
							"keep_first_letter": false, ////true：支持首字母  eg: 刘德华 -> [ldh]
							"keep_separate_first_letter" : false, //false：不支持首字母分隔 eg: 刘德华 -> [l,d,h]
							"keep_full_pinyin" : true, //true：支持全拼  eg: 刘德华 -> [liu,de,hua]
							"keep_joined_full_pinyin": true, //true：支持全拼  eg: 刘德华 -> [liudehua]
							"keep_original" : true,
							"limit_first_letter_length" : 16, //设置最大长度
							"lowercase" : true, //小写非中文字母
							"remove_duplicated_term" : true //重复的项将被删除,eg: 德的 -> de
						}
					}
				},
	            "number_of_shards": "5", //5个分片，多了影响搜索性能，少了影响容量，5个是官方建议值
	            "number_of_replicas": "2" //两个备份，相对较高的安全级别，用1个也没什么问题
	        }
	    },
	    "mappings": {
	        "_default_": {
	            "dynamic_templates": [
	                {
	                    "title": { //用于建立联想词的字段，同时也需要分词
	                        "match_mapping_type": "string",
	                        "match": "title", //该类型统一用title，用于区分其他字段
	                        "mapping": {
	                            "type": "text",
	                            "analyzer": "ik_smart",
	                            "search_analyzer": "ik_smart",
	                            "fields": {
	                                "keyword": {
	                                    "ignore_above": 256,
	                                    "type": "keyword"
	                                },
	                                "pinyin": {
	                                    "type": "text",
	                                    "analyzer": "pinyin_analyzer"
	                                },
	                                "suggest": {
	                                    "type": "completion",
	                                    "analyzer": "ik_smart"
	                                }
	                            }
	                        }
	                    }
	                },
	                {
	                    "info": { //每个索引存在一个字段用于长文
	                        "match_mapping_type": "string",
	                        "match": "info", //该类型统一用info，用于区分其他字段
	                        "mapping": {
	                            "type": "text",
	                            "analyzer": "ik_smart",
	                            "fields": {
	                                "pinyin": {
	                                    "type": "text",
	                                    "analyzer": "pinyin_analyzer"
	                                }
	                            }
	                        }
	                    }
	                },
	                {
	                    "strings": { //其他需要分词的字段
	                        "match_mapping_type": "string",
	                        "match": "*_text", //以"*_text"作为后缀
	                        "mapping": {
	                            "type": "text",
	                            "analyzer": "ik_smart",
	                            "fields": {
	                                "keyword": {
	                                    "ignore_above": 256,
	                                    "type": "keyword"
	                                },
	                                "pinyin": {
	                                    "type": "text",
	                                    "analyzer": "pinyin_analyzer"
	                                }
	                            }
	                        }
	                    }
	                }
	            ]
	        }
	    }
	}

## 搜索

	POST /reader_idx_book/_search
	{
	  "size": 20,
	  "query": {
	    "function_score": {
	      "query": {
	        "bool": {
	          "must": [
	            {
	              "multi_match": {
	                "query": "小",
	                "type": "best_fields",
	                "fields": [
	                  "title^1000000000",
	                  "author_name_text^100000000",
	                  "category_slave.keyword^10000000",
	                  "category3.keyword^1000000",
	                  "category_master.keyword^100000",
	                  "info^10000",
	                  "title.pinyin^1000",
	                  "author_name_text.pinyin^100",
	                  "info.pinyin^10",
	                  "tag"
	                ]
	              }
	            }
	          ],
	          "filter": [
	            {
	              "term": {
	                "shelf_status": "1"
	              }
	            },
	            {
	              "term": {
	                "category_master": "女"
	              }
	            }
	          ]
	        }
	      },
	      "script_score": {
	        "script": {
	          "lang": "expression",
	          "source": "_score * doc['allwords']/1000000"
	        }
	      }
	    }
	  },
	  "highlight": {
	    "fields": {
	      "title": {},
	      "title.pinyin": {},
	      "author_name_text": {},
	      "author_name_text.pinyin": {},
	      "category_slave.keyword": {},
	      "category3.keyword": {},
	      "category_master.keyword": {},
	      "info": {},
	      "info.pinyin": {},
	      "tag": {}
	    }
	  }
	}

## 直达区

	POST /_all/_search
	{
	  "size": 1,
	  "query": {
	    "bool": {
	      "filter": {
	        "term": {
	          "status": "1"
	        }
	      },"must": [
	        {"term": {
	          "title.keyword": {
	            "value": "万法唯心"
	          }
	        }}
	      ]
	    }
	  }
	}

## 联想词

	POST /reader_idx_*/_search
	{
	  "suggest": {
	    "song-suggest": {
	      "prefix": "太上",
	      "completion": {
	        "field": "title.suggest"
	      }
	    }
	  }
	}

## 问题分析

1. 使用_all搜索联想词没法查到所有的匹配项

		POST /reader_idx_*/_search
		{
		  "suggest": {"my-suggest":{"prefix":"小","completion":{"field":"title.suggest"}}}
		}

2. 单一索引上无法查到存在的项

		POST /reader_idx_book/_search
		{
		  "suggest": {"my-suggest":{"prefix":"小","completion":{"field":"title.suggest"}}}
		}

![](https://i.imgur.com/ZD1aDUK.png)

		POST /reader_idx_book/_search
		{
		  "suggest": {"my-suggest":{"prefix":"小小","completion":{"field":"title.suggest"}}}
		}

![](https://i.imgur.com/nEeoIEW.png)