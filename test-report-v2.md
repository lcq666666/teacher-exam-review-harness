# v2 前端优化测试报告（2026-11-12）

三套测试全部通过，诊断数学与旧用例不受影响。

## 1. 旧测试（run_test_pass.py）

Node 可跑部分：**21/21 通过**；T-10 黄金对照：JS 诊断与 Python 独立实现 max diff 2.22e-16。

## 2. v2 静态检查（test_v2.py）：34/34

- A1 token：55 个 token，定义外无硬编码色值，每个 token 都被引用。
- A2 深色模式：dark 重定义全部颜色 token；浅色/深色两套文字色对比度全部 ≥ 4.5（浅色最低 4.52，深色最低 7.14）；accent 图形对比度 ≥ 3.0。
- A3 字阶/聚焦：h1/h2/h3 与行高走 token；:focus-visible 聚焦环；reduced-motion 关闭过渡与动画。
- B1 响应式：960px 与 1280px 断点；统计/诊断/拆题区横向滚动；640px 下步骤条收窄。
- C1–C4 组件：ghost/danger/loading 按钮变体；拖拽区样式；数字右对齐/行悬停/紧凑密度；toast 样式与 role=status。
- D1–D2 呈现：分层/迷你条/KPI 样式。
- E1–E2 动效/打印：按压/悬停/淡入；reduced-motion；print 块含 @page A4、隐藏步骤条与按钮、3 处 break-inside: avoid。
- F2 无障碍：lang="zh-CN"、header/main 地标、文件输入与拖拽区 aria-label、进度条 role、17 个按钮都有可见文字。

## 3. v2 vm 渲染单测（test_v2_vm.js）：全部通过

B2 进度条（setStep(3) → 60%，aria-valuenow 同步）、B3 OCR 加载态与空状态、C1 按钮 loading 态与 ghost/danger 变体、C2 拖拽高亮与拖放载入、C3 语义化表格与密度切换、C4 toast 显示与 4 秒消失、D1 分层计数（总和 123）与迷你条、D2 KPI 四项数值（30/10/299/58.5%）、D3 提纲徽章与错题 chips、F2 拖拽区键盘操作（回车打开文件选择），以及模拟数据全流程。

## 执行中调整

- C3 吸顶表头没做：表格最长 15 行，没有纵向滚动区间，吸顶无收益；行悬停 + 密度切换已覆盖可读性。
- 打印样式用 #fff/#000 设备字面量，A1 检查排除 print 块。
- 旧测试桩补了 firstElementChild 与 setAttribute（B2 进度条需要），旧用例本身未改。

## 待浏览器验收

桌面 + 手机全流程、深色模式、打印预览（见 F3 浏览器验收）。
