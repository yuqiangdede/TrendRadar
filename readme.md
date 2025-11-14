# TrendRadar（GitHub Actions 版本）

> 只保留“Fork 仓库 + GitHub Actions 定时触发”这一种运行模式，无需再关心 MCP、Docker 或本地脚本。

## 项目简介

TrendRadar 会定时抓取多个热门资讯平台的榜单，根据自定义权重合并为一份热度报告，并把结果推送到你配置的渠道（企业微信、飞书、钉钉、Telegram、邮件、ntfy 等）。  
仓库只依赖 `requests / PyYAML / pytz`，所有逻辑都在 `main.py` 中，方便审计与二次开发。

## 工作原理

- **唯一入口：GitHub Actions**（`.github/workflows/crawler.yml`）  
  - 默认从每天 08:00–22:00（UTC+8 对应 00:00–14:00 UTC）每小时触发一次。
  - 读取 `config/config.yaml` 作为爬虫与通知配置。
  - 将 Secrets 中的敏感凭证注入环境变量。
  - 执行 `python main.py`，推送成功后把生成的内容提交回仓库（便于生成静态页面或备份）。

如果你需要修改抓取频率，只需调整 `crawler.yml` 的 `cron` 表达式；其它触发方式（HTTP、本地脚本、Docker、MCP）已移除。

## 快速上手

1. **Fork 仓库**  
   点击右上角 `Fork`，建议保持默认设置（包含所有分支）。

2. **更新配置文件**  
   - 复制 `config/config.yaml` 至你的 Fork 中，根据个人需求调整：
     - `platforms`：勾选要采集的平台；
     - `weight`：调节热度权重；
     - `notification`：设置消息批量、推送时间窗等；
     - `report.mode`：`daily/current/incremental` 三种模式任选其一。
   - 如需敏感词过滤或关注词权重，请同步修改 `config/frequency_words.txt`。

3. **配置 GitHub Secrets**  
   在 Fork 仓库中打开 `Settings → Secrets and variables → Actions → New repository secret`，根据你要使用的通知渠道新增以下键值：

   | Secret 名称 | 说明 |
   |-------------|------|
   | `FEISHU_WEBHOOK_URL` | 飞书自定义机器人 Webhook |
   | `DINGTALK_WEBHOOK_URL` | 钉钉机器人 Webhook |
   | `WEWORK_WEBHOOK_URL` | 企业微信群机器人 Webhook |
   | `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` | Telegram 机器人信息 |
   | `EMAIL_FROM` / `EMAIL_PASSWORD` / `EMAIL_TO` / `EMAIL_SMTP_SERVER` / `EMAIL_SMTP_PORT` | 邮件通知所需字段 |
   | `NTFY_SERVER_URL` / `NTFY_TOPIC` / `NTFY_TOKEN` | ntfy 推送设置（可选） |

   > 未配置的渠道会被自动跳过，因此可以只添加自己需要的 Secrets。

4. **首次手动触发**  
   - 打开 `Actions → Hot News Crawler → Run workflow`，确认能够成功执行；
   - 之后便由 `cron` 定时触发，无需人工参与。

## 日常维护

- **修改配置/词库/README**：直接在 Fork 中编辑并推送即可，Actions 会读取最新内容。
- **查看运行日志**：每次任务会给出详细日志，失败时可以在 GitHub Actions 页面查看原因（网络问题、配置缺失等）。
- **更新上游改动**：定期从原仓库拉取更新（尤其是 `main.py` 和 `config/config.yaml` 的平台列表），再推送到自己的 Fork。

## 常见问题

- **如何降低频率/只在工作日运行？**  
  修改 `crawler.yml` 中的 `cron`，例如 `0 0-14 * * 1-5` 代表周一至周五 08:00–22:00（UTC+8）每小时运行一次。注意 GitHub Actions 使用 UTC。

- **需要公网服务器吗？**  
  不需要，一切运行在 GitHub 的 Runner 上。你只需维护配置和 Secrets。

- **能否继续使用 Docker/MCP？**  
  本分支已完全删除相关文件，如有需要请参考历史提交或原项目仓库。

## 目录结构

```
TrendRadar/
├── .github/workflows/crawler.yml   # 唯一的运行入口
├── config/                        # 采集与推送配置
│   ├── config.yaml
│   └── frequency_words.txt
├── main.py                        # 热点抓取与推送逻辑
├── requirements.txt               # 运行依赖（仅 requests/pytz/PyYAML）
├── README.md                      # 当前文档
└── AGENTS.md                      # 针对代理环境的一些操作规约
```

## 贡献

欢迎提交 PR 改进抓取逻辑或增加新的通知渠道，但请保持“只通过 GitHub Actions 运行”的前提。如果需要讨论新功能，可直接开 issue。
