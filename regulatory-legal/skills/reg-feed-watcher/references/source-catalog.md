<!--
This file is a Chinese translation of the original by Anthropic PBC.
Original: https://github.com/anthropics/claude-for-legal
Licensed under Apache License 2.0
-->

# 监管信息源目录

reg-feed-watcher 的起始目录。冷启动访谈配置要监控哪些信息源；本目录提供可选项。URL 经 **2026 年 5 月**验证——信息源 URL 会变更；如果某个信息源停止返回结果，请进行验证。

**如何阅读本目录：**
- **Format（格式）** — 信息源返回的内容：JSON API（结构化，最佳）、RSS/Atom（半结构化，良好）、HTML 页面（需要爬取或变更检测）、仅邮件（需要 Gmail/Outlook MCP）。
- **Tier（层级）** — *Primary（主要）* 指监管机构本身；*Secondary（次要）* 指总结主要来源的评论员、聚合器或律所。将次要来源视为权威之前，务必追溯到主要来源。
- **Auth（认证）** — None 表示开放；Key 表示免费但需注册的 API 密钥；Paid 表示订阅制。
- **Notes（备注）** — 任何注意事项（速率限制、信息源停用、发现步骤）。

标有 ⚠️ 的信息源已被用户或监管机构报告为不可靠或已停用——配置前请验证。

---

## 美国联邦 — 主要来源

| 信息源 | Feed URL | 格式 | 覆盖内容 | 认证 | 备注 |
|---|---|---|---|---|---|
| Federal Register | `https://www.federalregister.gov/api/v1/documents.json` | JSON API | 所有联邦规则、拟议规则、通知、总统文件 | None | 通过 `conditions[agencies][]=<slug>`、`conditions[publication_date][gte]=<YYYY-MM-DD>`、`conditions[type][]=RULE\|PRORULE\|NOTICE\|PRESDOCU` 过滤。文档完善：federalregister.gov/developers/documentation/api/v1。返回摘要、生效日期、评论截止日期、引用。**优先使用此来源**——大多数联邦机构文件均通过此处发布。 |
| Regulations.gov | `https://api.regulations.gov/v4/documents` | JSON API | 规则制定案卷、公众评论、支持文件 | Key（免费） | 密钥获取：open.gsa.gov/api/regulationsgov/。用于案卷级追踪和提取评论。 |
| Congress.gov | `https://api.congress.gov/v3/bill` | JSON API | 联邦法案、法律、委员会报告 | Key（免费） | 密钥获取：api.congress.gov/sign-up。congress.gov/rss 也提供预建 RSS（范围较窄：提交总统的法案、浏览量最高、楼层）。 |
| SEC Press Releases | `https://www.sec.gov/news/pressreleases.rss` | RSS | 规则、执法、演讲（仅新闻稿） | None | SEC RSS 中心：sec.gov/about/rss-feeds。另有 EDGAR 结构化申报信息源：sec.gov/structureddata/rss-feeds（工作时间每 10 分钟更新）。已通过的规则新闻通常也会发布到 Federal Register——注意去重。 |
| FTC Press Releases | `https://www.ftc.gov/feeds/press-release.xml` | RSS | 执法、规则、博客帖子、和解 | None | 按主题子信息源：`ftc.gov/feeds/press-release-consumer-protection.xml`、`press-release-competition.xml`，博客信息源 `ftc.gov/feeds/business-blog.xml`。信息源中心：ftc.gov/news-events/stay-connected/ftc-rss-feeds。 |
| CFPB Newsroom | `https://www.consumerfinance.gov/about-us/newsroom/` | HTML + 页面上的 RSS 选项 | 规则、执法、通告、博客 | None | 页面提供 RSS 订阅；`consumerfinance.gov/activity-log/` 上的活动日志是范围最广的单一 URL。主要规则也会出现在 Federal Register。 |
| DOJ Antitrust Division | `https://www.justice.gov/atr/news-feeds` | RSS（多个信息源） | 新闻稿、演讲、利益声明 | None | 页面按内容类型列出多个 Atom/RSS URL。DOJ 主新闻稿信息源在 justice.gov（从 `justice.gov/news` 导航）。 |
| DOJ Main | `https://www.justice.gov/news/rss` | RSS | 所有 DOJ 各部门新闻稿 | None | 按主题在客户端过滤。民事部门、ATR、民权部门均包含在内。 |
| FCC Daily Digest | 通过 `fcc.gov/news-events/rss-feeds-and-email-updates-fcc` 订阅 | RSS + 邮件 | 命令、通知、公告 | None | 另有 ECFS 案卷专用信息源——从"热点案卷"中选择案卷，右键点击 RSS 图标。 |
| HHS OCR | `https://www.hhs.gov/ocr/newsroom/index.html` | HTML | HIPAA 执法、和解、指导意见 | None | 未找到直接 RSS URL；`hhs.gov/rss` 上的 HHS 全站新闻稿信息源涵盖 OCR。在 X 上关注 `@HHSOCR` 可获得推送提醒。 |
| OFAC Recent Actions | `https://ofac.treasury.gov/recent-actions` | HTML + 邮件 | 制裁认定、一般许可证、FAQ | None | ⚠️ RSS 已于 2025 年 1 月 31 日停用。邮件是受支持的推送渠道——在 service.govdelivery.com/service/multi_subscribe.html?code=USTREAS 订阅。页面有可浏览的列表。 |
| BIS（商务部） | `https://www.bis.gov/news-updates` | HTML | 出口管制更新、实体名单、最终规则 | None | `bis.gov/regulations/federal-register-notices` 上的 Federal Register 通知索引是最清晰的列表。未找到公开 RSS。 |
| DOL News Releases | `https://www.dol.gov/rss/releases.xml` | RSS | 工资/工时、OSHA、OFCCP、EBSA 新闻稿 | None | 其他信息源索引见 `dol.gov/rss`。 |
| NIST Cybersecurity | `https://www.nist.gov/news-events/cybersecurity/rss.xml` | RSS | 按主题的网络安全新闻 | None | AI/博客信息源：`nist.gov/blogs/cybersecurity-insights/rss.xml`。 |
| CISA Alerts/Advisories | `https://www.cisa.gov/news-events/cybersecurity-advisories` | HTML + RSS 选项 | ICS 建议、警报 | None | 在页面验证信息源 URL；多个按内容类型的子信息源。 |

---

## 美国州级 — 主要来源

覆盖范围不均。此处优先收录具有积极隐私/消费者保护执法的州。许多州监管机构仅发布 HTML 页面——如无 RSS，配置为"手动"或设置网页变更检测。

| 信息源 | Feed URL | 格式 | 覆盖内容 | 认证 | 备注 |
|---|---|---|---|---|---|
| California AG | `https://oag.ca.gov/news/feed/729/oag.ca.gov` | RSS | 新闻稿、CCPA 执法、多州行动 | None | 主新闻页：`oag.ca.gov/media/news`。 |
| California Privacy Protection Agency（CPPA） | `https://cppa.ca.gov/announcements/` | HTML | CCPA 法规、执法、建议 | None | ⚠️ 未找到直接 RSS URL——主要渠道为邮件列表（在页面注册）。监控页面变更或使用手动输入。 |
| New York AG | `https://ag.ny.gov/press-releases` | HTML | 新闻稿、多州 AG 行动 | None | ⚠️ 未找到公开 RSS。`ag.ny.gov/press-releases-for-month` 的月度存档结构足够清晰，可进行爬取。 |
| Texas AG — News Releases | `https://www2.texasattorneygeneral.gov/feeds/feeds.php?feed=pr` | RSS | 新闻稿 | None | 其他信息源见 `www2.texasattorneygeneral.gov/agency/feeds`。 |
| Illinois AG | `https://illinoisattorneygeneral.gov/news-room/` | HTML | 新闻稿 | None | ⚠️ 未找到公开 RSS。 |
| Washington AG | `https://www.atg.wa.gov/news` | 页面上的 RSS 选项 | 最新新闻、AGO 意见、消费者提醒 | None | 新闻、意见、消费者提醒有单独信息源——从页面订阅。 |
| Colorado AG | `https://coag.gov/press-releases/` | HTML | 新闻稿、CPA 规则制定 | None | ⚠️ 未找到公开 RSS。Colorado Privacy Act 规则制定也通过 SOS 发布。 |
| Connecticut AG | `https://portal.ct.gov/ag/press-releases/press-releases` | HTML | 新闻稿 | None | ⚠️ 未找到公开 RSS。 |
| Virginia AG | `https://www.oag.state.va.us/media-center/news-releases` | HTML | 新闻稿、VCDPA 监督 | None | ⚠️ 未找到公开 RSS。 |
| Massachusetts AG | `https://www.mass.gov/orgs/office-of-attorney-general-maura-healey/news` | HTML | 新闻稿 | None | ⚠️ 未找到公开 RSS。Mass.gov 为各机构提供新闻室页面。 |
| NYDFS | `https://www.dfs.ny.gov/reports_and_publications/press_releases` | HTML | 执法、法规、网络安全（Part 500） | None | ⚠️ 未找到公开 RSS。 |

---

## 欧盟 / 英国 — 主要来源

| 信息源 | Feed URL | 格式 | 覆盖内容 | 认证 | 备注 |
|---|---|---|---|---|---|
| EDPB News | `https://www.edpb.europa.eu/news/news_en` | RSS（提供 2 个信息源） | 指南、意见、执法摘要、约束性决定 | None | 信息源链接见 `edpb.europa.eu/sme-data-protection-guide/faq-frequently-asked-questions/answer/how-can-i-keep-edpbs-work_en`。 |
| European Commission Press Corner | `https://ec.europa.eu/commission/presscorner/` | RSS + 邮件 | 新闻稿、演讲、问答——DSA、DMA、AI 法案实施法令 | None | 在 `ec.europa.eu/commission/presscorner/login/en` 订阅。按主题的较窄子信息源。 |
| EUR-Lex（OJ） | `https://eur-lex.europa.eu/` | 网络服务 + 按搜索的 RSS | 官方杂志出版物 | Key（免费，网络服务） | 用于追踪最终形式的法规和指令。 |
| ICO（英国） | `https://ico.org.uk/global/rss-feeds/` | RSS（多个信息源） | 执法、指导意见、新闻、咨询 | None | 新闻、执法行动和博客有单独信息源。执法列表也见 `ico.org.uk/action-weve-taken/enforcement/`。 |
| CNIL（法国） | `https://www.cnil.fr/en/rss.xml`（验证——feeder.co 对此进行了索引） | RSS | 法国 DPA 决定、指导意见、制裁 | None | 英文新闻见 `cnil.fr/en/news`。第三方索引表明信息源存在；依赖前请验证。 |
| DPC（爱尔兰） | `https://www.dataprotection.ie/en/news-media/latest-news` | HTML | 调查、决定、指导意见——大多数美国科技公司的牵头 DPA | None | ⚠️ 未找到公开 RSS。对于针对美国公司的 GDPR 执法至关重要；值得设置变更检测或邮件订阅。 |
| BfDI（德国） | `https://www.bfdi.bund.de/EN/Home/home_node.html` | HTML | 德国联邦 DPA | None | ⚠️ 未找到公开 RSS。 |
| ENISA | — | 邮件 | 网络安全、NIS2 指导意见 | None | ⚠️ **RSS 信息源已随新网站停用。** 仅邮件提醒，直到新订阅机制上线（`enisa.europa.eu/rss-feeds-discontinued-new-subscription-mechanism-coming-soon`）。 |
| FCA（英国） | `https://www.fca.org.uk/news/rss.xml`（验证） | RSS + 邮件 | 英国金融服务规则、执法、警告 | None | `fca.org.uk/newsletters-emails-sign-up` 上的邮件提醒是受支持的渠道；历史上也提供 RSS。 |
| EDPS | `https://www.edps.europa.eu/press-publications/press-news_en` | HTML + RSS 选项 | 欧盟机构 DPA | None | |

---

## 国际

| 信息源 | Feed URL | 格式 | 覆盖内容 | 认证 | 备注 |
|---|---|---|---|---|---|
| OECD AI Policy Observatory | `https://oecd.ai/en/` | HTML + 通讯 | 各国 AI 政策、OECD 指导意见 | None | 最适合追踪非欧盟、非美国的 AI 规则制定。 |
| Council of Europe | `https://www.coe.int/en/web/portal/news` | RSS + HTML | 欧洲理事会条约，包括 AI 框架公约 | None | |
| UK Parliament Bills | `https://bills.parliament.uk/rss/publicbills.rss`（验证） | RSS | 英国法案 | None | |

---

## 次要来源 / 聚合器

**将这些来源的内容视为线索，而非权威。** 次要来源说"FTC 发布了 X"意味着：在 ftc.gov 上找到 X，然后再依赖它。在摘要中将从这些信息源获取的条目标记为 `[次要来源]`。

| 信息源 | Feed URL | 格式 | 覆盖内容 | 认证 | 备注 |
|---|---|---|---|---|---|
| IAPP Daily Dashboard | `https://iapp.org/rss/daily-dashboard/` | RSS | 全球隐私和 AI 治理新闻，经策划 | None（部分内容付费墙） | 隐私团队信噪比最高。 |
| Future of Privacy Forum | `https://fpf.org/feed/` | RSS（WordPress） | 隐私评论、州法律追踪器、报告 | None | |
| Hogan Lovells | `https://www.hoganlovells.com/en/rss` | RSS（按执业领域分类） | 客户提醒、业务 | None | 提供按执业领域的子信息源。 |
| Covington & Burling | `https://www.cov.com/`（按博客验证） | 按博客的 RSS | InsidePrivacy、Global Policy Watch、Inside Global Tech、Inside Tech Media | None | 每个主题博客都是 WordPress 风格的网站，有标准的 `/feed` 端点。 |
| WilmerHale | `https://www.wilmerhale.com/` | 邮件 / HTML | 客户提醒 | None | ⚠️ 未找到综合公开 RSS；邮件订阅是主要渠道。 |
| Wilson Sonsini | `https://www.wsgr.com/` | 邮件 / HTML | 客户提醒 | None | ⚠️ 未找到综合公开 RSS。 |
| Lexology | `https://www.lexology.com/account/rss` | RSS（按主题/司法管辖区可定制） | 聚合律所提醒 | 账户（免费） | 功能强大：按主题+司法管辖区构建信息源。由 LBR 拥有。 |
| JD Supra | `https://www.jdsupra.com/legal-news/rss-law-feeds.aspx` | RSS（按主题分类） | 聚合律所提醒 | None | 比 Lexology 覆盖更广但噪音更多。 |
| Artificial Lawyer | `https://www.artificiallawyer.com/feed/` | RSS | 法律科技 / AI 监管新闻 | None | |
| LawSites（Bob Ambrogi） | `https://www.lawsitesblog.com/feed` | RSS | 法律科技，也涵盖法律 AI 监管 | None | |

---

## 无信息源的来源（需要网页监控或邮件）

一些重要来源不发布信息源，或其 RSS 已停用。监控它们需要以下之一：
- 网页变更检测（当前未内置）
- 邮件通讯转发（需要 Gmail/Outlook MCP 集成）
- 通过 reg-feed-watcher 的"手动输入"路径进行手动检查

| 信息源 | URL | 备注 |
|---|---|---|
| OFAC Recent Actions | `https://ofac.treasury.gov/recent-actions` | RSS 已于 2025 年 1 月停用；邮件是受支持渠道 |
| ENISA | `https://www.enisa.europa.eu/news` | RSS 已停用；新订阅机制待定 |
| DPC Ireland | `https://www.dataprotection.ie/en/news-media/latest-news` | 无 RSS；对 GDPR 执法至关重要 |
| CPPA | `https://cppa.ca.gov/announcements/` | 仅邮件列表；未找到 RSS |
| 大多数州 AG（NY、IL、CO、CT、VA、MA） | 见上方州级表格 | 新闻稿 HTML 页面；无 RSS |
| NYDFS | `https://www.dfs.ny.gov/reports_and_publications/press_releases` | 仅 HTML |
| BIS（商务部） | `https://www.bis.gov/news-updates` | 仅 HTML；使用 Federal Register API 获取规则级事件 |
| HHS OCR 独立页面 | `https://www.hhs.gov/ocr/newsroom/` | 包含在 HHS 全站 RSS 中，但无 OCR 专用信息源 |
| BfDI（德国） | `https://www.bfdi.bund.de/EN/` | 仅 HTML |
| WilmerHale、Wilson Sonsini | 律所网站 | 邮件订阅是主要渠道 |

---

## 建议的入门套餐

**以美国+欧盟为重点的内部隐私团队：**
Federal Register（FTC、HHS/OCR 机构过滤器）、FTC RSS、CFPB、CA AG、CPPA（邮件）、
NY AG（页面监控）、EDPB、ICO、CNIL、DPC Ireland（页面监控）、IAPP、FPF。

**商业/监管内部团队（宽泛覆盖）：**
Federal Register（所有关注机构）、SEC RSS、CFPB、DOJ Antitrust、DOJ
Main、FCC、DOL、BIS 页面监控、OFAC 邮件、European Commission Press Corner、
FCA。添加 IAPP + Lexology 以获得聚合器覆盖。

**AI 治理团队：**
Federal Register（过滤：FTC、HHS、NIST、商务部）、NIST Cybersecurity RSS、EU
Commission Press Corner、EDPB、OECD AI Observatory、Council of Europe、IAPP、FPF、
Artificial Lawyer、CA AG（ADMT）、CPPA。

---

## 添加信息源

要添加本目录中没有的信息源：
1. 找到信息源 URL（尝试 `/rss`、`/feed`、`/news.rss`，或查看页面源代码中的 `<link rel="alternate" type="application/rss+xml">`）。
2. 在浏览器中或使用 `curl` 验证它返回 XML/JSON。
3. 在用户的 regulatory-legal CLAUDE.md 中的 **信息源配置 → 直接监管机构信息源** 下添加：信息源名称、URL、格式、覆盖内容。
4. 如果不存在信息源，在 **无信息源的来源** 下添加，并决定：手动、邮件还是变更检测。
