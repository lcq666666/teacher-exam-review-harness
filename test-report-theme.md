# 主题切换 + A/C 视觉落地 · 测试报告（2026-11-14）

范围：v3 分册手册（方向 A）+ 红笔版（方向 C）+ 共用主题切换模块。功能、诊断逻辑、演示数据、演示数字均未改动，断言金标准仍是 KPI 30/10/299/58.5%、分层 30/81/12（24.4/65.9/9.8%）、提纲 7 条。

## 1. Node / Python 全量测试（最终产物上跑过，全部通过）

| 测试 | 结果 | 新增内容 |
|---|---|---|
| `hidden_files/run_test_pass.py` | **36/36**，失败：无；T-10 golden 2.22e-16；T-24g golden（部分得分）2.22e-16 | T-33：切换器存在，三档切后 data-theme 与 localStorage 一致，非法值回落到 system；T-34：深色 token 里底色/卡片不是纯黑 |
| `hidden_files/test_v3.py` | **67/67** | TH-1~TH-8：防闪烁脚本在 `<style>` 之前读 examReviewTheme；`__theme` 模块已装进页面；三段切换器（浅色/深色/跟随系统，aria-pressed）；显式 `[data-theme]` token 块；显式深色三组对比度 ≥ 4.5:1；淡彩标签（非当前步 tint、当前步满色）；H1 34px；head-row + 切换器 |
| `hidden_files/test_redpen.py` | **44/44** | TH-1~TH-8：防闪烁脚本；`__theme` 模块；下拉切换器（themeMenuBtn + themeMenuList + menuitemradio + aria-checked）；显式 `[data-theme]` token 块；显式深色 `--bg #191919`；时间线（圆圈数字 + 连线 + 步名/小字）；大标题句式 + 红字只在 accent；H1 44px |
| `hidden_files/test_v3_vm.js`（v3） | 全部通过 | THEME 段：默认 system；切深/浅写 localStorage + data-theme；非法值回落；系统深色时 effective=dark |
| `hidden_files/test_v3_vm.js`（红笔版，MUSE_HTML=files/redpen-product.html） | 全部通过 | 同上 |
| `hidden_files/test_v1_core.py` | 全部通过 | 加载器加了 `__theme`（模块在无 documentElement 的桩里保持 inert） |
| `hidden_files/test_kc_tagger.js` | 通过 | 演示库 10/10，高中库抽查 15/16，边界全部通过 |
| `hidden_files/test_llm_tagger.js` | 通过 | 提示词、解析、分批调用、补位、Key/模型校验全部通过 |
| `hidden_files/test_cuojuan.js` | 25 个断言通过 | 泛化抽查 2/3（G2“化简”被字面命中为计算失误，已知局限） |
| `hidden_files/test_word_export.js` | 36 个断言通过 | — |
| `hidden_files/test_theme.js` | **7/7** | 默认 system + 浅色系统；system + 深色系统；setTheme 深色；setTheme 浅色 + 非法值回落；点击接线；已存深色加载；无 documentElement 时 inert 加载 |

说明：`test_v3.py` / `test_redpen.py` 的 `remove_blocks` 已更新，会剥掉新增的 `:root[data-theme="light"/"dark"]` 块，否则 A1-2/R1-2（token 定义之外无硬编码色值）会误报。`run_test_pass.py` 生成的报告写在 `files/test-report-cuojuan.md`（Node 可跑部分 36/36，含 T-33/T-34 行）；`files/test-report-p2.md` 是 2026-11-13 的旧文件，不再更新。

## 2. 无头浏览器验收（本地 file://，chrome-headless-shell + playwright-core）

| 脚本 | 结果 |
|---|---|
| `~/workspace/tools/pwcheck/check-theme.js`（新写） | 两版都通过：浅色系统默认浅色底（v3 `rgb(246,241,228)`、红笔 `rgb(251,250,248)`）；点深色 → `data-theme=dark`、localStorage=`dark`、底变深（v3 `rgb(23,20,15)`、红笔 `rgb(25,25,25)`），刷新后保持；点浅色同理，刷新后保持；点跟随系统 → 存 `system`；全新深色系统上下文默认就是深色底；手机 390 浅/深都不溢出；**零页面错误** |
| `check-v3.js` | 全流程数字不变：KPI 30/10/299/58.5%、分层 24.4/65.9/9.8、图例 30/81/12、提纲 7 条；手机 390 不溢出；深色底 `rgb(23,20,15)`；打印验收通过 |
| `check-redpen.js` | 同上，数字一致；深色底 `rgb(25,25,25)`；打印验收通过 |
| `check-cuojuan.js`（v3 浅色） | 典型错法进统计/诊断/提纲，`page errors: none` |
| `check-cuojuan-dark.js`（v3 深色，新增变体） | 同上，`page errors: none` |
| `check-cuojuan.js`（红笔版浅色，MUSE_HTML + MUSE_TAG=cj-rp） | 同上，`page errors: none` |
| `check-cuojuan-dark.js`（红笔版深色，MUSE_TAG=cj-rp-dark） | 同上，`page errors: none` |

## 3. 首屏截图（四张，`~/workspace/tools/pwcheck/`，已逐张看过）

- `theme-v3-light.png`：暖纸底、淡彩标签（第 1 步满色橙 + 墨色环）、右上角三段切换器（跟随系统激活）。
- `theme-v3-dark.png`：暖深底 `#17140f`、深底淡彩标签、切换器深色激活，文字清晰。
- `theme-redpen-light.png`：大标题“讲评，从一张成绩表**开始**”（开始为红）、时间线五步（第 1 步橙圈 + 光晕）、下拉切换器“☀ 浅色 ▾”、留白充足、按钮 pill 形。
- `theme-redpen-dark.png`：`#191919` 底、白字标题 + 红“开始”、时间线清晰、切换器显示深色。

## 4. Impeccable detect

- v3：**2 条**，都是已知打印块误报（打印强制 `#000` 字，detector 把它和深色底 `#221e17` 配对；实际打印是白底）。
- 红笔版：**4 条**——2 条同款打印块误报；另 2 条是跨主题误配对（深色字色 `#a3a3a3 / #f5f5f5` 配浅色底 `#ffffff`，同一主题里从不同时出现；主题内对比度已由静态测试逐项验过 ≥ 4.5:1）。detector 不按主题分开算，记为已知误报。
- 修掉的一条真问题：detect 曾报 v3 切换器“子元素贴边”（cramped-padding），已把容器内边距改成 4px、按钮改成 28px 高 pill，复跑后消失。

## 5. 没变的东西（回归确认）

- 功能、诊断引擎（NSKILL=123）、演示数据、演示数字、LLM 标注、KC 库、典型错卷输入都没动；演示数字在 Node 测试和无头浏览器两边都对上了。
- 没写任何 API Key 进文件或测试；没碰 `deploy/` 和 `deploy-redpen/`，没推 GitHub。
- 方向 B（克制仪表盘）只留在 `files/theme-research-20261114.md` 里备用，没删。
