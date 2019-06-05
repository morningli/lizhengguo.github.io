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
	          "tokenizer": "ik_max_word",
	          "filter": [
	            "pinyin_filter",
	            "word_delimiter"
	          ]
	        },
	        "pinyin_analyzer_smart": {
	          "tokenizer": "ik_smart",
	          "filter": [
	            "pinyin_filter",
	            "word_delimiter"
	          ]
	        }
	      },
	      "filter": {
	        "pinyin_filter": {
	          "type": "pinyin",
	          "keep_first_letter": false,
	          "keep_separate_first_letter": false,
	          "keep_full_pinyin": true,
	          "keep_joined_full_pinyin": true,
	          "keep_original": false,
	          "limit_first_letter_length": 16,
	          "lowercase": true,
	          "remove_duplicated_term": true
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
	                  "analyzer": "pinyin_analyzer",
	                  "search_analyzer": "pinyin_analyzer_smart"
	                },
	                "suggest": {
	                  "type": "completion",
	                  "analyzer": "ik_max_word",
	                  "search_analyzer": "ik_smart"
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
	                  "analyzer": "pinyin_analyzer",
	                  "search_analyzer": "pinyin_analyzer_smart"
	                  
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
	                  "analyzer": "pinyin_analyzer",
	                  "search_analyzer": "pinyin_analyzer_smart"
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

### 1.使用拼音无法获得高亮位置

描述：

![图片描述](resource/problem_highlight.png)

参考：[Day 2 - ES 6.x拼音分词高亮爬坑记](https://elasticsearch.cn/article/6166)

解决：因为pinyin bug导致无法获取高亮位置，仅使用pinyin作为filter可以解决。文章推荐的使用ngram作为分词器索引和查询的性能极差，这里改成了使用ik作为分词器。
	
	    "analysis": {
	      "analyzer": {
	        "pinyin_analyzer": {
	          "tokenizer": "ik_max_word",
	          "filter": [
	            "pinyin_filter",
	            "word_delimiter"
	          ]
	        }
	      },
	      "filter": {
	        "pinyin_filter": {
	          "type": "pinyin",
	          "keep_first_letter": false,
	          "keep_separate_first_letter": false,
	          "keep_full_pinyin": true,
	          "keep_joined_full_pinyin": true,
	          "keep_original": false,
	          "limit_first_letter_length": 16,
	          "lowercase": true,
	          "remove_duplicated_term": true
	        }
	      }
	    },

# 2.使用complete suggestion查询reader_idx_*时无法得到所有结果

描述:

	GET reader_idx_*/_search
	{
	  "_source": "false",
	  "suggest": {
	    "topics_suggest": {
	      "completion": {
	        "field": "title.suggest"
	      },
	      "prefix": "小"
	    }
	  }
	}

![图片描述](resource/problem_suggest1.png)

	GET reader_idx_book/_search
	{
	  "_source": "false",
	  "suggest": {
	    "topics_suggest": {
	      "completion": {
	        "field": "title.suggest"
	      },
	      "prefix": "小"
	    }
	  }
	}

![图片描述](resource/problem_suggest2.png)

解决：suggest数量限制，可控，加上size字段即可

	GET reader_idx_*/_search
	{
	  "_source": "false",
	  "suggest": {
	    "topics_suggest": {
	      "completion": {
	        "field": "title.suggest",
	        "size": 10
	      },
	      "prefix": "小"
	    }
	  }
	}

# 3. 使用搜索查询联想词时查出不相关数据
问题描述：
```
GET /reader_idx_*/_search
{
  "query": {
    "function_score": {
      "functions": [
        {
          "script_score": {
            "script": {
              "source": "_score * 1"
            }
          }
        }
      ],
      "query": {
        "bool": {
          "filter": {
            "term": {
              "status": "1"
            }
          },
          "must": {
            "multi_match": {
              "fields": [
                "title^10",
                "title.pinyin^1"
              ],
              "query": "黄泉",
              "tie_breaker": 0,
              "type": "best_fields"
            }
          }
        }
      }
    }
  }
}
```
![图片描述](resource/problem_pinyin1.png)

分析：输入会分别转换成中文分词和拼音分词在title及title.pinyin字段上进行分词，分词情况如下

```
GET /read_idx_category/_analyze
{
  "field": "title",
  "text": "黄泉"
}
```

![图片描述](resource/problem_pinyin2.png)

```
GET /read_idx_category/_analyze
{
  "field": "title.pinyin",
  "text": "黄泉"
}
```

![图片描述](resource/problem_pinyin3.png)

可以看到拼音分词的粒度更细，而且因为是拼音匹配，会匹配到同音字，显得搜索结果不大相关。拼音分词比中文分词更细的原因是中文分词建立索引使用了ik_max，搜索时分词使用ik_smart，拼音没有指定搜索分词器，直接使用了建立索引的更细的分词器。这里应该改成拼音搜索时采用更粗粒度的分词器。