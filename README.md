# uptime probe

站点可用性外部探测。每 5 分钟发起一次 HTTP 检查，异常时推送通知。

本仓库不含任何业务代码，只有一个 workflow。设为 public 是因为 GitHub Actions 对公开仓库免费且不限分钟数，私有仓库按分钟计费（每次 job 向上取整到整分钟，5 分钟一次约合 8,640 分钟/月）。

## 设计要点

- **仅在状态翻转时推送**（UP→DOWN、DOWN→UP），不会持续刷屏。状态存放在 Actions cache 中。
- 连续失败 **3 次**（间隔 10 秒）才判定为 DOWN，避免跨境网络抖动造成误报。
- **每周一发送一次心跳**，用于确认探测本身仍在运行。

## 配置

需要一个 repository secret：

- `NTFY_TOPIC` — ntfy.sh topic 名。存为 secret 而非明文，因为知道 topic 的人既能订阅通知内容、也能向该 topic 推送任意消息。

探测目标同样通过 secret 注入，不写在仓库里。

## 已知限制

GitHub 会在仓库 60 天无提交活动后自动禁用 scheduled workflow（会发邮件通知）。

> 若某个周一未收到心跳，说明探测已停止，需到 Actions 页面重新启用。

这条心跳是「监控的监控」—— 没有人看管的监控等于没有监控。
