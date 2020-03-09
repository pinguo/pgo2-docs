# 常用方法(Util)

封装了许多常用方法，搜索slice，类型转换，字符串，结构体相关

## 方法列表
### conv
  * ToBool(v interface{}) bool // 转换为bool
  * ToInt(v interface{}) int // 转换为int
  * ToFloat(v interface{}) float64 // 转换为 float
  * ToString(v interface{}) string // 转换为string

### Map
  * MapMerge(a map[string]interface{}, m ...map[string]interface{}) // 合并map
    *  util.MapMerge(map[string][string],map[string][string])
  * MapGet(m map[string]interface{}, key string) interface{} // map获取字段
    * util.MapGet(map[string]map[string]string,"key1.key2")
  * MapSet(m map[string]interface{}, key string, val interface{}) // 设置map
    * util.MapGet(map[string]map[string]string,"key1.key2","v2")
### misc 
  * FormatLanguage(lang string) string // 格式化语言
  * FormatVersion(ver string, minDepth int) string // 格式化版本
  * VersionCompare(ver1, ver2 string) int // 版本比较
### slice
  * SliceSearchInt(a []int, x int) int // 搜索数字slice x在a中的位置
  * SliceSearchString(a []string, x string) int //  搜索字符串slice x在a中的位置
  * SliceUniqueInt(a []int) []int // int slice去除重复
  * SliceUniqueString(a []string) []string // 字符串slice 去除重复
### string 
  * IsAllDigit(s string) bool //是否纯数字
  * IsAllLetter(s string) bool // 是否纯字母
  * IsAllLower(s string) bool // 是否小写字母
  * IsAllUpper(s string) bool //是否大写字母
  * Md5String(v interface{}) string // md5字符串
### struct
  * STMergeSame(s1, s2 interface{}) // 合并相同类型的结构体
  * STMergeField(s1, s2 interface{}) // 合并相同类型字段的结构体

## 使用示例

```go
    util.方法名()
```

