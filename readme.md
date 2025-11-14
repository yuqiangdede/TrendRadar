# TrendRadar（GitHub Actions 版）

> 运行模式仅保留“Fork 仓库 + GitHub Actions 定时触发”，推送渠道只支持企业微信机器人。

## 项目简介

TrendRadar 会定时抓取多个热门资讯平台的榜单数据，根据自定义权重合并为一份热度报告，并推送到企业微信群机器人。项目只依赖 `requests`、`PyYAML`、`pytz`，全部逻辑集中在 `main.py`，方便审计和二次开发。

## 工作原理

- 唯一入口：`.github/workflows/crawler.yml`
  - 默认按照北京时间 08:00–22:00（UTC 0 点到 14 点）每小时触发一次，可自行修改 `cron` 表达式。
  - 读取 `config/config.yaml` 作为爬虫与推送配置；敏感凭证从 GitHub Secrets 注入。
  - 执行 `python main.py`，推送完成后把输出提交回仓库，便于生成静态页面或备份。

## 快速上手

1. **Fork 仓库**：点击右上角 `Fork`，建议保留所有分支。
2. **修改配置文件**：在自己的 Fork 中编辑 `config/config.yaml` 与 `config/frequency_words.txt`，调节平台列表、权重、关键词过滤等参数。
3. **配置 Secrets**：前往 `Settings → Secrets and variables → Actions → New repository secret`，新增唯一所需的密钥：
   - `WEWORK_WEBHOOK_URL`：企业微信群机器人 Webhook 地址。
4. **首次手动触发**：在 `Actions → Hot News Crawler` 中点击 `Run workflow`，确认执行成功后即可交给定时任务。

## 日常维护

- **更新配置或词库**：直接在 Fork 中修改并推送，GitHub Actions 会读取最新内容。
- **查看运行日志**：每次任务在 Actions 页面都会记录详细日志，失败时可快速定位网络或配置问题。
- **同步上游改动**：定期从原仓库拉取更新（尤其是 `main.py` 与 `config/config.yaml`），再合并到自己的 Fork。

## 常见问题

- **如何调整触发频率？** 修改 `crawler.yml` 的 `cron`，例如 `0 0-14 * * 1-5` 代表工作日 08:00–22:00 每小时运行一次（注意使用 UTC）。
- **是否需要服务器？** 不需要，一切运行在 GitHub Actions 的 Runner 上，只需维护配置与 Secrets。
- **还能使用其他推送渠道吗？** 本版本已彻底移除，若有需要可参阅历史提交或原项目实现。

## 目录结构

```
TrendRadar/
├── .github/workflows/crawler.yml   # 唯一运行入口
├── config/                        # 采集与推送配置
│   ├── config.yaml
│   └── frequency_words.txt
├── main.py                        # 热点抓取与推送逻辑
├── requirements.txt               # 运行依赖
├── README.md                      # 当前文档
└── AGENTS.md                      # 代理/环境说明
```

## 贡献

欢迎提交 PR 改进抓取逻辑或扩展企业微信推送内容；如需讨论新功能，请直接开 issue。
