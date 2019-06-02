# ES中文搜索引擎

## 索引创建
	
	PUT /_template/template_reader
	{
	  "index_patterns": [
	    "reader_idx_*"
	  ],
	  "order": 2,
	  "settings": {
	    "analysis": {
	      "analyzer": {
	        "pinyin_analyzer": {
	          "tokenizer": "my_pinyin"
	        }
	      },
	      "tokenizer": {
	        "my_pinyin": {
	          "type": "pinyin",
	          "keep_first_letter": false,
	          "keep_separate_first_letter": false,
	          "keep_full_pinyin": true,
	          "keep_joined_full_pinyin": true,
	          "keep_original": true,
	          "limit_first_letter_length": 16,
	          "lowercase": true,
	          "remove_duplicated_term": true,
	          "ignore_pinyin_offset": false
	        }
	      }
	    },
	    "number_of_shards": "5",
	    "number_of_replicas": "1"
	  },
	  "mappings": {
	    "_default_": {
	      "dynamic_templates": [
	        {
	          "title": {
	            "match_mapping_type": "string",
	            "match": "title",
	            "mapping": {
	              "type": "text",
	              "analyzer": "ik_max_word",
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
	                  "analyzer": "ik_max_word"
	                }
	              }
	            }
	          }
	        },
	        {
	          "info": {
	            "match_mapping_type": "string",
	            "match": "info",
	            "mapping": {
	              "type": "text",
	              "analyzer": "ik_max_word",
				  "search_analyzer": "ik_smart",
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
	          "strings": {
	            "match_mapping_type": "string",
	            "match": "*_text",
	            "mapping": {
	              "type": "text",
	              "analyzer": "ik_max_word",
				  "search_analyzer": "ik_smart",
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
	      "*": {}
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

### 搜索

1. 使用拼音无法获得高亮位置

### 联想词

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

3. 使用fuzzy关联到莫名其妙的结果
	
		GET reader_idx_*/_search
		{
		  "_source": "false", 
		  "suggest": {
		    "topics_suggest": {
		      "completion": {
		        "field": "title.suggest",
		        "fuzzy":{"fuzziness":2}
		      },
		      "prefix": "小"
		    }
		  }
		}

	![](https://i.imgur.com/EOBT59t.png)

		GET reader_idx_*/_search
		{
		  "_source": "false", 
		  "suggest": {
		    "topics_suggest": {
		      "completion": {
		        "field": "title.suggest",
		        "fuzzy":{"fuzziness":2}
		      },
		      "prefix": "小小"
		    }
		  }
		}

	![](https://i.imgur.com/fTBiLeE.png)

		GET reader_idx_*/_search
		{
		  "_source": "false", 
		  "suggest": {
		    "topics_suggest": {
		      "completion": {
		        "field": "title.suggest",
		        "fuzzy":true
		      },
		      "prefix": "小"
		    }
		  }
		}

	![](https://i.imgur.com/fwhGqyl.png)

		GET reader_idx_*/_search
		{
		  "_source": "false", 
		  "suggest": {
		    "topics_suggest": {
		      "completion": {
		        "field": "title.suggest",
		        "fuzzy":true
		      },
		      "prefix": "小小"
		    }
		  }
		}

	![](https://i.imgur.com/8vPac8T.png)