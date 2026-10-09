# StateManager

`StateManager` 管理订阅状态、配置、缓存、轮询。插件通过继承 `concern.StateManager` 复用大部分能力。

---

## 创建 StateManager

`concern` 包没有通用的 `NewStateManager(option)`，需要按 id 类型选构造函数；第一个参数是站点名（一般用 `Site`），第二个是 `concern.GetNotifyChan()`：

```go
type StateManager struct {
    *concern.StateManager
    // 可加自己的扩展字段
}

func NewStateManager(c *Concern) *StateManager {
    s := &StateManager{}
    // id 是 string（如推特用户名）
    s.StateManager = concern.NewStateManagerWithStringID(Site, concern.GetNotifyChan())
    // id 是 int64（如 B 站 UID）时改用：
    // s.StateManager = concern.NewStateManagerWithInt64ID(Site, concern.GetNotifyChan())
    // 需要自定义 KeySet 时用：
    // s.StateManager = concern.NewStateManagerWithCustomKey(Site, keySet, concern.GetNotifyChan())
    // 初始化 EmitQueue（如果用轮询）
    s.UseEmitQueue()
    return s
}
```

可用构造函数（见 `lsp/concern/stateManager.go`）：

| 函数 | 说明 |
|------|------|
| `NewStateManagerWithStringID(name, notifyChan)` | id 为 string |
| `NewStateManagerWithInt64ID(name, notifyChan)` | id 为 int64 |
| `NewStateManagerWithCustomKey(name, keySet, notifyChan)` | 自定义 `KeySet` |

`concern.StateManager` 提供 `FreshIndex`、`UseEmitQueue`、`UseFreshFunc`、`UseNotifyGeneratorFunc` 等能力；`Add`/`Remove`/`Get` 属于 `Concern` 接口，需要在你的 Concern 上实现（内置订阅源通常委托给内嵌的 StateManager 和自己的爬虫逻辑）。

---

## 轮询器 EmitQueue

当订阅网站没有高效的爬虫方式时，只能**轮询**--依次访问每个订阅目标的页面看是否有新信息。DDBOT 内置了 EmitQueue 轮询器。

### 启用

```go
s.UseEmitQueue()
```

### 提供 FreshFunc

```go
s.UseFreshFunc(s.EmitQueueFresher(func(p concern_type.Type, id interface{}) ([]concern.Event, error) {
    // id 是此时轮到的目标信息
    // p 是 id 所有订阅过的 Type 的集合
    // 这里访问目标页面，返回新的 Event 列表
    return []concern.Event{...}, nil
}))
```

框架会按 `concern.emitInterval`（默认 5s）依次调用 FreshFunc 轮询每个订阅 id。

### 工作流程

```mermaid
flowchart LR
    A[EmitQueue 按间隔出队 id] --> B[FreshFunc 请求页面]
    B --> C[产生 Event]
    C --> D[框架 Dispatch]
    D --> E[按订阅群生成 Notify]
    E --> F[写入 notifyChan]
```

---

## FreshIndex

刷新群索引，框架启动和每 30 秒会自动调用。通常不需要自己实现，用默认即可：

```go
func (c *Concern) FreshIndex(groupCode ...int64) {
    c.StateManager.FreshIndex(groupCode...)
}
```

---

## Notify 生成

轮询产生 Event 后，框架需要知道如何把 Event 变成 Notify（加上群信息）。用 `UseNotifyGeneratorFunc` 配置，且必须在 `Start()` 之前设置，否则会 panic：

```go
s.UseNotifyGeneratorFunc(func(groupCode int64, event concern.Event) []concern.Notify {
    return []concern.Notify{
        &ExampleNotify{
            Event:     event,
            GroupCode: groupCode,
        },
    }
})
```

`concern.NotifyGeneratorFunc` 的签名是 `func(groupCode int64, event Event) []Notify`——**返回的是切片**，一个 Event 可对应多个 Notify。你的 `Notify` 结构体需实现 `GetGroupCode()` 和 `ToMessage()` 把事件转成 QQ 消息。

---

## 缓存

`StateManager` 继承了 `*localdb.ShortCut`，可直接用 `Set` / `Get` / `SetJson` 等 KV 方法做缓存。内置的 B 站/推特/抖音等实现会在各自的 StateManager 上另加 `SetGroupCompactMarkIfNotExist` / `SetNotifyMsg` 辅助方法（**不是 `concern.StateManager` 自带的**）：

```go
// 标记已处理（该方法由具体订阅源的 StateManager 提供）
err := c.SetGroupCompactMarkIfNotExist(groupCode, compactKey)
if localdb.IsRollback(err) {
    // 已存在，需要 compact（简化推送）
}
```

通用 KV 用法见 [持久化](persistence.md)。

---

## 配置

每个「群 × 订阅 id」可独立配置（@、过滤、推送开关）。`StateManager` 提供 `GetGroupConcernConfig`：

```go
func (s *StateManager) GetGroupConcernConfig(groupCode int64, id interface{}) concern.IConfig {
    return NewGroupConcernConfig(s.StateManager.GetGroupConcernConfig(groupCode, id))
}
```

重写这个方法可以注入自定义的 `GroupConcernConfig`，详见 [配置钩子](config-hook.md)。

---

## 消息发送

`StateManager` 不直接发送消息，而是把 Notify 写入 `notifyChan`，由框架统一消费发送：

```go
// 在 FreshFunc 或事件处理中
c.notify <- &ExampleNotify{...}
```

框架的 `ConcernNotify` 协程会读取并处理：

- 检查禁言状态
- 跑 NotifyBeforeCallback
- 渲染消息
- AtBeforeHook 决定 @
- 限流后发送
- NotifyAfterCallback

---

## 下一步

- [持久化](persistence.md) - 数据存储
- [配置钩子](config-hook.md) - 自定义过滤逻辑
- [完整范本](example-walkthrough.md) - 真实实现走查
