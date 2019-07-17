#集群管理
##列出节点列表 		
	GET /_cat/nodes?v
#索引管理
##列出所有索引 		
	GET /_cat/indices?v
##创建索引 			
	PUT /customer?pretty
##删除索引 			
	DELETE /customer?pretty
#文档管理
##插入
	PUT /customer/external/1?pretty
	{
	       　　  "name": "John Doe"
	}
	POST /customer/external/1/_update?pretty
	{
	　 "doc": { "name": "Jane Doe", "age": 20 }
	}
##获取 			
	GET /customer/external/1?pretty
##批量修改
    POST /customer/external/_bulk?pretty
    {"index":{"_id":"1"}}
    {"name": "John Doe" }
    {"index":{"_id":"2"}}
    {"name": "Jane Doe" }  
##查询所有文档
GET /product_index/product/_search
#查询
##查询所有数据			
	GET /customer/_search?q=*&pretty
	GET /_search
	{
	    "query": {
	        "match_all": {}
	    }
	}
##查询英文名称为："Golden State Warriors" 的球队信息
	GET /nba/nba/_search
	{
	   "query": {
	        "match": {
	            "name_en": "Golden State Warriors"
	        }
	    }
	}
