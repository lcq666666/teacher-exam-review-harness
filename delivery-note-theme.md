# 主题切换 + A/C 视觉落地 · 交付说明（2026-11-14）

结论先说：两个版本都按用户定的方向落完了——v3 分册手册用 A（纸上分册浅色），红笔版用 C（讲台白板），主题切换两版共用一套模块，三档都能点、刷新后保持。全部测试通过，演示数字一个没变。**还没推 GitHub**，等用户看完下面四张首屏截图确认后再推。

## 一、v3 分册手册 · 方向 A 落点

- 浅色默认：暖纸底 `#f6f1e4`、白卡片 `#fdfbf3`，正文 `#1f2937`，不再是以前的深色底。
- 五色退成淡彩标签：非当前步是淡彩底 + 深色字（橙 `#fbe9df/#c2410c`、黄 `#fdf3cd/#854d0e`、绿 `#eef3df/#4d7c0f`、青 `#e2efec/#0f766e`、蓝 `#e4ebfb/#1d4ed8`）；只有当前步是满色 + 墨色描边环，往右伸出 12px。
- 页头右上角三段切换器（浅色 / 深色 / 跟随系统），当前档是深色实心 pill。
- 深色版按调研报告的深色 token 做（底 `#17140f`、卡片 `#221e17`、淡彩标签是深底亮字），不是浅色直接反色，底色不是纯黑。
- 字阶放大：H1 34px、H2 21px；内容宽 1080px；卡片内边距 28/32px。

## 二、红笔版 · 方向 C 落点

- 大标题“讲评，从一张成绩表**开始**”（“开始”两个字是红笔红），上面一行小字 eyebrow“教师讲评助手 · 红笔版”；页头不再是卡片条，大量留白。
- 五步排成横向时间线：圆圈里是步号，细线相连；当前步是橙色圆圈 + 光晕（`--accent #c2410c`），做完的步是红底白勾；圆圈下是步名 + 小字说明（成绩表 · 粘贴 / OCR · 标注 / 正确率 · 分层 / 知识点掌握度 / 重点讲 · 导出）。步骤语义没变，还是五步。
- 红笔身份没丢：红字只出现在标题 accent 和批改标记上；重点讲标题下的红波浪线、做完的步红框红勾、偏弱分段红色都保留。别处不用红。
- 页头右上角下拉式切换器（“☀ 浅色 ▾”，点开三档，当前档打勾），圆角 16px + 柔和阴影；按钮全改成 pill 形；内容宽 1120px；卡片内边距 32/36px。
- 深色版是 Notion 式深色（底 `#191919`、卡片 `#202020`、正文 `#f5f5f5`、红 `#cd4436`）。

## 三、主题切换实现要点

- 共用模块 `hidden_files/theme.js`（`globalThis.__theme`）：`KEY = 'examReviewTheme'`，值 `'light' / 'dark' / 'system'`；`getTheme / setTheme / effective / applyTheme / initTheme / syncUI`。
- 默认跟随系统（localStorage 里没值时就是 system），点一次后记住；跟随系统档监听 `matchMedia('(prefers-color-scheme: dark)')` 变化，系统切了页面跟着变。
- `<head>` 内联小脚本在 CSS 之前读 localStorage 设 `data-theme`，刷新不闪错主题；`build_v1.py` 用 `MUSE_THEME_VARIANT`（segmented / dropdown）、`MUSE_HEADLINE`、`MUSE_EYEBROW` 控制两版的切换器形态和标题。
- CSS 是四块 token：`:root`（浅色）→ `@media (prefers-color-scheme: dark)`（深色）→ `:root[data-theme="light"]` → `:root[data-theme="dark"]`（最后，显式选择优先于系统）。
- 无障碍：三段版 `role="group" aria-label="主题切换"` + `aria-pressed`；下拉版 `aria-haspopup="menu"` + `menuitemradio` + `aria-checked`，点外面或按 Esc 关菜单。
- 对比度：两套主题正文、正文/弱化字、主按钮字都 ≥ 4.5:1（静态测试逐项算过，见测试报告）。
- 打印样式保持可用：打印强制浅色白底（`@media print` 里 `body { background: #fff; color: #000 }`），不把深色底印出来。
- 覆盖全部五步：上传、试卷处理、统计、诊断、提纲的卡片、表格、按钮、KPI、分层色条、提纲徽标都走新 token；功能、诊断逻辑、演示数据、演示数字一个字没动。

## 四、重建产物

- `files/v1-product.html`：**1,060,629 字节**（v3 · 方向 A）
- `files/redpen-product.html`：**1,060,099 字节**（红笔版 · 方向 C）
- 构建命令：v3 用默认环境变量；红笔版用 `MUSE_STYLE=hidden_files/redpen-style.css MUSE_OUT=files/redpen-product.html MUSE_TITLE='教师讲评助手 · 红笔版' MUSE_THEME_VARIANT=dropdown MUSE_HEADLINE='讲评，从一张成绩表<em class="hl">开始</em>' MUSE_EYEBROW='教师讲评助手 · 红笔版'`。
- **没碰 `deploy/` 和 `deploy-redpen/`，没推 GitHub。** 等用户看图确认后再推。

## 五、首屏截图（四张，都在 `~/workspace/tools/pwcheck/`）

| 版本 | 浅色 | 深色 |
|---|---|---|
| v3 | `theme-v3-light.png` | `theme-v3-dark.png` |
| 红笔版 | `theme-redpen-light.png` | `theme-redpen-dark.png` |

## 六、测试结果（全部通过）

详见 `files/test-report-theme.md`。要点：

- `run_test_pass.py`：**36/36**（新增 T-33 切换后 data-theme 与 localStorage 一致、T-34 深色底色不是纯黑），T-10 / T-24g golden 都是 2.22e-16。
- `test_v3.py`：**67/67**（新增 TH-1~TH-8：防闪烁脚本、切换器三档、显式深色对比度、淡彩标签、H1 34px）。
- `test_redpen.py`：**44/44**（新增 TH-1~TH-8：下拉切换器、深色 `#191919`、时间线、大标题、H1 44px）。
- `test_v3_vm.js`：两版都通过（新增 THEME 段：默认 system、切深/浅写 localStorage、非法值回落）。
- `test_v1_core.py`、`test_kc_tagger.js`（高中库抽查 15/16）、`test_llm_tagger.js`、`test_cuojuan.js`（25 个断言）、`test_word_export.js`（36 个断言）、`test_theme.js`（7/7）全部通过。
- 无头浏览器：`check-theme.js`（新写）两版 × 浅/深：点切换就变、刷新后保持、跟随系统在深色系统下生效、手机 390 浅/深都不溢出、**零页面错误**；`check-v3.js` / `check-redpen.js` 全流程数字不变（KPI 30/10/299/58.5%、分层 24.4/65.9/9.8、图例 30/81/12、提纲 7 条）；`check-cuojuan.js` 两版 × 浅/深四遍都通过、无页面错误。
- Impeccable `detect`：v3 剩 2 条，红笔版剩 4 条——都是已知误报，见下节。

## 七、已知问题

1. **Impeccable detect 的误报（不用修）：**v3 的 2 条和红笔版的 2 条是打印块误报（打印强制 `#000` 字，detector 把它和深色底 `#221e17 / #202020` 配对了，实际打印是白底）；红笔版另外 2 条是跨主题误配对（深色字色 `#a3a3a3 / #f5f5f5` 配浅色底 `#ffffff`，这两个颜色在同一主题里从不同时出现；主题内对比度静态测试已逐项验过 ≥ 4.5:1）。detector 不懂按主题分开算，记为已知误报。
2. **修掉的一条真问题：**detect 曾报 v3 切换器“子元素贴边”，已把容器内边距改成 4px、按钮改成 28px 高 pill，复跑后消失。
3. **方向 B 留着备用：**克制仪表盘方向只在 `files/theme-research-20261114.md` 里，没删，没落地。
4. **没动的东西：**功能、诊断引擎（NSKILL=123）、演示数据、演示数字、LLM 标注、KC 库、典型错卷输入都没动；没写任何 API Key 进文件或测试。

## 八、下一步（等用户决定）

1. 看四张首屏截图，确认 A/C 方向可以推。
2. 确认后推 GitHub：main（v3）和 redpen 分支各推一次，线上验证。
3. 再往后：M2 真实考试验证仍是卡住的事；登录 + 管理员后台的方案已另出，用户定了本地优先 + 可选云端账号。
