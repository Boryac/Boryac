<div align="center">

<img src="https://raw.githubusercontent.com/Boryac/Boryac/main/assets/banner.png" alt="BORV — boryac.me" width="100%" />

**黄博弈 · BORV** — AI × 商务拓展

把检索和系统做出来，也把内容和增长跑起来。

[boryac.me](https://boryac.me) · [yikang.boryac.me](https://yikang.boryac.me) · [18 条档案](https://boryac.me)

</div>

---

## 一、在做的三件事

### 1 · 检索与应答系统：宁可不答，不硬编

面向美团度假与闲鱼来客的多平台客服场景，三个月攒下 **1.4 万条**工单，可用语料只有这些；平台不允许系统直连订单库，客服也不接受把不确定的答复发给客人。

- 工单做成语义索引 → 检索 → 重排 → 每条候选打分，阈值定在 **0.62**
- 离线集 **n=800** 上准确率 **92.4%**，关键词基线 **78.1%**
- 上线首 6 周承接 **75%** 入站咨询，人工复核从每天 **40 条**降到 **12 条**，误答投诉 **3 起**
- 阈值上下都试过：下调到 **0.5** 误答率到 **9%**，抬到 **0.7** 拒答率过 **20%**

> 一次真实失败：最初让模型直接生成答复，两周内它在退改规则上把两个平台的条款混在一起，误答率从 **4%** 抬到 **11%** —— 整版下线，改成答案只从历史语料取、模型只做召回与排序，找不到就拒答。这一段写在 [X-001 档案](https://boryac.me) 里。

技术栈：Python / 全文与向量混合检索 / Cross-Encoder 重排 / 阈值拒答 / 降级分支。

### 2 · 产品工程：一套后端，三个端

[弈康 YiKang](https://yikang.boryac.me) 是面向家庭健康信息整理的导诊系统：一个 FastAPI 服务端同时支撑 Windows 桌面端与微信小程序，三端共用 1 套接口契约。

- 桌面端冷启动 **1.69 s**，安装包 **114 MB**（三次打包裁剪，去掉 2 个冗余依赖）
- **479 项测试**覆盖检索、降级与权限分支，回归每 2 周一轮
- 无密钥时进入降级状态并**如实标注**，四段问答仍能跑通
- 医务科看板实测：平均对话 **112.1 分钟**、知识库命中率 **85.7%**、转人工率 **7.4%**、首字延迟 **5,673 ms**、吐字 **77.0 字/秒**（63 轮次 / 128 次采样）

> 另一次失败：早期给回答加了自动转诊建议，试跑 3 天它把普通感冒推到呼吸科专家号，**26 例里 7 例导错科室**。整版下线，改成只给科室范围与就诊准备，是否转诊由导诊台医生决定。

技术栈：Electron + Vue 3 + FastAPI / 权限切 8 类角色 / 草稿→审核→生效三段知识库 / RAG 降级设计。

### 3 · 增长与内容：先建基线，再谈优化

- **县域商户基线盘点**：扫街拜访 **80 家**餐饮与民宿（**5 个片区**、每户平均 2 次回访），用一张 **21 字段**的盘点表记录线上曝光、点评内容与核销方式。字段从首版 40 多个砍到 21，缺项率从 **52%** 降到 **5%**；据此把商户分成三类，签约 **21 家（26%）**，首月后续约 **13 家**，续约率 **62%**。
- **内容账号运营**：一年发布 **60 篇**，单篇最高阅读 **3.2 万**，平均收藏率 **8.7%**（n=60），涨粉 **1,900**，零付费推广。60 篇里带清单的 **12 篇**吃掉了 **40%** 的收藏；招聘季前后阅读量是平时的 **2 倍多**。
- **GTM 打包**：把县域文旅资源做成可售卖产品，以党建团建与文化体验切入，联动景区与酒店议价锁定资源。

技术栈：RFM 分层 / 漏斗与线索分级 / 抽样盘点方法论 / 内容选题机制。

---

## 二、写在代码里的东西

### boryac.me 构建管线（本项目）

一条命令 `node tools/make.mjs` 跑完 `plates → build → assets → fonts → perf`，**每次输出逐字节一致**。

| 指标 | 数值 | 怎么验 |
|---|---|---|
| 档案条目 | **18** 条（`X-001`～`X-018`） | `ls archives/ \| wc -l` → 18 |
| 示例图版 | **40** 张（`media/xNNN/NN.jpg`，2000×1250 JPEG） | `tools/media/pages.json` 求和 |
| 影像通道 | **2** 条 H.264 720p：`x017.mp4` 4,464,052 B / `x018.mp4` 2,475,204 B | `ls -l videos/` |
| 首屏 HTML | **57,818 B**（gzip 传输） | `curl -H 'Accept-Encoding: gzip' -sI https://boryac.me` |
| 关键路径合计 | **629.6 KiB**（HTML + JS + CSS + 3 字体，全 gzip） | 6 个请求 `size_download` 求和 |
| SW 预缓存 | **9** 个文件（图片/字体 runtime 分级，视频不进缓存） | `sw.js` 的 `CORE` 数组 |
| 字体切片 | `@font-face` **100** 条（与 CSS 文件数 1 对齐，未超上限） | `grep -c @font-face assets/*.css` |
| 构建脚本 | **23** 个 `.mjs` + 211 行 `plates.py`，共 **3,424** 行 | `find tools -name '*.mjs'` |

**安全基线**（线上实测响应头，不是配置意图）：

```
Content-Security-Policy: default-src 'self'; img-src 'self' data: blob:; font-src 'self';
  frame-ancestors 'none'; base-uri 'self'; form-action 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=(), interest-cohort=()
```

全站零第三方请求：字体、图标、脚本、图片全部自托管。

### 其它仓库

| 仓库 | 语言 | 文件数 | 是什么 |
|---|---|---|---|
| [autobookforge](https://github.com/Boryac/autobookforge) | Python 437 KB / HTML 111 KB | **97** | TXT 精排电子书生产引擎：编码检测、章节识别、双纠错、EPUB3/MOBI/AZW3 输出，**规则驱动、零 LLM API 依赖**。本地 11,972 行 Python |
| [boryac-me](https://github.com/Boryac/boryac-me) | JavaScript 122 KB / Python 9 KB | **48** | 本页背后的构建管线与 18 条档案数据 |
| [cognitivecomputing-firm](https://github.com/Boryac/cognitivecomputing-firm) | Python 157 KB | **69** | 公司化多角色工作流技能：每次输入登记工单，按职能/角色/流程/门禁/风格法执行，产出可审计、可回滚。**G0–G11 门禁 + Red Team 否决权** |
| [PinHaoTUpyEasy](https://github.com/Boryac/PinHaoTUpyEasy) | Python 39 KB | — | 反色拼图小工具：分块分散 + 混合叠加（Screen / Multiply） |
| [tabler](https://github.com/Boryac/tabler) · [tremor](https://github.com/Boryac/tremor) · [LibreChat](https://github.com/Boryac/LibreChat) | fork | — | 上游开源项目的中文后台适配与界面取证，本地化改动集中在 `docs/zh-CN.md` 与 `extra/`，**保留上游许可证与版权声明**，便于随上游 rebase |

---

## 三、工作方式

我做事的顺序是：**先建可测量的基线，再改，改完用同一把尺子复测**。写进流程的四条：

1. **数字必须有出处** —— 图上、文档里、README 里的每个数字都要能追到源文件或命令输出；追不到就删掉，而不是改小。
2. **门禁要 fail-closed** —— 测不出来按失败处理，不许"无法测量"静默变成 PASS。这条吃过亏：对比度检查器曾把解析不了的色值记为"未验证"，整站违规全部消失，规则误报通过。
3. **幂等与可复现** —— 构建两次产物逐字节一致；截图关动画、固定 DPR 与 locale、等字体就绪。
4. **隐私先于发布** —— 上传前跑三道审计：密钥/凭据、本地路径与内部代号、PII 与高熵串。图片里的第三方 logo、真实姓名、访客 IP 一律打码或换图，且不可逆。

---

## 四、联系

写邮件比打电话更快：**huangboryac@gmail.com**

许可：本仓库构建脚本以 MIT 发布。页面基底与美术资源来自 [KIRAHOOK-CV-Web](https://github.com/kirahook/KIRAHOOK-CV-Web)（MIT），已保留原始版权声明与站内致谢；第三方字体按其自身许可随包分发，未做二次修改。

<div align="center"><sub>© 2027 黄博弈 · BORV</sub></div>
