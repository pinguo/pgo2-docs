# Orm

关系数据库组件(Orm)是对GORM组件的封装，参见 <https://gorm.io/zh_CN/docs/index.html>，默认支持mysql，如其他数据库需要在mian.go里面引入:
```go
import "gorm.io/driver/sqlite"

func main(){
	 pgo2.App().Component("sqlite", orm.New, map[string]interface{}{"dialect":sqlite.Open})
    
	 pgo2.Run()
}
```

## 配置文件

```yaml
app.yaml
# 组件ID，db组件可以配置多个，只要分配唯一的组件ID, 通过
# GetObj(adapter.NewDb(), <id>)来获取非默认db组件
components:
    orm:
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
        # 最大连接维持时间(默认1小时)
        # maxConnTime: "1h"
        # 慢日志最小时间(默认100ms)
        # slowLogTime: "100ms"
# mysql config: https://gorm.io/zh_CN/docs/connecting_to_the_database.html#MySQL
        #  string 类型字段的默认长度
        # defaultStringSize:256
        #  禁用 datetime 精度，MySQL 5.6 之前的数据库不支持
        # disableDatetimePrecision:false
        # 重命名索引时采用删除并新建的方式，MySQL 5.7 之前的数据库和 MariaDB 不支持重命名索引
        # dontSupportRenameIndex:false
        # 用 `change` 重命名列，MySQL 8 之前的数据库和 MariaDB 不支持重命名列
        # dontSupportRenameColumn:false
        # 根据当前 MySQL 版本自动配置
        # skipInitializeWithVersion:false
# gorm config: https://gorm.io/zh_CN/docs/gorm_config.html
        # 为了确保数据一致性，GORM 会在事务里执行写入操作（创建、更新、删除）。如果没有这方面的要求，您可以在初始化时禁用它。
        # skipDefaultTransaction:false
        # PreparedStmt 在执行任何 SQL 时都会创建一个 prepared statement 并将其缓存，以提高后续的效率
        # prepareStmt:false
        # 生成 SQL 但不执行，可以用于准备或测试生成的 SQL
        # dryRun  :false
        # 启用全局 update/delete
        # allowGlobalUpdate :false
        #  使用单数表名，启用该选项，此时，`User` 的表名应该是 `user`
        # singularTable :false
        # 表名前缀，`User` 的表名应该是 `t_user`/`t_users`
        # tablePrefix :false
        # 在完成初始化后，GORM 会自动 ping 数据库以检查数据库的可用性，若要禁用该特性，可将其设置为 true
        # disableAutomaticPing :false
        # 在 AutoMigrate 或 CreateTable 时，GORM 会自动创建外键约束，若要禁用该特性，可将其设置为 true
        # disableForeignKeyConstraintWhenMigrating :false
# log config:https://gorm.io/zh_CN/docs/logger.html
        # logLevel:1  //  Silent:1 Error:2  Warn:3 Info:4
```
## 注意
  * gorm默认所有增删改都会用事务提交，速度大幅下降，需要关闭清配置 skipDefaultTransaction:true
  * 获取到adapter orm对象 如果断言是adapter.IOrm  将获取不到gorm.DB 的Callback() 方法

## 功能列表

```go
// 从对象池获取对象 
db:=t.GetObjBox(adapter.OrmClass,"dbmysql").(*adapter.Orm/adapter.IOrm) // (v0.1.131+)
// 普通方法列表
NewDb(ctr iface.IContext, componentId ...string) // 对象  this.GetObj(adapter.NewOrm(t.Context())).(*adapter.Orm/adapter.IOrm)

GetError() error // 返回错误
GetRowsAffected() int64 // 返回影响行数
GetStatement() *gorm.Statement // 返回gorm.Statement
GetConfig() *gorm.Config //   返回gorm.Config 
```
更多方法列表 参见原始的gorm:
<https://pkg.go.dev/gorm.io/gorm>
## 使用示例

```go

type OrmController struct {
    pgo2.Controller
}

// curl -v http://127.0.0.1:8000/orm/exec
// 使用db.Exec/db.ExecContext在主库上执行非查询操作，
// pgo默认使用10s超时，可通过context来自定义超时。
func (m *MysqlController) ActionExec() {
    // 获取db的上下文适配对象
    db:=t.GetObjBox(adapter.OrmClass).(adapter.IOrm)
    product := &Product{Name: t.Context().LogId(),Age:12}
    db.Create(&product)
    t.Context().PushLog("insertId",product.ID)

    result := &Product{}
    db.Model(&Product{}).Select("name").Where("id=?",product.ID).Take(&result)
    t.Context().PushLog("name",result.Name)

    t.JsonV2(nil,200)
}

type Product struct {
    ID           uint
    Name         string
    Age          uint8
}
```



