[Uploading README.md…]()
# Temu 日本本地仓选品雷达 / Temu JP Selection Radar

> 一个运行在 Chrome / Edge 本地的 Manifest V3 浏览器扩展，用于整理 Temu 日本站页面上可见的商品与店铺信息，并通过可解释的规则评分辅助选品。

![项目状态](https://img.shields.io/badge/status-portfolio%20demo-4c8bf5)
![Manifest](https://img.shields.io/badge/Chrome-Manifest%20V3-4285F4)
![数据处理](https://img.shields.io/badge/data-local%20only-2ea44f)

## 项目简介

该项目来自跨境电商选品场景：手工逐页记录商品价格、销量、店铺和趋势信息耗时且容易遗漏。扩展在用户主动启动后自动滚动当前 Temu 页面，读取页面上已经展示的商品与店铺字段，将多次观察形成历史快照，并用固定权重规则生成候选排序，帮助使用者更快完成初筛。

本项目**不是 AI/机器学习模型**，不会预测未来销量，也不保证选品或盈利结果。所谓“雷达评分”是基于日销、短期趋势、爆发系数、累计销量和店内排名的可解释启发式评分。

## 功能

- 自动滚动并收集当前列表页或店铺页上可见的商品信息
- 识别价格、累计销量、日销/周销、本地仓标签和广告标记
- 保存最多 40 个本地历史销量快照，计算短期均值、趋势与爆发系数
- 从商品详情页补全店铺链接，并可继续扫描店铺商品
- 按价格、最低日销、本地仓和非广告条件筛选
- 生成商品与店铺规则评分，展示候选 Top 5
- 导出商品和店铺 CSV
- 所有采集结果仅保存在 `chrome.storage.local`，代码不包含外部数据上传逻辑

## 安装

1. 下载或克隆本仓库。
2. 打开 `chrome://extensions/`（Chrome）或 `edge://extensions/`（Edge）。
3. 开启“开发者模式”。
4. 选择“加载已解压的扩展程序”。
5. 选择本仓库中的 `extension/` 文件夹。
6. 打开或刷新 Temu 页面，再点击浏览器工具栏中的扩展图标。

## 使用方法

1. 打开 Temu 日本站的商品列表、搜索结果或店铺页。
2. 在扩展面板中设置筛选条件，点击“开始自动采集”。
3. 随时点击“停止”；扫描也会在页面底部连续多轮无新增数据后结束。
4. 在商品详情页使用“一键补全并挖店铺”，补全当前商品与店铺信息。
5. 查看候选商品或导出 CSV 做进一步人工判断。

页面结构随时可能变化；若采集字段为空，请先刷新页面并重新加载扩展。请控制使用频率，并遵守所在地法律、网站条款与访问限制。

## 规则评分说明

商品分数由以下规则相加（广告商品额外减 100 分）：

| 指标 | 权重上限 | 计算方式 |
| --- | ---: | --- |
| 最佳日销 | 50 | 日销在 0–150 区间线性映射 |
| 3 日对 7 日趋势 | 20 | 趋势截断在 -50% 至 150% 后映射 |
| 爆发系数 | 15 | `3 日均值 / 7 日均值`，1–4 倍线性映射 |
| 累计销量 | 10 | 0–5,000 线性映射 |
| 店内排名 | 5 | 店内排名前 3 加 5 分 |

店铺分数同样是固定规则，综合有效商品数、日销达到 20/50 的商品数、中位日销、爆款数量与爆款密度，并对头部商品过度集中作轻微扣分。完整实现见 [`extension/popup.js`](extension/popup.js)。这些阈值来自业务启发，不构成统计验证或效果承诺。

## 架构与数据流

```text
用户主动点击开始
       │
       ▼
content.js 读取当前 Temu 页面 DOM ──► 解析/去重/广告与本地仓识别
       │                                      │
       │                                      ▼
       └────────────────────────────► chrome.storage.local
                                              │
                                              ▼
popup.js 读取本地数据 ──► 筛选、规则评分、Top 5、CSV 导出
```

- `extension/manifest.json`：Manifest V3 权限、Temu 域名范围与入口
- `extension/content.js`：页面识别、DOM 解析、滚动扫描、快照与趋势指标
- `extension/popup.html` / `popup.css`：扩展面板界面
- `extension/popup.js`：筛选、评分、汇总、交互与 CSV 导出

## 隐私、安全与合规

- 未发现硬编码密钥、密码、Cookie、个人联系方式或远程遥测。
- 扩展只声明 Temu 域名访问范围；`storage` 用于本地记录，`activeTab` / `tabs` 用于与当前页通信和打开店铺/候选链接。
- 扫描只读取当前页面已经加载的内容，但自动滚动和卖家入口点击仍可能受网站条款、robots 策略、地区法律或账号政策约束。
- 页面内容属于不可信输入。界面展示前对标题和 URL 做 HTML 转义；CSV 导出内容仍应在打开前人工确认。
- 不建议高频、无人值守或绕过验证码/访问控制地运行。本项目不包含绕过验证码、登录控制或反自动化机制的代码。

更完整的公开前审查记录见 [`docs/SECURITY_AND_COMPLIANCE.md`](docs/SECURITY_AND_COMPLIANCE.md)。

## 截图与演示

仓库尚未包含真实业务页面截图，以避免公开账号、店铺或商品数据。发布前可按 [`docs/screenshots/README.md`](docs/screenshots/README.md) 的清单添加脱敏截图或短视频。

## 已知限制

- DOM 选择与正则解析依赖 Temu 当前页面结构，网站改版可能导致字段缺失。
- 页面显示的“已售”“日销”等值可能是近似值，且不同页面口径可能不同。
- 趋势需要跨时间重复采集；单次扫描不能形成可靠趋势。
- 规则评分未经因果实验或离线回测验证，不应替代利润、物流、合规与供应链评估。
- 自动滚动最多执行 320 轮，每轮约 1.25 秒；可随时手动停止。

## 许可与权属

本仓库暂未附开源许可证。公开可见不等于允许复制、修改或商用；在确认原实习单位对代码、品牌和业务数据的权属，并获得必要授权之前，建议保留“未许可 / All rights reserved”状态。详见 [`RIGHTS.md`](RIGHTS.md)。

---

## English

### Overview

Temu JP Selection Radar is a local Chrome/Edge Manifest V3 extension that organizes product and store information already visible on Temu Japan pages. After the user explicitly starts a scan, it scrolls the current page, parses visible fields, stores observation snapshots locally, and ranks candidates with transparent, fixed-weight heuristics.

This project **does not use AI or machine learning**, does not forecast future sales, and does not guarantee product-selection or profit outcomes.

### Features

- User-initiated scrolling and collection on listing/store pages
- Parsing for price, sold count, daily/weekly sales, local-warehouse labels, and ad labels
- Up to 40 local history snapshots per product for simple trend metrics
- Product-detail enrichment and optional store-page follow-up
- Filters for price, minimum daily sales, local warehouse, and non-ad items
- Explainable product/store scores and a Top-5 candidate list
- Product and store CSV export
- Local-only storage through `chrome.storage.local`; no remote upload logic in the codebase

### Install and use

1. Download or clone this repository.
2. Open `chrome://extensions/` or `edge://extensions/`.
3. Enable Developer mode and choose **Load unpacked**.
4. Select the `extension/` directory.
5. Open or refresh a Temu page, open the extension, configure filters, and start a scan.
6. Stop at any time, inspect the candidates, or export CSV for manual analysis.

### Architecture

`content.js` reads the current page DOM, parses and deduplicates records, computes snapshot-based metrics, and writes them to local extension storage. `popup.js` reads that local data, applies filters and rule-based scores, renders the Top 5, and exports CSV files. No backend service is used.

### Responsible use

The extension depends on Temu's current DOM and may break when the site changes. Automated scrolling and seller-entry clicks may be governed by website terms, account policies, robots rules, and local law. Use low frequency, do not bypass access controls or CAPTCHAs, and only collect data you are authorized to access. See [`docs/SECURITY_AND_COMPLIANCE.md`](docs/SECURITY_AND_COMPLIANCE.md).

### License status

No open-source license is included yet because the code originated in an internship/business context and ownership should be confirmed first. Public visibility does not grant reuse rights. See [`RIGHTS.md`](RIGHTS.md).
