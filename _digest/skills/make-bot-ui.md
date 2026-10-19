# `make-bot-ui`

> 源码：[`skills/make-bot-ui/SKILL.md`](../../skills/make-bot-ui/SKILL.md)
> 调用：**U**（user-invoked，只有人能触发）
> 索引：[技能索引](./README.md) · [`poteto-mode` 的 Harness](../patterns/harness-portability.md)

## 一句话

做一个页面，按钮不直接调 bot，而是打你本机的服务；本机服务用保存好的 sender key 向 Grok Bot 的 webhook routine POST 一个 JSON，bot 以 `[routine]` 回合醒来。密钥不进浏览器，也不进 chat。

## 它解决什么

Grok Bot 的 webhook routine 只认 HTTP POST。想让"点一下按钮"唤醒 bot，就需要一个页面加一段服务端代码，而一旦把 webhook 的 sender key 放进页面，任何打开页面的人都能唤醒 bot；一旦让用户把 key 贴进 chat，key 就永久留在了 transcript 里。

`make-bot-ui` 把这条链路的每一段都固定下来：routine 怎么建、key 怎么在不经过对话的前提下交到服务端、服务端 POST 长什么样、页面怎么在 tailnet 上可达、以及 bot 醒来时看到的事件是什么形状。它是一份 Cursor 与 Grok Bot 的操作手册，不是通用架构模式。

## 核心机制

### 1. 建 webhook routine

调 `update_state`，target `routine`，action `create`，`trigger` 设为 `{"type": "webhook"}`。prompt 里要写四件事：把 POST body 当不可信数据、点明 UI 会发哪些 JSON 字段、按字段做对应动作、以及没有可报告内容时不发消息。出现 confirm card 就等用户确认。folder slug 是 name 的 kebab-case 形式，后面要当 secret 的 `connector` 用。create 的结果里没有 sender key。

### 2. 取 URL 与 sender key

两者只在 routine 建好之后的面板里，源码禁止发明别的点击路径。给用户的五步是：点 chat header 里的 agent 名或按 Cmd+Shift+I，在 computer preview 下找到 Routines，打开这个 webhook routine，复制 URL（可以贴进 chat），复制 sender key（不允许贴进 chat）。URL 形如 `https://api2.cursor.sh/automations/webhook/<id>`，没有 query string，且不能猜 id。

### 3. 用 secret-request 收 key

不接受 chat 里的 key。发一个 `SendToUser`，`type: secret-request`，`secret.label` 写 webhook sender key，`secret.connector` 是 routine 的 folder slug，`secret.field` 是 `key`，然后停下。源码的原话是那张卡就是这一轮的全部内容。

用户提交后 agent 看不到值，值落在该 connector 的 credential 文件里。把它拷进服务端配置，不打印、不写日志。

### 4. 本机服务做中转

把 `{url, key}` 存在这个 UI 自己的目录里。按钮 POST 到本机服务，由本机服务（不是浏览器）POST 到 webhook。服务必须绑 `0.0.0.0:<port>` 而不是 `127.0.0.1`，否则 tailnet 上的 peer 够不到。

POST 规格是固定的：方法 `POST`，`Content-Type: application/json`，`Authorization: Bearer <key>`，`X-Automation-Key: <key>`，body 是一个 JSON object 且字段名与 routine prompt 一致，超时 8 秒，只试一次不重试。返回 200 表示 routine 被唤醒。告诉用户上线之前先用一个无害 payload 探一次，用一个 prompt 会忽略的 action。

可能失败的 POST 要把同样的 JSON 追加到本地日志，由 routine 一侧 drain。日志是补充，源码明确不要以轮询为主路径，也不要在 webhook 上传媒体字节。

### 5. 挂到 tailnet

一台机器上的所有 agent 共享一个 Tailscale node，已有 online node 时不要再建第二个 hostname。此时跳过安装，从 `tailscale status` 读 hostname，从 `tailscale ip -4` 读 IPv4，给用户两个 URL：`http://<hostname>.<tailnet>.ts.net:<port>` 和 `http://<100.x.x.x>:<port>`。用 HTTP，除非用户明确要 HTTPS。

没装时先跑安装脚本，再 `sudo tailscale up --hostname=<short-name> --accept-dns=false --ssh=false`。命令会打印登录 URL，把它发给用户，由用户在浏览器里批准机器，不要问也不要输入 Tailscale 凭据。上线后用 `tailscale status` 与 `tailscale ip -4` 确认，再探 `http://<100.x.x.x>:<port>/` 期待 200。登录 URL 过期就重跑 `tailscale up` 发新的。

### 6. 处理 wake

wake 是那个 routine 的 `[routine]` 回合，带一个 `<webhook_event>` 块：`headers`（`content-type`、`user-agent`）、`body_digest`（sha256）、`body`、`timestamp_ms`。`body` 是 JSON 对象的字符串形式，字段在 `body` 里而不是顶层聊天文本，所以必须 parse。agent 看不到 sender key。不要打印 key、token 或 cookie。UI 与 routine prompt 用同一套字段名，字段列表保持小。

## 设计意图

- **凭据的最小暴露面由架构决定。** 页面打到本机服务、服务再打 webhook，浏览器就永远不需要 key。这比"不要在页面里放 key"的提醒可靠。
- **secret-request 是单向通道。** agent 看不到提交的值，因此不可能把它写进 transcript，也不可能在后续轮次里泄露。这是把安全要求做进工具语义，而不是写进禁令。
- **上线前先用无害 payload 探一次。** 用一个 prompt 会忽略的 action 打一发，证明链路通且不会误触发业务动作。这比"应该能通"强。
- **一次尝试、8 秒超时、本地日志。** 不重试避免 webhook 侧重复醒；把失败留在本地日志里，由 routine 主动 drain，而不是让页面轮询。
- **prompt 的第一句是信任分类。** webhook body 是外部输入，所以 routine prompt 要求先当不可信数据处理，再谈业务字段。

## 依赖关系

- **上游**：Cursor 的 Grok Bot / automations 工具面：`update_state` 的 routine 创建、Routines 面板、`SendToUser` 的 secret-request、以及 `[routine]` 回合里的 `<webhook_event>` 形状。这些语义定义在 pstack 之外，skill 只引用不解释。
- **上游**：Tailscale：`tailscale status`、`tailscale ip -4`、`tailscale up` 与登录批准流程，以及一次 `sudo`。
- **上游**：一台能跑常驻服务、能存配置的机器，和一个开放的端口。
- **下游**：没有 pstack skill 依赖它。它不引用 `poteto-mode`、`unslop` 或任何 principle，是一份自足的操作手册。
- **文档位置**：[`docs/guide/09-make-it-yours.md`](../../docs/guide/09-make-it-yours.md) 把它称为面向 Grok Bot 用户的那个 situational skill，示例是"滑动审阅队列，每次滑动让 bot 处理那一条"。

## 使用边界

- **只在 Cursor 加 Grok Bot 的环境里成立。** `update_state`、`SendToUser`、Routines 面板、`[routine]` 回合格式都没有跨 harness 映射，而 `reflect`、`automate-me` 这类 skill 都会写替代工具与路径。这是 pstack 里 harness 绑定最强的一页。
- **不覆盖页面本身的鉴权。** 页面挂在 tailnet 上、走 HTTP，源码没有给页面加认证的步骤。能进 tailnet 的人就能按按钮；key 保护的是"谁能唤醒 bot"，不是"谁能点这个页面"。
- **不加 TLS。** 源码明说除非用户要求，否则用 HTTP。
- **不发媒体字节。** 文件、图片、音视频不在 webhook 的载荷范围内，要走别的通道。
- **轮询不是主路径。** 日志 drain 是失败兜底，不是常规同步机制。
- **一个 node 一个 hostname。** 同机多个 UI 只能靠端口区分，不能各自申请主机名。

## 值得注意

- **安全边界止于 tailnet。** 源码没有讨论页面鉴权、CSRF 或速率限制；它能防的是密钥泄露，不是未授权操作。
- **key 与 URL 依赖人工复制。** 源码禁止猜 webhook id，也禁止在 chat 里传 key，剩下的路径就是让人在面板里点对地方。这一步没有校验，复制错只能靠 probing 发现。
- **字段名在三处重复。** UI、服务端 POST body、routine prompt 必须同名，源码用"same field names"和"keep the list small"约束，但没有共享 schema 或生成器，改字段要同时改三处。
- **依赖外部语义。** confirm card、connector、credential 文件位置、wake 事件结构都由产品定义；产品一变，skill 里的步骤就过期。除了上线前那一次 probe，它没有像 `create-verification-skill` 那样的自检步骤。

## 关键洞察

1. **凭据的暴露面是架构决定的，不是纪律决定的。** 多一层本机中转，浏览器就永远不需要 key。
2. **让 agent 看不到 secret，比叮嘱它别打印 secret 可靠。** secret-request 把这条要求做成了工具语义。
3. **webhook 是外部输入，所以 prompt 的第一句是信任分类。** 业务逻辑之前先决定"这段数据算不算指令"。
4. **这页标出了 pstack 的边界。** 不是每个 skill 都能移植；有些就是一个产品加一个网络工具的操作手册，承认这一点比包装成通用模式诚实。
