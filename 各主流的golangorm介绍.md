# 各主流的golang orm介绍

[原文链接](https://studygolang.com/articles/13563)

当前较为主流/活跃的orm有gorm、xorm、gorose等

## xorm
- 文档
	- github
	- document
	- godoc
- 支持的数据库有：mysql、mymysql、postgres、tidb、sqlite、mssql、oracle
- 事务性支持
- 链式api

	    has, err := engine.Where("name = ?", name).Desc("id").Get(&user)
	    err := engine.Where(builder.NotIn("a", 1, 2).And(builder.In("b", "c", "d", "e"))).Find(&users)
- 支持原生sql操作
- 查询缓存
- 可根据数据库反转生成代码
- 级联加载
- 提供sql语句日志输出
- 支持批量查询处理

		// 每次处理100条
		// SELECT * FROM user Limit 0, 100
		// SELECT * FROM user Limit 101, 100
		err := engine.BufferSize(100).Iterate(&User{Name:name}, func(idx int, bean interface{}) error {
		    user := bean.(*User)
		    return nil
		})

- 自动化的读写分离/主从式

		dataSourceNameSlice := []string{masterDataSourceName, slave1DataSourceName, slave2DataSourceName}
		engineGroup, err := xorm.NewEngineGroup(driverName, dataSourceNameSlice)

## gorm
- 文档
	- github
	- gorm
- hook机制(Before/After Create/Save/Update/Delete/Find)
对象关系Has One, Has Many, Belongs To, Many To Many, Polymorphism
- 热加载
- 支持原生sql操作
- 事务性
- 链式api

		tx := db.Where("name = ?", "jinzhu").Where("age = ?", 20).Find(&users)

- 支持的数据库有：mysql、postgre、sqlite、sqlserver
- 查询操作
		
		// Get first record, order by primary key
		db.First(&user)
		//// SELECT * FROM users ORDER BY id LIMIT 1;
		
		// plain sql
		db.Where("name = ? AND age >= ?", "jinzhu", "22").Find(&users)
		// map
		db.Where(&User{Name: "jinzhu", Age: 20}).First(&user)
		//// SELECT * FROM users WHERE name = "jinzhu" AND age = 20 LIMIT 1;

## gorose
- 文档
	- github
	- document
- 支持的数据库有：mysql、postgres、sqlite、mssql、oracle
- 链式api
- 同时连接多个数据库和切换
- 支持原生sql操作
- 支持批量查询处理
- 事务性

		User.Fields("id, name").Where("id",">",2).Chunk(2, func(data []map[string]interface{}) {
		    // for _,item := range data {
		    //     fmt.Println(item)
		    // }
		    fmt.Println(data)
		})

## upper/db

同时支持nosql和sql的orm不多,这是其中之一 (另一个是beedb,已经四年没有更新了). upper/db对多种数据库进行封装,提供统一的接口进行CRUD.

- 文档
	- github
	- document or tour
- 支持的数据库有:PostgreSQL, MySQL, SQLite, MSSQL, QL and MongoDB.
- 不支持根据数据库类生成数据库表等DCL操作,只有DQL,DML
与大部分orm 框架相同,提供连接池
- 对RDBMS支持事务性

		sess, err := postgresql.Open(settings)
		if err != nil {
		    log.Fatalf("db.Open(): %q\n", err)
		}
		defer sess.Close()
		
		var books []Book
		err = sess.Collection("books").Find().All(&books)

## 总结
- 相似性
	- 各orm支持的数据库都基本相同（主流数据库都支持）
	- 支持事务性、链式查询等
- 差异
	- xorm、gorose支持批量查询处理
	- xorm支持主从式读写分离
	- gorm支持热加载
	- gorose便于在多个数据库切换
	- 文档全面性gorm>xorm>gorose