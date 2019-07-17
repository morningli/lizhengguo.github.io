# MongoDB官方推出的Go驱动库“mongo-go-driver”快速教程

[原文链接](https://zh.shellman.me/articles/mongo-go-driver-demo/)

## 0. 关于“mongo-go-driver”

MongoDB一直缺乏官方的Go驱动库，官方仅做了社区驱动的推荐：Community Supported Drivers Reference。这其中Go驱动推荐的是mgo，以前mgo维护者是个人，后来因为精力的原因将其移交给社区。

mgo我用过相当长的一段时间，这是一个非常好用的驱动，基本可以满足我的需求。它最大的问题就是更新不够及时，总是落后官方一到两个版本。移交给社区后有一定的改善，但是依然存在类似的问题。比如说在今年的6月份MongoDB 4.0的正式版已经发布了，但是到目前为止半年过去了，mgo对4.0的支持依然是试验性的。

官方的驱动一般都会更新的更及时，新版本带来的特性也会支持的更好。而在今年初，MongoDB官方公开了Go的官方驱动“mongo-go-driver”的Alpha 1版本。到目前为止已经是Alpha 18了，虽然已经迭代了18个版本，但是依然不建议在生产环境使用。不过这不妨碍我们试用它，并在未来恰当的时机导入到生产环境中。

## 1. 演示

### 1.1. 映射

	type User struct {
		ID    objectid.ObjectID "_id,omitempty"
		Name  string	`bson:"dbname",json:"jsonname"`
		Phone string
	}

有三种规则：

1. 什么都不写，就像上面的Phone一样，这种的话就默认属性名全小写作为key。
1. 第二种就是像ID一样简写bson映射。
1. 第三种是像Name一样完整的写出来，这样的话可以同时做bson和json的映射。

Key规则：
必须作为第一个参数，但是可以不写。如果这个字段不想映射到bson的话，可以这样`bson:"-"`

其它参数：

- omitempty —— 允许值为空，比如说在ID中使用后，MongoDB会自动为_id生成ObjectId
- minsize —— 如果在保留数值的情况下可行，将MongoDB大于32位的整数值解析为int32。
- truncate —— 如果在损失精度的情况下，将MongoDB的double解析为float32。
- inline —— 可以让内联struct和外联效果一样

### 1.2. 连接MongoDB

	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	
	client, err := mongo.Connect(ctx, "mongodb://127.0.0.1")
	if err != nil {
		log.Fatal(err)
	}

官方库需要配合context.Context使用，这样可以让用户能够更好的在并发中控制。

### 1.3. DB、Collection

	db := client.Database("demo")
	userColl := db.Collection("user")

### 1.4. 新增

	// Insert one
	if result, err := userColl.InsertOne(ctx, User{Name: "UserName", Phone: "1234567890"}); err == nil {
		log.Println(result)
	} else {
		log.Fatal(err)
	}
	
	// Insert many
	{
		users := []interface{}{
			User{Name: "UserName_0", Phone: "123"},
			User{Name: "UserName_1", Phone: "456"},
			User{Name: "UserName_2", Phone: "789"},
		}
		if result, err := userColl.InsertMany(ctx, users); err == nil {
			log.Println(result)
		} else {
			log.Fatal(err)
		}
	}

### 1.5. 查询

	// Find one
	{
		result := userColl.FindOne(ctx, bson.M{"phone": "1234567890"})
		var user User
		if err := result.Decode(&user); err != nil {
			log.Fatal(err)
		}
		log.Println(user)
	}
	
	// Find
	if cur, err := userColl.Find(ctx, bson.M{"phone": primitive.Regex{Pattern: "456", Options: ""}}); err == nil {
		defer cur.Close(ctx)
		for cur.Next(ctx) {
			var user User
			if err := cur.Decode(&user); err != nil {
				log.Fatal(err)
			}
			log.Println(user)
		}
	} else {
		log.Fatal(err)
	}

### 1.6. 修改

	// Update one
	if result, err := userColl.UpdateOne(
		ctx, bson.M{"phone": "123"},
		bson.M{"$set": bson.M{"dbname": "UserName_changed"}}); err == nil {
		log.Println(result)
	} else {
		log.Fatal(err)
	}
	
	// Update many
	if result, err := userColl.UpdateMany(
		ctx, bson.M{"phone": primitive.Regex{Pattern: "456", Options: ""}},
		bson.M{"$set": bson.M{"dbname": "UserName_changed"}}); err == nil {
		log.Println(result)
	} else {
		log.Fatal(err)
	}
	
	// Replace one
	{
		user := User{Name: "UserName_2_replaced", Phone: "789"}
		if result, err := userColl.ReplaceOne(ctx, bson.M{"phone": "789"}, user); err == nil {
			log.Println(result)
		} else {
			log.Fatal(err)
		}
	}

Update是用来直接修改数据库的，ReplaceOne则是整个替换。

### 1.7. 删除

	// Delete one
	if result, err := userColl.DeleteOne(ctx, bson.M{"phone": "123"}); err == nil {
		log.Println(result)
	} else {
		log.Fatal(err)
	}
	
	// Delete many
	if result, err := userColl.DeleteMany(ctx, bson.M{"phone": primitive.Regex{Pattern: "456", Options: ""}}); err == nil {
		log.Println(result)
	} else {
		log.Fatal(err)
	}

## 2. 总结

使用过mgo的朋友肯定能发现，CRUD相关的代码和mgo大同小异。总的来说我感觉mongo-go-driver封装的更加底层一些，不如mgo方便。但是我想这是因为官方为了顾及到各种使用场景，更底层的库可以让用户拥有更多的控制权。我们在使用的时候只需要根据自身的场景适度封装，即可方便的使用了。