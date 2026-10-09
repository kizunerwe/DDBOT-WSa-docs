# 持久化

DDBOT 使用 [buntdb](https://github.com/tidwall/buntdb) 作为嵌入式 key-value 数据库，文件 `.lsp.db`。`concern.StateManager` 继承了 `*localdb.ShortCut`，插件可直接用它的 KV 方法。

---

## 内置 KV 操作

`ShortCut` 提供了一组 helper，无需手写事务：

```go
// int64
s.SetInt64("myInt64", 123456)
v, _ := s.GetInt64("myInt64")  // v == 123456

// string
s.Set("myKey", "value")
v, _ := s.Get("myKey")

// JSON（任意结构）
s.SetJson("myStruct", &MyStruct{...})
var out MyStruct
err := s.GetJson("myStruct", &out)  // GetJson 只返回 error，结果写回第二个参数

// 带过期
s.Set("temp", "x", localdb.SetExpireOpt(time.Minute*10))
```

完整方法见 `lsp/buntdb/shortcut.go`。

---

## key 规范

所有插件共用一个 `.lsp.db`，key 需要避免与其他订阅源冲突。

内置订阅源并没有给 key 加 `site` 前缀，而是用不同的 key 名区分。key 生成集中在 `lsp/buntdb/key.go`，例如：

```go
func BilibiliUserInfoKey(keys ...interface{}) string {
    return NamedKey("UserInfo", keys)
}
func TwitterUserInfoKey(keys ...interface{}) string {
    return NamedKey("TwitterUserInfo", keys)
}
func WeiboUserInfoKey(keys ...interface{}) string {
    return NamedKey("WeiboUserInfo", keys)
}
```

`NamedKey` 用 `:` 拼各部分，所以 `BilibiliUserInfoKey(97505)` 是 `UserInfo:97505`，`TwitterUserInfoKey("elonmusk")` 是 `TwitterUserInfo:elonmusk`。

自定义插件可参考 `lsp/twitter/extraKey.go`，在自己的 `ExtraKey` 上转发一组 key 函数：

```go
// lsp/twitter/extraKey.go
type ExtraKey struct{}

func (e *ExtraKey) UserInfoKey(keys ...interface{}) string {
    return buntdb.TwitterUserInfoKey(keys...)
}
```

`StateManager` 用的 `KeySet`（订阅状态/配置的 key）可参考 `lsp/bilibili/keyset.go`。

!!! tip "避免 key 冲突"
    自定义 key 名要避开框架和内置订阅源已用的名字（如 `UserInfo`、`TwitterUserInfo`、`WeiboUserInfo`、`Permission`、`GroupPermission` 等，见 `lsp/buntdb/key.go`）。

---

## 事务

buntdb 支持串行事务。DDBOT 提供了 `RWCover` 包装读写事务：

```go
err := localdb.RWCover(func() error {
    s.Set("key1", "v1")
    s.Set("key2", "v2")
    return nil
})
```

同一时间只会存在一个可写事务，事务内的操作要么全部成功，要么全部回滚；可写事务在同一个 Goroutine 内可以嵌套（见 `lsp/buntdb/shortcut.go` 的注释）。耗时操作不要放进可写事务。

---

## key TTL

key 可设置过期时间：

```go
s.Set("temp", "x", localdb.SetExpireOpt(time.Minute*10))  // 10 分钟后过期
```

过期后 `Get` 返回 not found。适合存临时缓存、验证状态等。

---

## 迁移

DDBOT 有数据库版本迁移机制（`lsp/migrations.go`）：

```go
const LspVersionName = "lsp"
const LspSupportVersion int64 = 1

var lspMigrationMap = version.NewMigrationMapFromMap(
    map[int64]version.Migration{
        0: new(V1),
    },
)
```

启动时如果当前数据库版本低于 `LspSupportVersion`，会先把 `.lsp.db` 备份成 `<文件名>-<Unix 时间戳>`，再执行迁移（`lsp/module.go`）。

---

## 常用模式

### 缓存用户信息

```go
func (c *Concern) getUserInfo(id interface{}) (*UserInfo, error) {
    key := userInfoKey(id)
    var info UserInfo
    if err := c.StateManager.GetJson(key, &info); err == nil {
        return &info, nil
    }
    // 缓存未命中，请求接口
    newInfo, err := fetchUserInfo(id)
    if err != nil {
        return nil, err
    }
    c.StateManager.SetJson(key, newInfo, localdb.SetExpireOpt(time.Minute*30))
    return newInfo, nil
}
```

### 记录上次见到的状态

```go
func (c *Concern) checkLive(id interface{}) {
    key := lastSeenKey(groupCode, id)
    last, _ := c.GetInt64(key)

    current := fetchLiveStatus(id)
    if current != last {
        // 状态变化，产生 Event
        c.notify <- &LiveNotify{...}
        c.SetInt64(key, current)
    }
}
```

---

## 下一步

- [配置钩子](config-hook.md) - 自定义推送过滤
- [keyset 与注册](keyset-register.md) - 注册与 key 规范
- [完整范本](example-walkthrough.md)
