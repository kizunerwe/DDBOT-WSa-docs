# 持久化

DDBOT 使用 [buntdb](https://github.com/tidwall/buntdb) 作为嵌入式 key-value 数据库，文件 `.lsp.db`。`StateManager` 已内置 KV 操作，插件可直接用。

---

## 内置 KV 操作

`StateManager` 提供了一组 helper，无需手写事务：

```go
// int64
s.SetInt64("myInt64", 123456)
v, _ := s.GetInt64("myInt64")  // v == 123456

// string
s.Set("myKey", "value")
v, _ := s.Get("myKey")

// JSON（任意结构）
s.SetJson("myStruct", &MyStruct{...})
v, _ := s.GetJson("myStruct")

// 带过期
s.Set("temp", "x", localdb.SetExpireOpt(time.Minute*10))
```

完整方法见 `lsp/buntdb/shortcut.go`。

---

## keyset 定义

为避免 key 冲突，每个插件应定义自己的 key 前缀。参考 `lsp/bilibili/keyset.go`、`lsp/twitter/extraKey.go`：

```go
package example

// keyset.go
const (
    keyPrefix = "example"

    KeyUserInfo = keyPrefix + ":userinfo:"  // + id
    KeyLastSeen = keyPrefix + ":lastseen:"  // + groupCode + id
)

// 封装
func userInfoKey(id interface{}) string {
    return KeyUserInfo + fmt.Sprintf("%v", id)
}
```

!!! tip "key 一定要带前缀"
    所有插件共用一个 `.lsp.db`，key 不带前缀会和其他插件冲突。

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

事务内的操作要么全部成功，要么全部回滚。

---

## key TTL

key 可设置过期时间：

```go
s.Set("temp", "x", localdb.SetExpireOpt(time.Minute*10))  // 10 分钟后过期
```

过期后 `Get` 返回 not found。适合存临时缓存、验证状态等。

---

## 嵌套事务

DDBOT 重构 buntdb 支持嵌套事务。例如被禁言时不会尝试推送：

```go
// 框架内部已处理，插件通常不需要关心
```

---

## 迁移

DDBOT 有数据库版本迁移机制。如果你的插件需要随版本变更数据结构，可注册迁移函数：

```go
// 见 lsp/migrations.go、lsp/migration_v1.go
var lspMigrationMap = map[int64]version.MigrationFunc{
    1: migrationV1,
    2: migrationV2,
}
```

启动时框架会检查当前版本，按需执行迁移并自动备份。

---

## 常用模式

### 缓存用户信息

```go
func (c *Concern) getUserInfo(id interface{}) (*UserInfo, error) {
    key := userInfoKey(id)
    var info UserInfo
    if v, err := c.StateManager.GetJson(key, &info); err == nil && v != "" {
        return &info, nil
    }
    // 缓存未命中，请求接口
    info, err = fetchUserInfo(id)
    if err != nil {
        return nil, err
    }
    c.StateManager.SetJson(key, info, localdb.SetExpireOpt(time.Minute*30))
    return info, nil
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

## buntdb-cli 运维

[buntdb-cli](https://github.com/Sora233/buntdb-cli) 可查看/修改数据库：

```bash
buntdb-cli .lsp.db
> keys example:*    # 查看所有 example 插件的 key
> get example:userinfo:123
```

!!! danger "不要在 DDBOT 运行时操作"
    buntdb 不支持多写，DDBOT 运行时用 cli 操作会损坏数据库。务必先停止 DDBOT。

---

## 下一步

- [配置钩子](config-hook.md) - 自定义推送过滤
- [keyset 与注册](keyset-register.md) - 注册与 key 规范
- [完整范本](example-walkthrough.md)
