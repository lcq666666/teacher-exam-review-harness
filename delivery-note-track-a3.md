# Track A-3 统计 / 诊断 / 提纲交付说明（2026-11-14）

结论先说：A-3 做完了，本地已建好，没推 GitHub。统计页写清口径、诊断页先给一句话结论加分组建议、提纲能拖能删能恢复有版本号、Word 导出改成真 .docx 带预览和校验、导出同步带错卷信息；LLM 标注默认切到凌云 API，真接口实测 15/16。演示数字一个没动，全量测试 59/59，无头检查 44/44，零页面错误。

## 每项现在是什么样

- **A11 统计口径写在页面上**：统计页每项指标旁一句话口径——整体正确率 = 得满分的作答 ÷ 全部作答；平均得分率 = 每道题得分 ÷ 满分的均值；空白格不算 0 分。每道题点“看分布”看全班得分分布（满分 / 部分得分 / 0 分人数 + 平均得分率 + 三段条），分布数和统计表对得上（第 9 题：满分 8 人、部分 0 人、0 分 22 人，平均得分率 27%）。按钮文字跟着状态变，点开是“收起”，再点变回“看分布”。
- **A12 诊断结论先给一句话**：诊断页顶部一句规则生成的话——“这次最弱的是 92 号知识点：……没得满分，一共 N 人次，掌握度 0.42。建议先讲这个知识点的概念，再讲这几道错题。”表格在下面当证据。
- **A13 分组建议落地**：诊断页按最弱 5 个知识点上的平均掌握度分三组——基础巩固组 / 稳固提高组 / 掌握较好组，每组一句话讲评建议（巩固组：先补概念，再跟全班讲评，课后练 3 道同类基础题），组内按掌握度从低到高排。分组边界默认 0.5/0.7，老师能改，改完重算并记在浏览器本地（`examReviewGroupCuts`）；非法边界拒绝并提示。注意：演示数据在默认边界下基础巩固组是 0 人——全班在最弱 5 个知识点上的平均掌握度都在 0.5 以上，最弱的学生也够不上巩固组线；组照样显示，边界能调，不是 bug。
- **A14 错因五类可配置——受阻，没做**：依赖 A-5 的 A20（KC 库按学科独立），本轮不做。错因仍是固定五类（概念不清、审题错误、计算失误、方法不当、书写规范）。钉住的一半：挂不上知识点的错卷不丢，标“未对应知识点”进确认清单（T-54）。
- **A15 提纲顺序可调**：按住拖动（⋮⋮ 手柄）或点上移/下移调顺序，改完立即保存、版本号 +1；删掉的进“已删”列表，点“恢复”放回去；旧版留着能查（只读，最多 10 版）；没改动时（移到头、原地拖拽）版本号不变。提纲头上写“提纲第 N 版”。
- **A16 导出真 .docx**：Word 导出改成真 OOXML .docx（纯 JS 手写 ZIP，stored 压缩、CRC32 表实现，不引第三方包；A4 版面 11906×16838 twips，宋体样式，标题/两级标题）。点导出先出预览 + 校验（ZIP 头、6 个文件齐全、XML 都能解析、每个知识点和错卷行都在正文里），校验通过才能点“确认下载”，不通过按钮点不动。`.doc` 生成函数还留在模块里（测试覆盖），页面上不再提供 `.doc` 下载。
- **A17 导出带错卷信息**：真 .docx 和预览里，有错卷的题下面多一行“典型错法（第 N 题）：…；错因：…”。没记错卷时提纲一个字不变——去掉错卷行后，两份 docx 正文逐字一致（T-57）。
- **LLM 默认切凌云 API**（A-3 附带）：默认接口 `https://api.lingyunapi.xyz/v1`，默认模型 `deepseek-v4.1-flash`；设置区（接口/模型/Key 都能改）不变，已存的老师设置不受影响；Key 仍是 BYOK，不写进产品和任何文件；关键词兜底和 llm/keyword 来源标签不变。设计文档里 5 处过时的 OpenRouter 默认已同步改成凌云。
- **LLM 真接口实测**（A6 遗留，A-3 补上）：2024 新高考 I 卷 16 道题，deepseek-v4.1-flash 走凌云 API，产品自己的提示词与解析，90 个候选知识点（高中库 85 + 演示库 5），temperature 0.2，两批（10 + 6），一次成功。结果 **15/16（93.8%），低置信度 1 道（6.2%）**。关键词同一批题也是 15/16，但低置信度 7 道（含 1 道没标上）；LLM 错的那道是 Q11 新定义曲线，模型给了 1063、置信度只有 0.35，会进老师确认清单，不会悄悄通过。两批用量共 6882 tokens（提示 2418 + 生成 4464，其中推理 3885）。完整记录见 `hidden_files/llm-tagging-2024-gaokao.json`，原始返回见同目录两个 `-raw.txt`，复跑脚本见 `hidden_files/measure_llm_tagging.py`（只走连接器 helper，不碰 Key）。

## 无头检查修掉的两个真问题

这次无头检查抓到两个产品真 bug，都修了（Node 单测没抓到，因为沙箱的 DOM 桩不还原真实浏览器的行为）：

1. **分布按钮点开后文字不变**：`toggleQDist` 只开关分布行，不改按钮文字，点开后按钮还写“看分布”，老师不知道点一下是开还是关。现在点开变“收起”，再点变回“看分布”（按钮加了 `data-qdist` 属性定位）。
2. **分组非法边界的提示被重画擦掉**：`applyGroupCuts` 拒绝时先写提示、再调 `renderGroups()` 重画，重画把提示所在的 `<span>` 换成新的空元素，提示一次都没出现过。现在先重画（输入框恢复成已保存的边界），再写提示。

## 改了哪些文件

- `hidden_files/build_v1.py`：A11–A17 全部产品逻辑 + 上面两个修补；`__core` 新加 9 个只读口子（`scoreDistribution`、`diagnosisSummary`、`groupStudents`、`getGroupCuts`、`moveOutlineItem`、`getOutlineVersion`、`getOutlineHistory`、`getDeletedItems`、`getOutline`）。
- `hidden_files/word_export.js`：真 .docx（`buildDocx` + `validateDocx`，A4、宋体、标题样式）+ 预览共用的 `buildExportItems`/`docxPreviewHtml` 在 build 里。
- `hidden_files/llm_tagger.js`：默认接口和模型切凌云。
- `hidden_files/v3-style.css`、`hidden_files/redpen-style.css`：只加布局（结论间距、分布行、拖拽手柄、已删项、预览面板边线），不改颜色。
- `hidden_files/run_test_pass.py`：新增 T-51–T-58 + T-56b（Python zipfile 复核）；报告生成改到 `files/test-report-track-a3.md`；LLM 局限行改成实测数字。
- `hidden_files/test_v3_vm.js`：新增 A3 段（A11–A17 全流程）；两个 vm 沙箱补了 `TextEncoder`/`TextDecoder`（浏览器本来就有，沙箱缺了它 `buildDocx` 跑不起来）。
- `hidden_files/measure_llm_tagging.py`（新增）：LLM 实测复跑脚本；`hidden_files/llm-tagging-2024-gaokao.json`（新增）：实测结果；同目录两个 `-raw.txt`：模型原始返回。
- `files/teacher-exam-review-harness-requirements.md`：升到 v1.9，A-3 变更写在最前面，v1.8 内容不动。
- `files/teacher-exam-review-harness-design-v2.md`：5 处 OpenRouter 默认同步改成凌云（§3.2 表格、架构图、选型表、3.3 流程、外部依赖表）。
- `~/workspace/tools/pwcheck/check-track-a3.js`（新增）：A-3 无头检查；截图 6 张（`a3-{v3,redpen}-light-{stats,outline}.png`、`a3-{v3,redpen}-mobile-stage5.png`）。
- 构建产物：`files/v1-product.html`（1,162,560 字节）、`files/redpen-product.html`（1,162,030 字节）。没动 `deploy/`、`deploy-redpen/`，没推 GitHub。

## 测试结果（都在最终构建上跑过）

- `run_test_pass.py`：**59/59**（新增 T-51–T-58 全过；T-56b 的 Python zipfile + ElementTree 复核通过：6 个文件齐全，XML 都能解析；JS 与 Python 诊断最大差 2.22e-16）。
- `test_v3.py` 73/73、`test_redpen.py` 50/50、`test_auth.js` 59 通过 0 失败、`test_cuojuan.js` 31、`test_theme.js` 7/7、`test_kc_tagger.js` 通过（10/10、15/16、8/10）、`test_llm_tagger.js` 通过、`test_word_export.js` **71**（docx 新增 35 条）、`test_v3_vm.js` 全过（含新 A3 段）、`test_v1_core.py` 通过。
- 无头检查：`check-track-a3.js` **44/44**（两版 × 浅色/深色 + 手机 390，零页面错误）；`check-v3.js`、`check-redpen.js`、`check-next-station.js` 零失败；`check-track-a1.js` 48/48；`check-track-a2.js` 36/36。
- 演示数字没动：KPI 30/10/299/58.5%，分层 30/81/12，提纲 7 条。

## 没验到的事

- `.docx` 文件的实际下载没点过：无头壳不处理下载，只验了按钮可用、预览和校验；下载用的是和 Markdown 导出同一套 Blob 机制。真机上点一次“确认下载”就能确认。
- 拖拽本身没在真浏览器里拖过：拖拽调顺序的逻辑（`reorderOutline`）在 Node 单测和无头检查里都验过，页面上的拖拽手柄和 `draggable` 属性在无头检查里验过；真机上拖一次就能确认。
- 低置信度清单里的“点备选换知识点”只在合成数据上单测过（关键词低置信度题通常没有备选）；无头检查走的是“就用这个”和拆题表填编号，两条都通。
- LLM 实测是单次跑的结果（temperature 0.2，同一批题重跑可能有小波动）；Q8 置信度 0.55 刚过线，重跑可能进确认清单——进清单是设计好的行为，不是问题。
- M2（真老师真考试）还是没做，照样卡着后面的事。

## 接下来

A-4（错卷：10 条以上新例子验泛化 + 固定位置）、A-5（KC 库：学科 + 版本可插拔，A20 解锁 A14）、A-6（登录/后台：管理员密码找回 + 登录边界测试）、A-7（性能与质量门：5 分钟计时实测、断点续做、全量回归）都还没动。A-3 本地已建好，等用户确认后推。
