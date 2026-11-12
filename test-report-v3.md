# v3 测试报告（2026-11-12）

构建：`hidden_files/build_v1.py` → `files/v1-product.html`（728,301 字符）。
视觉世界：分册手册（挑战牌 `rw-manual-acetate-tab-board`，seed `df5926c9d687c630`）。
样式源：`hidden_files/v3-style.css`，已换入 build 脚本的 `<style>` 块。

## 通过的套件

| 套件 | 结果 |
|---|---|
| 旧全量 `hidden_files/run_test_pass.py`（T-01..T-20 + S4） | **21/21 通过** |
| T-10 golden：JS 诊断 vs Python 独立实现 | **max diff 2.22e-16，通过** |
| v3 静态 `hidden_files/test_v3.py` | **45/45 通过** |
| v3 vm `hidden_files/test_v3_vm.js` | **全部通过** |
| Impeccable detector（`detect --json`） | 2 条，均为打印块误报（见下） |

## v3 静态 45 项覆盖

- token：≥40 个、定义外无硬编码色值、每个 token 至少被引用一次。
- 深色模式：重定义全部颜色 token；浅/深两模式全部文字色对比度 ≥4.5（含五步分册色与其文字色、--step-fg/--step）；accent 图形对比度 ≥3.0。
- 字阶/聚焦：h1/h2/h3 与行高走 token；聚焦环跟随当前步骤色；reduced-motion 关闭过渡。
- 响应式：960/1280/640 断点；统计/诊断/拆题区横向滚动；640px 下步骤条可收窄。
- 组件：ghost/danger/loading 按钮、拖拽区、数字右对齐/行悬停/紧凑密度、toast。
- 数据呈现：比例分层条（三段 strong/mid/weak）+ 迷你条；KPI。
- 动效与打印：90ms 两帧步进；@page A4；打印隐藏步骤条/按钮/**操作记录**；卡片表格不跨页断开。
- 无障碍：lang、header/main 地标、aria-label、进度条 role、按钮均有可见文字。
- v3 世界：五步分册色互不相同；body[data-step] 1..5 切换；左侧标签栏；旧蓝 #2563eb 已移除；::selection 跟随步骤色；标题与页头为 v3；分层条按比例渲染；提纲标题与按钮分行（oi-head）。

## 修掉的 5 个 v2 遗留问题（均有测试覆盖）

1. 手机上提纲标题在按钮旁断句：提纲项头部改为 `.oi-head` flex 布局，标题占满一行、按钮另起一组；窄屏按钮自动换到标题下方。
2. 打印带出操作记录：操作记录卡片加 `id="auditCard"`，打印样式隐藏它（E2-4 覆盖）。
3. 页头仍写 v1：`<title>` 与 `<h1>` 改为 v3（V3-6 覆盖）。
4. 加载模拟数据后进度超前：demo 处理器不再调用 `show('stage2')`/`setStep(2)`，只填数并启用确认按钮；进度停在第 1 步 20%，直到老师点确认（vm 流程段覆盖）。
5. 分层三格等宽：改为比例分段条（较好 24.4% / 中等 65.9% / 偏弱 9.8%）+ 图例（D1-1/D1-2 与 vm D1 覆盖）。

## 构建中修掉的两个测试桩问题

- `run_test_pass.py` 的 DOM 桩 `document.body` 缺 `dataset`，v3 的 `setStep` 会写 `body.dataset.step`：桩里补了 `dataset: {}`（与 v2 时补 `firstElementChild` 同类处理）。
- T-01/T-02 曾因 `/tmp/ocr-probe/xlsx.full.min.js` 被重部署清掉而失败：改为优先用仓库内持久副本 `hidden_files/vendor/xlsx.full.min.js`，/tmp 只作后备。

## Detector 结果

初扫 5 条，3 条真实（步骤标签小字在分册色上用 85% 透明度，对比度掉到 4.1–4.4）：去掉 `.step b` 的透明度，改用全色文字，已修复。
复扫剩 2 条，**同一根因，都是误报**：打印块 `body { background: #fff; color: #000; }` 里的 `#000` 被配到深色底 `#221e17` 上。打印时底色强制是 `#fff`，黑字白底对比度 21:1，真实渲染里不存在这组搭配。与 v2 时打印块用设备字面量被排除在硬编码色值检查之外同理。

## 已知局限（与 v1/v2 相同）

- T-06 的 OCR 识别准确率依赖真实试卷图，本报告只验了按题号拆题的规则部分。
- 自动标注是关键词匹配，题干换说法会漏标；置信度是相对分，不是真实概率，分界线待真实试卷实测后再定。
- Word 导出是 HTML 格式的 .doc，不是真 .docx，严格只认 OOXML 的场景可能被拒。
- 除 OCR 需要联网下载模型外，上传、诊断、导出都能离线用。
