# MaxMind

MaxMind组件封装了`github.com/oschwald/maxminddb-golang`的GEO查询功能，能通过IP来确定国家、省市等地理信息。geo数据库文件需要用户自行获取(获取免费或收费的geo数据文件)。

GEO查询结果定义：

```go
// Geo lookup result
type Geo struct {
    Code      string `json:"code"`               // country/area code
    Continent string `json:"-"`                  // continent name (en)
    Country   string `json:"country,omitempty"`  // country/area name (en)
    Province  string `json:"province,omitempty"` // province name (en)
    City      string `json:"city,omitempty"`     // city name(en)

    // i18n name, default is en
    I18n struct {
        Continent string
        Country   string
        Province  string
        City      string
    } `json:"-"`
}
```

## 功能列表

```go
// 从对象池获取对象
this.GetObjBox(adapter.MaxMindClass).(adapter.IMaxMind)/(*adapter.MaxMind) // (v0.1.131+)
// 普通方法
NewMaxMind(componentId ...string) // 对象 this.GetObj(adapter.NewMaxMind()).(adapter.IMaxMind)/(*adapter.MaxMind)
mmd.GeoByIp()	// 根据IP查询GEO信息


```

## 配置文件

```yaml
app.yaml

components:
    # 组件ID，默认maxMind，国家、城市数据文件至少需要设置一个
    maxMind: 
        # 国家数据文件
        countryFile: "@app/../geoip/GeoIP2-Country.mmdb"
        # 城市数据文件
        cityFile:    "@app/../geoip/GeoIP2-City.mmdb"
        
                
```

## 使用示例

```go

type MaxMindController struct {
    pgo2.Controller
}

// curl -v http://127.0.0.1:8000/max-mind/geo-by-ip
func (m *MaxMindController) ActionGeoByIp() {
    // 获取要解析的IP地址
    ip := m.Context().ValidateParam("ip", "182.150.28.13").Do()

    // 获取MaxMind的上下文件适配对象
    // mmd := m.GetObj(adapter.NewMaxMind()).(*adapter.MaxMind)
    // 从对象池获取MaxMind的上下文件适配对象
    mmd := m.GetObjPool(adapter.MaxMindClass, adapter.NewMaxMindPool).(*adapter.MaxMind)

    // 解析IP的geo信息
    geo := mmd.GeoByIp(ip)

    m.Json(geo, http.StatusOK)
}

```

