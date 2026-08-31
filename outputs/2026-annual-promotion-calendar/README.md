# 2026 三平台年度促销邮件资料库

从 Zoho Mail 检索 Amazon、Rakuten、Mercari Shops 官方促销邮件，并按平台及邮件接收日期整理。检索范围为 **2025-11-01 至 2026-08-31**，提取日期为 **2026-08-31**。

## 内容

- [Amazon](amazon.md)：6 封
- [Rakuten](rakuten.md)：105 封
- [Mercari Shops](mercari.md)：20 封
- `raw/`：逐封可读正文与来源元数据

## 方法与边界

- Amazon 查询 Prime Day、Prime 感謝祭、Black Friday 与タイムセール相关主题。
- Rakuten 查询 スーパーSALE、お買い物マラソン、キャンペーン，并排除银行、客服、培训及纯广告邮件。
- Mercari 使用官方商家资讯发件人，保留平台级活动、费用/广告返还、促销日历及タイムセール规划信息。
- 重叠检索结果按 Message ID 去重；促销口令、token、认证信息按 `[REDACTED]` 脱敏。
- 原始邮件正文来自 Zoho 的 plain-content 输出；移除无阅读价值的 CSS/HTML 样式噪音，不包含订单、客户咨询或客户个人信息邮件。
- 当前账户自 2025-11 起有数据，因此这不是完整自然年度历史；活动日期与截止日需以邮件原文为准，未明确的信息不推断。

## 检索统计

| 平台 | 候选条目 | 去重后 | 收录 |
|---|---:|---:|---:|
| Amazon | 11 | 6 | 6 |
| Rakuten | 150 | 150 | 105 |
| Mercari Shops | 61 | 61 | 20 |
