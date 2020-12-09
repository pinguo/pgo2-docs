# Db

关系数据库组件(DB)是对GO内置的`database/sql`包的组件封装，需要和具体的数据库驱动一起使用，驱动列表参见 <https://golang.org/s/sqldrivers>，例如，使用mysql需在程序的main入口中导入mysql的驱动包：
```go
import _ "github.com/go-sql-driver/mysql"
```

Db组件支持读写分离，`SELECT`语句默认在从库中执行，其余语句在主库中执行，支持慢日志记录。可以通过SetMaster设置.

## 配置文件

```yaml
app.yaml
# 组件ID，db组件可以配置多个，只要分配唯一的组件ID, 通过
# GetObj(adapter.NewDb(), <id>)来获取非默认db组件
components:
    db:
        # 驱动名(需导入指定的驱动包)
        driver: "mysql"
        # 主库地址(参见各驱动包说明)
        dsn: "user:pass@tcp(127.0.0.1:3306)/db?charset=utf8&timeout=0.5s"
        # 从库地址(可选，默认空)
        # slaves: ["slave1 dsn", "slave2 dsn"]
        # 最大空闲连接数(默认5)
        # maxIdleConn: 5
        # 数据库建立连接的最大数目(默认0 无限制)
        # maxOpenConn: 10
        # 是完整的记录sql日志，且记录耗时(默认false 只记录执行方法和耗时)
        # sqlLog:true
        # 最大连接维持时间(默认1小时)
        # maxConnTime: "1h"
        # 慢日志最小时间(默认100ms)
        # slowLogTime: "100ms"
```

## 功能列表

```go
// 从对象池获取对象 
this.GetObjBox(adapter.DbClass).(adapter.IDb)/(*adapter.Db) // (v0.1.131+)
// 普通方法列表
NewDb(componentId ...string) // 对象  this.GetObj(adapter.NewDb()).(adapter.IDb)/(*adapter.Db)
db.SetMaster(v bool) // 设置查询操作是否从master
db.Begin(opts ...*sql.TxOptions) ITx // 开始一个新事务,并返回一次性的事务对象(默认超时上下文) 
db.BeginContext(ctx context.Context, opts *sql.TxOptions) ITx // 开始一个新事务，并返回一次性的事务对象(指定上下文)
db.QueryOne(query string, args ...interface{}) IRow //  查询单个文档(默认超时上下文)
db.QueryOneContext(ctx context.Context, query string, args ...interface{}) IRow // 查询单个文档(指定上下文)
db.Query(query string, args ...interface{}) *sql.Rows // 查询多个文档(默认超时上下文)
db.QueryContext(ctx context.Context, query string, args ...interface{}) *sql.Rows // 查询多个文档(指定上下文)
db.Exec(query string, args ...interface{}) sql.Result // 非查询操作(默认超时上下文)
db.ExecContext(ctx context.Context, query string, args ...interface{}) sql.Result // 非查询操作(指定上下文)
db.PrepareSql(query string) IStmt // 批量操作，并返回一个状态对象(默认超时上下文)
db.PrepareContext(ctx context.Context, query string) IStmt // 批量操作，并返回一个状态对象(指定上下文)

// 事务相关
tx.Commit() bool // 提交一个事务
tx.Rollback() bool // 回滚一个事务
tx.QueryOne(query string, args ...interface{}) IRow // 查询单个文档(默认超时上下文)
tx.QueryOneContext(ctx context.Context, query string, args ...interface{}) IRow // 查询单个文档(指定上下文)
tx.Query(query string, args ...interface{}) *sql.Rows // 查询多个文档(默认超时上下文)
tx.QueryContext(ctx context.Context, query string, args ...interface{}) *sql.Rows // 查询多个文档(指定上下文)
tx.Exec(query string, args ...interface{}) sql.Result // 非查询操作(默认超时上下文)
tx.ExecContext(ctx context.Context, query string, args ...interface{}) sql.Result // 非查询操作(指定上下文)
tx.PrepareSql(query string) IStmt  // 批量操作，并返回一个状态对象(默认超时上下文)
tx.PrepareContext(ctx context.Context, query string) IStmt // 批量操作，并返回一个状态对象(指定上下文)

// 单列相关
row.Scan(dest ...interface{}) error // 获取单列字段值

// 状态
stmp.Close() // 关闭状态
stmp.QueryOne(args ...interface{}) IRow // 查询单个文档(默认超时上下文)
stmp.QueryOneContext(ctx context.Context, args ...interface{}) IRow // 查询单个文档(指定上下文)
stmp.Query(args ...interface{}) *sql.Rows // 查询多个文档(指定上下文)
stmp.QueryContext(ctx context.Context, args ...interface{}) *sql.Rows  // 查询多个文档(指定上下文)
stmp.Exec(args ...interface{}) sql.Result // 非查询操作(默认超时上下文)
stmp.ExecContext(ctx context.Context, args ...interface{}) sql.Result // 非查询操作(指定上下文)
```

## 使用示例

```go

type MysqlController struct {
    pgo2.Controller
}

// curl -v http://127.0.0.1:8000/mysql/exec
// 使用db.Exec/db.ExecContext在主库上执行非查询操作，
// pgo默认使用10s超时，可通过context来自定义超时。
func (m *MysqlController) ActionExec() {
    // 获取db的上下文适配对象
    db := m.GetObj(adapter.NewDb()).(*adapter.Db)

    // 执行插入操作
    age := (time.Now().Nanosecond() / 1000) % 100
    name := fmt.Sprintf("name%d", age)
    query := "INSERT INTO test1 (name, age) VALUES (?, ?)"

    res := db.Exec(query, name, age)
    if res == nil {
        panic("exec insert failed, " + query)
    }

    lastId, _ := res.LastInsertId()
    numRow, _ := res.RowsAffected()
    fmt.Println("insert: ", lastId, numRow)

    // 执行修改操作
    query = "UPDATE test1 SET age=age+1 WHERE id=?"
    res = db.Exec(query, lastId)
    if res == nil {
        panic("exec update failed, " + query)
    }

    lastId1, _ := res.LastInsertId()
    numRow1, _ := res.RowsAffected()
    fmt.Println("update: ", lastId1, numRow1)

    // 执行删除操作
    query = "DELETE FROM test1 WHERE id=?"
    res = db.Exec(query, lastId)
    if res == nil {
        panic("exec delete failed, " + query)
    }

    lastId2, _ := res.LastInsertId()
    numRow2, _ := res.RowsAffected()
    fmt.Println("delete: ", lastId2, numRow2)
}

// curl -v http://127.0.0.1:8000/mysql/query
// 使用db.Query/QueryOne/QueryContext/QueryOneContext来查询数据，
// 若当前db对象未执行过任何操作，则使用从库进行查询，否则复用上一次获取的db连接。
func (m *MysqlController) ActionQuery() {

    // 对象池获取db的上下文适配对象
    db := m.GetObjBox(adapter.DbClass).(*adapter.Db)

    // 执行插入操作
    age := (time.Now().Nanosecond() / 1000) % 100
    name := fmt.Sprintf("name%d", age)
    query := "INSERT INTO test1 (name, age) VALUES (?, ?)"

    db.Exec(query, name, age)

    // 查询单条数据
    id, name, age := 0, "", 0
    query = "SELECT id, name, age FROM test1 WHERE id=?"
    err := db.QueryOne(query, 3).Scan(&id, &name, &age)
    if err != nil {
        fmt.Println("query one failed, " + err.Error())
    } else {
        fmt.Println("query one for id=3, ", id, name, age)
    }

    // 查询多条数据
    query = "SELECT id, name, age FROM test1 WHERE age>?"
    rows := db.Query(query, 10)
    if rows == nil {
        panic("query failed, query: " + query)
    }

    defer rows.Close()
    for rows.Next() {
        id, name, age := 0, "", 0
        if err := rows.Scan(&id, &name, &age); err != nil {
            panic("query scan faild, err: " + err.Error())
        }
        fmt.Println("user: ", id, name, age)
    }
}

// curl -v http://127.0.0.1:8000/mysql/prepare
// 使用db.PrepareSql/db.PrepareContext来执行批量操作，默认查询操作在
// 从库上进行，其余操作在主库上进行，若当前db对象有过其它操作，则查询
// 操作会复用之前的连接。
func (m *MysqlController) ActionPrepareSql() {
    // 获取db的上下文适配对象
    db := m.GetObj(adapter.NewDb()).(*adapter.Db)

    query := "INSERT INTO test1 (name, age) VALUES (?, ?)"
    stmt := db.PrepareSql(query)
    defer stmt.Close()

    for i := 0; i < 10; i++ {
        name := fmt.Sprintf("name%d", i)
        res := stmt.Exec(name, i)
        if res == nil {
            panic("stmt exec failed, " + query)
        }
    }
}

// curl -v http://127.0.0.1:8000/mysql/transaction
// 使用db.Begin/BeginContext/Commit/Rollback来进行事务操作
func (m *MysqlController) ActionTransaction() {
    // 获取db的上下文适配对象
    db := m.GetObj(adapter.NewDb()).(*adapter.Db)

    tx := db.Begin()
    defer func() {
        if err:=recover();err!=nil{
            tx.Rollback()
        }
    }()
    tx.Exec("INSERT INTO test1 (name, age) VALUES (?, ?)", "name1", 1)
    tx.Exec("UPDATE test1 SET age=age+1 WHERE id=?", 1)
    if tx.Commit() == false{
        tx.Rollback()
    }else {
        fmt.Println("Transaction ok")
    }

}
```



