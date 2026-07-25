# Concern 接口

`Concern` 是 DDBOT 订阅模块的核心抽象。每个订阅源都需要实现这个接口。

---

## 接口定义

```go
type Concern interface {
    // Site 必须全局唯一，不允许注册两个相同的 site
    Site() string

    // Types 返回该 Concern 支持的 concern_type.Type
    // 每一项必须是单个 type，第一个 type 为默认 type
    Types() []concern_type.Type

    // Start 启动 Concern 模块，记得调用 StateManager.Start
    Start() error

    // Stop 停止 Concern 模块，记得调用 StateManager.Stop
    Stop()

    // ParseId 解析一个 id，返回的 id 类型即其他地方的 id interface{} 类型
    // 推荐选择 int64（如 bilibili UID）或 string（如 douyu 房间号）
    ParseId(string) (interface{}, error)

    // Add 添加一个订阅
    Add(ctx mmsg.IMsgCtx, groupCode int64, id interface{}, ctype concern_type.Type) (IdentityInfo, error)

    // Remove 删除一个订阅
    Remove(ctx mmsg.IMsgCtx, groupCode int64, id interface{}, ctype concern_type.Type) (IdentityInfo, error)

    // Get 获取一个订阅信息
    Get(id interface{}) (IdentityInfo, error)

    // GetStateManager 获取 IStateManager
    GetStateManager() IStateManager

    // FreshIndex 刷新 group 的 index，通常用 StateManager.FreshIndex 默认实现
    FreshIndex(groupCode ...int64)
}
```

---

## Event 与 Notify

### Event

`Event` 是对订阅对象行为的抽象（发动态、开播等），由爬虫产生：

```go
type Event interface {
    Site() string
    Type() concern_type.Type
    GetUid() interface{}
    Logger() *logrus.Entry
}
```

**Event 不应关联接收方信息**（如 QQ 群号）。

### Notify

`Notify` 在 Event 基础上加了接收方信息：

```go
type Notify interface {
    Event
    GetGroupCode() int64
    ToMessage() *mmsg.MSG
}
```

一个 Event 可能对应多个 Notify（多个群订阅了同一对象）。

### IdentityInfo

```go
type IdentityInfo interface {
    GetUid() interface{}   // 订阅对象 id
    GetName() string       // 订阅对象名字
}
```

---

## 实现示例

以 `example` 网站为例：

```go
package concern

import (
    "github.com/cnxysoft/DDBOT-WSa/lsp/concern"
    "github.com/cnxysoft/DDBOT-WSa/lsp/concern_type"
    "github.com/cnxysoft/DDBOT-WSa/lsp/mmsg"
)

const (
    Site        = "example"
    ExampleType concern_type.Type = "example"
)

type Concern struct {
    *StateManager
    notify chan<- concern.Notify
}

func (c *Concern) Site() string {
    return Site
}

func (c *Concern) Types() []concern_type.Type {
    return []concern_type.Type{ExampleType}
}

func (c *Concern) ParseId(s string) (interface{}, error) {
    // example 用 string id
    return s, nil
}

func (c *Concern) GetStateManager() concern.IStateManager {
    return c.StateManager
}

func NewConcern(notify chan<- concern.Notify) *Concern {
    c := &Concern{
        notify: notify,
    }
    c.StateManager = NewStateManager(c)
    return c
}
```

---

## Add / Remove / Get

通常直接复用 `concern.StateManager` 的默认实现，无需自己写：

```go
func (c *Concern) Add(ctx mmsg.IMsgCtx, groupCode int64, id interface{}, ctype concern_type.Type) (concern.IdentityInfo, error) {
    return c.StateManager.Add(ctx, groupCode, id, ctype)
}

func (c *Concern) Remove(ctx mmsg.IMsgCtx, groupCode int64, id interface{}, ctype concern_type.Type) (concern.IdentityInfo, error) {
    return c.StateManager.Remove(ctx, groupCode, id, ctype)
}

func (c *Concern) Get(id interface{}) (concern.IdentityInfo, error) {
    return c.StateManager.Get(id)
}
```

如果订阅时需要做额外操作（如 B 站自动关注），可在自己的 `Add` 里实现后调用 `StateManager.Add`。

---

## Start / Stop

```go
func (c *Concern) Start() error {
    // 初始化爬虫、cookie 等
    c.StateManager.Start()
    return nil
}

func (c *Concern) Stop() {
    c.StateManager.Stop()
}
```

`StateManager.Start` 会启动轮询（如果用了 EmitQueue），见 [StateManager](statemanager.md)。

---

## 类型选择建议

### id 类型

| 场景 | 推荐 | 例子 |
|------|------|------|
| 订阅源有数字唯一标识 | `int64` | bilibili UID |
| 有数字也有字符，或纯字符 | `string` | douyu 房间号、huya 主播 ID |

### ParseId

```go
// int64 类型
func (c *Concern) ParseId(s string) (interface{}, error) {
    return strconv.ParseInt(s, 10, 64)
}

// string 类型
func (c *Concern) ParseId(s string) (interface{}, error) {
    return s, nil
}
```

---

## 下一步

- [StateManager](statemanager.md) - 状态管理与轮询
- [配置钩子](config-hook.md) - 自定义推送过滤
- [keyset 与注册](keyset-register.md) - 注册与数据库 key
