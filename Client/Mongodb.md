# Mongodb

Mongodb组件是对`https://github.com/qiniu/qmgo`的封装，更多文档请参见[qmgo](https://github.com/qiniu/qmgo)。
支持链式调用，validation tags 基于tag的字段验证,Hooks,事务
## 配置文件

```yaml
app.yaml
components:
    # 组件ID，默认为"mongodb"
    mongo:
        # mongo地址
        dsn: "mongodb://host1:port1/[db][?options]"
        # 连接超时，默认1秒
        # connectTimeout: "1s"
        # 读取超时，默认10秒
        # readTimeout: "10s"
        # 写入超时，默认10秒
        # writeTimeout: "10s"
```

其中`dsn`配置字段参考[mongodb URI Format](https://docs.mongodb.com/manual/reference/connection-string/)。默认的dsn参数：

| 参数           | 默认           | 说明                                       |
| -------------- | -------------- | ------------------------------------------ |
| replicaSet     | 空             | 复本集名称                                 |
| directConnection        | false     | 连接方式(true\|false),true-表示直连，false-表示自动发现               |
| maxPoolSize    | 100            | 连接池最大连接数                           |
| minPoolSize    | 1              | 连接池最小连接数                           |
| maxIdleTimeMS  | 300000         | 连接最大空闲毫秒时间                       |
| ssl            | false          | 是否通过ssl连接                            |
| j              | false          | 是否等待journal写入                        |
| wtimeoutMS     | 10000          | 写操作超时毫秒时间                         |
| readPreference | readPreference | 读取优先选项(readPreference\|primary\|...) |

## 功能列表
  * 带Ctx的方法 超时时间为配置的readTimeout writeTimeout ,默认为10s
  * 后缀带Ctx的方法 超时时间根据ctx参数

```go
// 从对象池获取对象
this.GetObjectBox(adapter.MongodbClass,db, coll)).(adapter.IMongodb)/(*adapter.Mongodb)
// 普通方法
// 插入1条数据 
InsertOne(doc interface{}, opts ...opts.InsertOneOptions) (result *qmgo.InsertOneResult, err error)
InsertOneCtx(ctx context.Context, doc interface{}, opts ...opts.InsertOneOptions) (result *qmgo.InsertOneResult, err error)
// 插入多条数据
InsertMany(docs interface{}, opts ...opts.InsertManyOptions) (result *qmgo.InsertManyResult, err error)
InsertManyCtx(ctx context.Context, docs interface{}, opts ...opts.InsertManyOptions) (result *qmgo.InsertManyResult, err error)
// 找到修改，没找到插入
Upsert(filter interface{}, replacement interface{}, opts ...opts.UpsertOptions) (result *qmgo.UpdateResult, err error)
UpsertCtx(ctx context.Context, filter interface{}, replacement interface{}, opts ...opts.UpsertOptions) (result *qmgo.UpdateResult, err error)
// 根据主键id 找到修改，没找到插入
UpsertId(id interface{}, replacement interface{}, opts ...opts.UpsertOptions) (result *qmgo.UpdateResult, err error)
UpsertIdCtx(ctx context.Context, id interface{}, replacement interface{}, opts ...opts.UpsertOptions) (result *qmgo.UpdateResult, err error)
// 修改一条数据
UpdateOne(filter interface{}, update interface{}, opts ...opts.UpdateOptions) (err error)
UpdateOneCtx(ctx context.Context, filter interface{}, update interface{}, opts ...opts.UpdateOptions) (err error)
// 根据主键id 修改一条数据
UpdateId(id interface{}, update interface{}, opts ...opts.UpdateOptions) (err error)
UpdateIdCtx(ctx context.Context, id interface{}, update interface{}, opts ...opts.UpdateOptions) (err error)
// 修改条件的所有数据
UpdateAll(filter interface{}, update interface{}, opts ...opts.UpdateOptions) (result *qmgo.UpdateResult, err error)
UpdateAllCtx(ctx context.Context, filter interface{}, update interface{}, opts ...opts.UpdateOptions) (result *qmgo.UpdateResult, err error)
// 根据filter替换文档
ReplaceOne(filter interface{}, doc interface{}, opts ...opts.ReplaceOptions) (err error)
ReplaceOneCtx(ctx context.Context, filter interface{}, doc interface{}, opts ...opts.ReplaceOptions) (err error)
// 根据filter删除1个文档
Remove(filter interface{}, opts ...opts.RemoveOptions) (err error)
RemoveCtx(ctx context.Context, filter interface{}, opts ...opts.RemoveOptions) (err error)
// 根据id删除1个文档
RemoveId(id interface{}, opts ...opts.RemoveOptions) (err error)
RemoveIdCtx(ctx context.Context, id interface{}, opts ...opts.RemoveOptions) (err error)
// 根据filter删除所有文档
RemoveAll(filter interface{}, opts ...opts.RemoveOptions) (result *qmgo.DeleteResult, err error)
RemoveAllCtx(ctx context.Context, filter interface{}, opts ...opts.RemoveOptions) (result *qmgo.DeleteResult, err error)
// 执行聚合命令
Aggregate(pipeline interface{}) qmgo.AggregateI
AggregateCtx(ctx context.Context, pipeline interface{}) qmgo.AggregateI
// 创建索引
EnsureIndexes(uniques []string, indexes []string) (err error)
EnsureIndexesCtx(ctx context.Context, uniques []string, indexes []string)(err error)
// 创建多个索引
CreateIndexes(indexes []opts.IndexModel) (err error)
CreateIndexesCtx(ctx context.Context, indexes []opts.IndexModel) (err error)
// 创建一个索引
CreateOneIndex(index opts.IndexModel) error
CreateOneIndexCtx(ctx context.Context, index opts.IndexModel) (err error)
// 删除所有索引
DropAllIndexes() (err error)
DropAllIndexesCtx(ctx context.Context) (err error)
// 删除索引
DropIndex(indexes []string) error
DropIndexCtx(ctx context.Context, indexes []string) error
// 删除表
DropCollection() error
DropCollectionCtx(ctx context.Context) error
// 获取表连接 不建议使用，会跳出封装
CloneCollection() (*mongo.Collection, error)
// 获取表名
GetCollectionName() string

// 获取client
GetClient() *mongodb.Client
// 事务相关连接
Session() (*qmgo.Session, error)
// 事务
DoTransaction(ctx context.Context, callback func(sessCtx context.Context) (interface{}, error)) (interface{}, error)

// 获取query对象
Find(filter interface{}, options ...opts.FindOptions) IMongodbQuery
FindCtx(ctx context.Context, filter interface{}, options ...opts.FindOptions) IMongodbQuery

// IMongodbQuery
// 设置排序字段
Sort(fields ...string) IMongodbQuery
// 选择字段
Select(selector interface{}) IMongodbQuery
// skip 数
Skip(n int64) IMongodbQuery
// limit 数
Limit(n int64) IMongodbQuery
// 获取1条记录信息
One(result interface{}) error
// 获取所有记录信息
All(result interface{}) error
// count
Count() (n int64, err error)
// distinct
Distinct(key string, result interface{}) error
// cursor
Cursor() CursorI
// findAndModify
Apply(change Change, result interface{}) error
// 指定索引
Hint(hint interface{}) IMongodbQuery

// IMongodbAggregate
// 
All(results interface{}) error
One(result interface{}) error
Iter() qmgo.CursorI
```

## 使用说明

```go


import (
	"fmt"

	"github.com/pinguo/pgo2"
	"github.com/pinguo/pgo2/adapter"
	"github.com/pinguo/pgo2/util"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
)

type Mongodb struct {
pgo2.Controller
}

// curl -v http://127.0.0.1:8000/mongodb/insert
func (m *Mongodb) ActionInsert() {
// 获取mongo的上下文适配对象
mongo := m.GetObj(adapter.NewMongodb("test", "test")).(*adapter.Mongodb)

// 通过map插入
doc1 := map[string]interface{}{"f1": "val1", "f2": true, "f3": 99}
ret,err := mongo.InsertOne(doc1)
fmt.Println("insert one doc1 id:",ret.InsertedID,"err", err)
// 通过bson.M插入

doc2 := bson.M{"f1": "val2", "f2": false, "f3": 10}
ret,err = mongo.InsertOne(doc2)
fmt.Println("insert one doc2 id:",ret.InsertedID,"err", err)

// 通过struct插入
doc3 := struct {
F1 string `bson:"f1"`
F2 bool   `bson:"f2"`
F3 int    `bson:"f3" validation:"gte:3"` // 自动验证是否大于等于3
}{"val3", false, 6}
ret,err = mongo.InsertOne(doc3)
fmt.Println("insert one  doc3 id:",ret.InsertedID,"err", err)

// 批量插入
docs := []interface{}{
bson.M{"f1": "val4", "f2": true, "f3": 7},
bson.M{"f1": "val5", "f2": false, "f3": 8},
bson.M{"f1": "val6", "f2": true, "f3": 9},
}
rets,err := mongo.InsertMany(docs)
fmt.Println("insert all docs ids:",rets.InsertedIDs,"err", err)
}

// curl -v http://127.0.0.1:8000/mongodb/update
func (m *Mongodb) ActionUpdate() {
// 对象池获取mongo的上下文适配对象
mongo := m.GetObjBox(adapter.MongodbClass, "test", "test").(*adapter.Mongodb)

// 更新单个文档
query := bson.M{"f1": "val1"}
update := bson.M{"$inc": bson.M{"f3": 2}}
err := mongo.UpdateOne(query, update)
fmt.Println("update one f1==val1:", err)

// 更新多个文档
query = bson.M{"f3": bson.M{"$gte": 7}}
update = bson.M{"$set": bson.M{"f2": false}}
result,err := mongo.UpdateAll(query, update)
fmt.Println("update all f3>=7: update count :",result.ModifiedCount,"err", err)

// 更新或插入
query = bson.M{"f1": "val10"}
update = bson.M{"f3": 2}
	result,err = mongo.Upsert(query, update)
fmt.Println("update or insert f1==val10:",util.ToString(result),"err", err)
}

// curl -v http://127.0.0.1:8000/mongodb/query
func (m *Mongodb) ActionQuery() {
// 获取mongo的上下文适配对象
mongo := m.GetObj(adapter.NewMongodb("test", "test")).(*adapter.Mongodb)

// 查询单个文档(未指定结果类型，结果为bson.M)
var v1 interface{}
err := mongo.Find(bson.M{"f1": "val1"}).Select(bson.M{"f1": 1}).One( &v1)
fmt.Println("query one f1==val1:", v1, err)

// 查询单个文档(结果类型为map)
var v2 map[string]interface{}
err = mongo.Find(bson.M{"f1": "val2"}).One(&v2)
fmt.Println("query one f1==val2:", v2, err)

// 查询单个文档(结果类型为struct)
var v3 struct {
Id primitive.ObjectID `bson:"_id"`
F1 string        `bson:"f1"`
F2 bool          `bson:"f2"`
F3 int           `bson:"f3"`
}
err = mongo.Find(bson.M{"f1": "val3"}).One(&v3)
fmt.Println("query one f1==val3:", v3, err)

// 查询多个文档(指定结果为map)
var docs []map[string]interface{}
err = mongo.Find(bson.M{"f3": bson.M{"$gte": 6}}).Sort("-_id").All( &docs)
fmt.Println("query all f3>=6:", docs, err)
}
```

