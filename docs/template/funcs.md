# 模板函数

DDBOT 模板内置了大量函数，本页按分类列出全部函数。WSa 新增的函数有标注。

!!! tip "函数分类"
    左侧目录可快速跳转到各分类。

---

## 消息控制

| 函数 | 说明 |
|------|------|
| `{{ cut }}` | 分段消息，把后续内容作为新消息发送 |
| `{{ reply .msg }}` | 回复原消息 |
| `{{ prefix }}` | 引用命令前缀（默认 `/`） |
| `{{ abort }}` / `{{ abort "msg" }}` | 退出当前模板并丢弃已产生内容，有参数则发送参数 |
| `{{ fin }}` | 退出当前模板并发送已产生内容，跳过后续代码 |

---

## 媒体发送

| 函数 | 说明 |
|------|------|
| `{{ pic "uri" }}` | 发送图片，支持 URL/路径/base64/文件夹随机 |
| `{{ video "path" }}` | 发送视频（WSa） |
| `{{ record "path" }}` | 发送语音（WSa） |
| `{{ file "path" }}` | 发送任意文件（WSa） |
| `{{ remoteDownloadFile "url" }}` | 让实现端下载远端文件，返回服务器绝对路径（WSa） |
| `{{ openFile "path" }}` | 读取本地文件为 `[]byte` |
| `{{ downloadFile "url" "path" "filename" }}` | 下载文件到本地，支持 Headers/Cookies/UA（WSa） |

!!! warning "openFile 安全"
    `openFile` 不对参数做安全检查，**绝对不要把用户输入作为参数**。

---

## 交互

| 函数 | 说明 |
|------|------|
| `{{ at 123456 }}` | @指定 QQ 号 |
| `{{ icon 123456 }}` | 发送指定 QQ 号的头像 |
| `{{ poke 123456 }}` | 戳一戳（仅群聊，v1.0.9+） |
| `{{ reCall .msg }}` | 撤回消息（WSa） |
| `{{ getMsg .msg_id }}` | 根据消息 ID 获取消息（WSa） |

---

## 成员与群信息

| 函数 | 说明 |
|------|------|
| `{{ bot_uin }}` | 获取 BOT 的 QQ 号 |
| `{{ member_info .group_code .member_code }}` | 获取群成员信息，返回 `name`/`gender`/`permission`/`uin` |
| `{{ member_list .group_code }}` | 获取群成员列表（WSa） |
| `{{ isAdmin .member_code .group_code }}` | 检查是否管理员（WSa） |
| `{{ getFileUrl .group_code .file_id }}` | 获取群文件下载 URL（WSa） |

`member_info` 返回字段：

| 字段 | 含义 |
|------|------|
| `name` | 群名片，没设则是 QQ 昵称 |
| `gender` | 性别：2 男 / 1 女 / 0 未公开 |
| `permission` | 群权限：10 群主 / 5 管理员 / 1 普通成员 |
| `uin` | QQ 号 |

---

## 时间函数

| 函数 | 说明 |
|------|------|
| `{{ hour }}` | 当前小时 [0,23] |
| `{{ minute }}` | 当前分钟 [0,59] |
| `{{ second }}` | 当前秒 [0,59] |
| `{{ year }}` | 当前年份 |
| `{{ month }}` | 当前月份 [1,12] |
| `{{ day }}` | 当月第几天 |
| `{{ weekday }}` | 本周第几天 [1,7]（v1.0.8+） |
| `{{ yearday }}` | 当年第几天 [1,365/366] |
| `{{ getUnixTime 1640995200 "2006-01-02 15:04:05" }}` | Unix 时间戳转格式化字符串（WSa） |
| `{{ getTimeStamp "2022-01-01 12:00:00" }}` | 时间字符串转 Unix 时间戳（WSa） |
| `{{ getTime "now" "2006-01-02" }}` | 格式化时间，第一参可为 `"now"` 或时间串或 `time.Time`（WSa） |

!!! note "Go 时间格式化"
    Go 的时间格式化用**参考时间** `2006-01-02 15:04:05`，不是 `YYYY-MM-DD`。

    `getTime` 第二参也支持预设：`"datetime"`/`"dateonly"`/`"timeonly"`/`"stamp"`。

---

## 随机与选择

| 函数 | 说明 |
|------|------|
| `{{ roll a b }}` | 在 a~b 范围随机一个 int64 |
| `{{ choose "a" "b" "c" }}` | 从参数中随机返回一个 |
| `{{ choose "a" "b" 1 "c" 5 }}` | 带权重选择（v1.0.8+），权重省略默认 1 |

带权重示例：`{{ choose "a" "b" 1 "c" 5 }}` 中，"c" 的概率是 5/7。

---

## 类型转换

| 函数 | 说明 |
|------|------|
| `{{ float64 123 }}` | 转 float64 |
| `{{ int 123 }}` | 转 int（32/64 位表现不一致） |
| `{{ int64 123 }}` | 转 int64 |
| `{{ toString 123 }}` | 转 string |
| `{{ toJson $v }}` | 转 JSON `[]byte`（WSa） |
| `{{ jsonToDictOrArray $json true }}` | JSON 转 Dict 或 Dict 数组，第二参 true 为数组（WSa） |

---

## 数学函数

末尾带 `f` 的返回 float64，否则 int64。

| 函数 | 示例 |
|------|------|
| `add` / `addf` | `{{ add 1 2 }}` = 3 |
| `sub` / `subf` | `{{ sub 1 2 }}` = -1 |
| `mul` / `mulf` | `{{ mul 2 2 }}` = 4 |
| `div` / `divf` | `{{ div 10 5 }}` = 2 |
| `mod` / `modf` | `{{ mod 10 3 }}` = 1 |
| `max` / `maxf` | `{{ max 1 2 3 4 5 }}` = 5 |
| `min` / `minf` | `{{ min 1 2 3 4 5 }}` = 1 |

---

## 哈希函数

| 函数 | 示例 |
|------|------|
| `base64encode` | `{{ base64encode "hello" }}` |
| `base64decode` | `{{ base64decode "aGVsbG8=" }}` |
| `md5sum` | `{{ md5sum "hello" }}` |
| `sha1sum` | `{{ sha1sum "hello" }}` |
| `sha256sum` | `{{ sha256sum "hello" }}` |
| `adler32sum` | `{{ adler32sum "hello" }}` |
| `uuid` | `{{ uuid }}` 生成 UUID |

---

## 字符串函数

| 函数 | 说明 |
|------|------|
| `hasPrefix "pre" "str"` | 是否有指定前缀 |
| `hasSuffix "suf" "str"` | 是否有指定后缀 |
| `contains "sub" "str"` | 是否包含子串 |
| `trim " str "` | 去前后空白 |
| `trimAll "abc" "abcHelloabc"` | 去两端指定字符（WSa） |
| `trimSuffix "suf" "str"` | 去指定后缀 |
| `trimPrefix "pre" "str"` | 去指定前缀 |
| `split "sep" "str"` | 按分隔符分割，返回 list |
| `join "sep" (list ...)` | 按分隔符拼接 |
| `trunc 2 "abcde"` | 按长度截取前 N 字符 |
| `reTrunc "str" 5` | 从末尾截取 N 字符（WSa） |
| `upper "abc"` | 转大写 |
| `lower "ABC"` | 转小写 |
| `title "hello world"` | 单词首字母大写 |
| `snakecase "FirstName"` | 蛇形命名 |
| `camelcase "first_name"` | 驼峰命名 |
| `kebabcase "FirstName"` | 短横线命名（WSa） |
| `replace "old" "new" "str"` | 替换第一次出现（WSa） |
| `replaceAll "old" "new" "str"` | 替换全部出现（WSa） |
| `find "sub" "str"` | 查找第一次出现位置（WSa） |
| `findLast "sub" "str"` | 查找最后一次出现位置（WSa） |
| `count "sub" "str"` | 统计子串出现次数（WSa） |
| `uriEncode "hello world"` | URI 编码（WSa） |
| `uriDecode "hello%20world"` | URI 解码（WSa） |

---

## 默认值与空值

| 函数 | 说明 |
|------|------|
| `empty $v` | 是否为空 |
| `nonEmpty $v` | 是否非空 |
| `coalesce $a $b $c` | 返回第一个非空的值 |
| `ternary "有参" "无参" (nonEmpty .args)` | 三元运算糖 |
| `all "" 0 1` | 是否全部非空 |
| `any "" 0 1` | 是否有一个非空 |

---

## list 函数

| 函数 | 说明 |
|------|------|
| `list "a" "b" "c"` | 创建 list |
| `append $list "d"` | 末尾添加，返回新 list |
| `prepend $list "d"` | 开头添加，返回新 list |
| `concat $l1 $l2 $l3` | 连接多个 list |
| `loop 1 5` | 创建 1 到 5 的循环（WSa），配合 `range` 用 |

`loop` 示例：

```text
{{ range $i := loop 1 5 }}{{ $i }}{{ end }}
```

---

## dict 函数

| 函数 | 说明 |
|------|------|
| `dict "a" 1 "b" 2` | 创建 dict（key 必须是 string） |
| `get $dict "key"` | 取值 |
| `set $dict "key" "val"` | 设置值 |
| `unset $dict "key"` | 删除 key |
| `hasKey $dict "key"` | 是否存在 key |
| `keys $dict` | 获取所有 key（WSa） |
| `values $dict` | 获取所有值（WSa） |
| `pluck "key" $d1 $d2` | 从多个 dict 提取指定 key（WSa） |
| `omit $dict "k1" "k2"` | 排除指定 key（WSa） |
| `pick $dict "k1" "k2"` | 只保留指定 key |
| `merge $d1 $d2` | 合并，不覆盖已有 key |
| `mergeOverwrite $d1 $d2` | 合并，覆盖已有 key |
| `mustMerge $d1 $d2` | 合并（带错误处理，WSa） |
| `mustMergeOverwrite $d1 $d2` | 覆盖合并（带错误处理，WSa） |
| `dig "k1" "k2" "default" $dict` | 嵌套查找（WSa） |
| `deepCopy $obj` | 深拷贝（WSa） |

---

## JSON 处理

使用 [gjson](https://github.com/tidwall/gjson) 库：

```text
{{- $data := `{"name":{"first":"Janet"},"age":47}` -}}
{{- $j := toGJson $data -}}
{{- ($j.Get "name.first").String -}}
{{- ($j.Get "age").Int -}}
```

| 函数 | 说明 |
|------|------|
| `toGJson $jsonStr` | 转 GJson 对象 |
| `toJson $v` | 转 JSON `[]byte`（WSa） |
| `jsonToDictOrArray $bytes true` | JSON 转 Dict/数组（WSa） |

---

## HTTP 请求

| 函数 | 说明 |
|------|------|
| `httpGet "url"` / `httpGet "url" $params` | GET 请求 |
| `httpPostJson "url" $params` | POST JSON |
| `httpPostForm "url" $params` | POST 表单 |

`$params` 是 dict，可用 `dict` 创建。

### 特殊参数

通过 dict 传特殊参数控制 HTTP 行为，不会真正发送：

| 参数 | 说明 |
|------|------|
| `DDBOT_REQ_DEBUG` | 输出请求细节（含隐私，慎用） |
| `DDBOT_REQ_USER_AGENT` | 自定义 UA |
| `DDBOT_REQ_HEADER` | 自定义 header，list 形如 `["A=B"]` |
| `DDBOT_REQ_COOKIE` | 自定义 cookie，list 形如 `["A=B"]` |
| `DDBOT_REQ_PROXY` | 代理控制，见下表 |
| `DDBOT_REQ_TIMEOUT` | 超时，如 `10s`/`1m`，默认 15s，**不能设 0**（WSa） |
| `DDBOT_REQ_RETRY` | 重试次数，如 `3`，默认 0（WSa） |

`DDBOT_REQ_PROXY` 取值：

| 值 | 说明 |
|----|------|
| `prefer_mainland` | 用国内代理（需配代理池） |
| `prefer_oversea` | 用翻墙代理（需配代理池） |
| `prefer_none` | 不用代理 |
| `prefer_any` | 随机选 |
| `http://localhost:7890` | 直接用指定代理 |

示例：

```text
{{- $d := dict -}}
{{- $d = set $d "DDBOT_REQ_USER_AGENT" "MyBot" -}}
{{- $d = set $d "DDBOT_REQ_PROXY" "prefer_none" -}}
{{- $j := httpGet "https://example.com/api" $d | toGJson -}}
{{ ($j.Get "data").String }}
```

---

## 文件操作

!!! warning "安全"
    文件操作函数不做安全检查，**绝对不要把用户输入作为参数**。

| 函数 | 说明 |
|------|------|
| `openFile "path"` | 读取文件为 `[]byte` |
| `readLine "path" 3` | 读取第 N 行（WSa） |
| `findReadLine "path" "keyword"` | 查找含关键字的行（WSa） |
| `findWriteLine "path" "old" "new"` | 查找并替换行（WSa） |
| `writeLine "path" 3 "content"` | 写入第 N 行（WSa） |
| `updateFile "path" "content"` | 追加到文件末尾（WSa） |
| `writeFile "path" "content"` | 覆盖写入（WSa） |
| `delFile "path"` | 删除文件（WSa） |
| `renameFile "old" "new"` | 重命名（WSa） |
| `lsDir "path" true` | 列出目录，第二参是否递归（WSa） |

---

## 积分管理

| 函数 | 说明 |
|------|------|
| `getScore $uin $groupCode` | 获取积分（WSa） |
| `addScore $uin $groupCode 10` | 增加积分（WSa） |
| `subScore $uin $groupCode 5` | 减少积分（WSa） |
| `setScore $uin $groupCode 100` | 设置积分（WSa） |

---

## 列表输出

| 函数 | 说明 |
|------|------|
| `getIListJson $groupCode "site" $msgCtx` | 获取群关注列表 JSON（WSa） |
| `outputIList $msgCtx $groupCode "site"` | 输出原版 LIST 列表（自带分片，WSa） |

`site` 可不填，输出所有站点。`msgContext` 由 list 命令模板提供。

---

## 其它

| 函数 | 说明 |
|------|------|
| `getEleType $element` | 获取元素类型：`image`/`file`/`unknown`（WSa） |
| `cooldown "10s" "key"` | 冷却计时，设定时间内只有第一次返回 true（v1.0.9+） |
| `sleep "1s"` | 暂停指定时间（WSa） |

### `cooldown` 详解

第一个参数为时间单位，支持：

- `500ms`、`1s`、`20m`、`1.5h`、`2h45m`
- 设为 0 或负数自动替换为 `5m`

后续参数为 cooldown 关键字，相同关键字在时间范围内只能触发一次。

```text
{{- if (cooldown "10s" .template_name) -}}
成功
{{- else -}}
失败，正在冷却
{{- end -}}
```

把 `.member_code` 加入关键字可实现每人独立冷却：

```text
{{- if (cooldown "10s" .member_code .template_name) -}}
```

---

## 下一步

- [命令模板](command-tmpl.md) -- 各命令的专用变量
- [推送模板](notify-tmpl.md) -- 各订阅源的推送变量
- [事件模板](trigger-tmpl.md) -- 各事件的变量
