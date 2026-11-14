# Track A-5 KC 库按学科独立 + 错因可配置交付说明（2026-11-14）

结论先说：A-5 做完了，本地已建好，没推 GitHub。KC 库从一个写死的高中数学库变成注册表（学科+版本两维），标注、诊断、提纲、导出都走注册表；库的版本在第 2 步、诊断页脚、提纲页脚、后台都看得见；A-3 被 A20 卡住的 A14 也解封了——五类错因的名字老师可以改，改名只换显示的名字，归类结果一个字不变，调参集还是 14/14，新泛化集还是 17/20。演示数字一个没动，全量测试 66/66，无头检查 40/40，零页面错误。

## 每项现在是什么样

- **A20 KC 库按学科独立**：`hidden_files/kc_tagger.js` 里加了一个注册表（`KC_BANK_REGISTRY` / `KC_BANK_ORDER` / `ACTIVE_BANK_ID`），五个函数：`registerBank`（注册一学科）、`getBank`（按 id 取库）、`listBanks`（只给元数据：学科、版本、来源、点数）、`activeBank`（当前学科的库）、`setActiveBank`（切换当前学科）、`bankInfo`（一学科的展示信息）。高中数学库（85 点，1001–1085，人教版 2019 版，2026-11-14 版）和演示库（5 点）都注册在表里，产品默认的活动库是高中数学。标注（关键词标注、LLM 候选库）、诊断（知识点名）、提纲、导出、后台 KC 信息原来直接读 `HS_BANK` / `DEMO_BANK` 的地方，现在都走注册表（`activeBank()` / `getBank('demo')`）。加一个新学科是数据-only：一个学科对象（学科名、版本、来源、知识点 + 关键词）加一行 `registerBank` 调用，不改标注、诊断、提纲、导出的任何流程代码。演示库走同一结构，就是“加学科”的现成证明。
  - 没编第二个学科：没有真实数据，编 85 个假知识点是造假。测试里用一个 2 点的合成小库（1998/1999，关键词“合成甲/合成乙”）演示注册、切换、切回，只存在测试进程里，不进产品。
  - 编号段的口子：标注模块的编号规则只认 1–123（演示/旧编号）和 1001–1999（高中库编号段），高中库只用到 1085，段里还剩 914 个空位。真加一个学科时，编号段要在那时定——现在这段还够用，但两个学科不能撞号，这个决定留给加学科的那一天。
  - 产品里没有学科切换界面：现在只有一个首发学科（高中数学），页面上不给老师一个切来切去的开关；`setActiveBank` 是给以后加学科留的口子，只在测试里验证。
- **A21 库版本看得见**：一句话“知识点库：高中数学 · 2026-11-14（人教版 2019 版），85 个知识点”，出现在三处：第 2 步标注区（“本次标注用知识点库：……”）、诊断页页脚、提纲页脚，都是 `bankLineText()` 一个函数拼的；后台 KC 信息从 `listBanks()` 取，文案和以前一个字没变（“高中数学 KC 库：85 个知识点（1001–1085），2026-11-14 版。演示词库：5 个知识点。”）。
- **A14 错因五类可配置**：第 2 步未解析清单下面多了“错因分类设置”区，五个输入框（第 1 类到第 5 类）+ 保存 / 恢复默认五类两个按钮。五类是五个槽位，名字只是标签：`classifyCause(text, causes)` 计分全程按默认名算，最后一步才换成配置里同一槽位的名字，所以改名不改变任何一句话的归类结果，只换显示的名字。改完：新记的错卷按新名字归类；原来记的错卷按槽位改名（旧有效名或默认名在第 i 个槽位，就换成新名字里第 i 个），不丢；诊断、提纲、导出里的错因跟着变；改动记进审计。配置存在浏览器本地 `examReviewCauses` 里（JSON 数组，5 个名字）。校验：5 个非空、不重名才保存，否则拒绝并提示，旧名字不动；名字没变时不写配置（测试抓到过一次反例：原来没变也写，已修）。恢复默认五类把配置清掉，已记的错卷按槽位改回默认名。
  - 默认五类没变：`CAUSES` 仍是概念不清、审题错误、计算失误、方法不当、书写规范；`resolveCauses` 对不合法的配置（少于/多于 5 个、空名、重名、不是数组）一律回默认五类。

## 改了哪些文件

- `hidden_files/kc_tagger.js`：注册表（`KC_BANK_REGISTRY` / `KC_BANK_ORDER` / `ACTIVE_BANK_ID`，`registerBank` / `getBank` / `listBanks` / `activeBank` / `setActiveBank` / `bankInfo`）；高中数学库和演示库在模块末尾注册；`__kcTagger` 导出新函数。`HS_BANK` / `HS_BANK_VERSION` / `DEMO_BANK` 还在（静态检查要它们），流程代码不再直读。
- `hidden_files/cuojuan.js`：`resolveCauses`（配置校验：5 个非空、不重名才通过，否则回默认）、`causeSlot`（默认名按槽位取 0–4）、`classifyCause(text, causes)` / `parseCuojuanNotes(text, causes)` 接受可选的有效名字数组，计分按默认名、返回按槽位换名；`__cuojuan` 导出两个新函数。默认五类、`CAUSES`、关键词规则、平局顺序一个字没动。
- `hidden_files/build_v1.py`：注册表取用点——`bankPoints` / `activeBankPoints` / `activeBankMeta` 三个辅助函数，`kcNameById` / `skillName` / `buildLLMBank` / `runKeywordTag` / LLM 兜底关键词库 / 拆题表 datalist / 后台 KC 信息都改走注册表；A21——`bankLineText()` + `renderBankInfoLine()`（第 2 步 `#bankInfoLine`）、诊断页与提纲页页脚；A14——模板里错因下拉改成空 select（初始化时 `refreshCauseDropdowns()` 按当前名字拼）、新增“错因分类设置”区（`#causeSettings` + `#causeSaveBtn` / `#causeResetBtn` / `#causeMsg`），`CAUSES_DEFAULT` / `loadCauses` / `effectiveCauses` / `migrateCuojuanCauses` / `refreshCauseDropdowns` / `renderCauseSettings` / `saveCauseSettings` / `resetCauseSettings`，解析按钮把有效名字传进 `parseCuojuanNotes`，初始化多调三行（`refreshCauseDropdowns(); renderCauseSettings(); renderBankInfoLine();`）；`__core` 新加只读口子 `getCauses` / `getBankInfo` / `getBankLine`。修了一个真问题：名字没变时 `saveCauseSettings` 原来嘴上说“不用保存”实际还是写了配置，现在没变不写。
- `hidden_files/test_kc_tagger.js`：第 7 节注册表（内置两库、元数据不带 points、默认活动库、合成小库注册/切换/切回、演示库走注册表、非法注册与未知切换拒绝）。
- `hidden_files/test_cuojuan.js`：第 12 节 A14（`resolveCauses` 8 种非法配置回默认、`causeSlot` 槽位、改名后同一句话归到同一槽位、调参集 14 条用新名字重验 14/14、新泛化集 20 条用新名字重验 17/20 严格 16/20）。共 74 个断言（原来 52 个）。
- `hidden_files/run_test_pass.py`：新增 T-62–T-65；T-54 的标签和断言文案更新（A14 已解封，断言本身不变：默认五类 + 挂不上的错卷不丢）；沙箱桩补两个真实浏览器本来就有的方法（元素 `querySelectorAll`、`localStorage.removeItem`），否则 T-63 的 `renderAdminPanel` 和 T-64 的恢复默认测不了。
- `hidden_files/test_v3.py` / `hidden_files/test_redpen.py`：各加 A5-1（注册表在页面里）/A5-2（错因分类可配置）静态断言。
- `hidden_files/test_v3_vm.js`：A14 段的过时注释改了（“受 A-5 的 A20 阻塞”→“默认五类不变”）；新增 A5 段——注册表与库信息走 `__core`，第 2 步、诊断、提纲的库信息走真实点击，改名走真实保存按钮（诊断/提纲用新名字、旧名字消失），空名拒绝走真实按钮，恢复默认走真实按钮。
- `~/workspace/tools/pwcheck/check-track-a5.js`（新）：A-5 专项无头检查，两版 × 浅色/深色 + 手机 390，40/40，4 张截图（`a5-{v3,redpen}-light-cause.png`、`a5-{v3,redpen}-mobile-stage5.png`）。
- 产品重建：`files/v1-product.html` 1,182,350 字节；`files/redpen-product.html` 1,181,820 字节。红笔版按 `files/delivery-note-theme.md` 里的完整环境变量重建，一次通过。

## 以后加一个学科怎么做（数据-only）

1. 准备数据：学科名、版本、来源、知识点清单（编号、名称、每个点 3–8 个标注关键词），编号用 1001–1999 段里还没被占的号（高中数学占到 1085）。
2. 在 `kc_tagger.js` 里加一行 `registerBank('新学科 id', { subject, version, source, points })`，和高中数学、演示两行并排。
3. 不用改流程代码：标注、诊断、提纲、导出、后台都从注册表取库。页面上的学科切换（如果要给老师一个开关）走 `setActiveBank`，那是以后加界面的事，不在本轮。
4. 测：新学科的标注抽查（像高中库的 2024 高考卷 15/16 那样，用该学科的真题量一次），再跑全量套件。

## 测试结果

- 主套件 `run_test_pass.py`：**66/66 通过**（62 + 新增 T-62–T-65；T-10/T-24g 金标 JS 与 Python 诊断最大差 2.22e-16；T-56b Python zipfile 复核通过）。
- `test_cuojuan.js`：**74 个断言通过**（原来 52 个；新增第 12 节 22 个断言：改名重验调参集 14/14、新泛化集 17/20 严格 16/20，与默认五类一致）。
- `test_kc_tagger.js`：通过（演示库 10/10、2024 新高考 I 卷 15/16、模拟高中卷 8/10；新增第 7 节注册表：合成库注册/切换/切回、非法注册与未知切换拒绝）。
- `test_v3.py`：**77/77**；`test_redpen.py`：**54/54**；`test_v3_vm.js`：全过（两版都跑过，含新 A5 段）；`test_auth.js`：59 通过 0 失败；`test_word_export.js`：71 个断言通过；`test_kc_tagger.js`：通过；`test_llm_tagger.js`：通过；`test_theme.js`：7/7；`test_v1_core.py`：全过。
- 无头浏览器（本地文件 file://，零页面错误）：`check-track-a5.js` **40/40**；`check-track-a4.js` 36/36；`check-track-a3.js` 44/44；`check-track-a2.js` 36/36；`check-track-a1.js` 48/48；`check-v3.js`、`check-redpen.js`、`check-next-station.js` 退出 0。
- 演示数字没动：KPI 30/10/299/58.5%、分层 30/81/12、提纲 7 条。
- 测试抓到的真问题（已修）：T-65 发现名字没变时保存函数还是写了配置——现在没变不写（见上面）。

## 没验的东西

- 没有第二个真实学科的数据，所以“加学科数据-only”只在测试里用合成小库演示过，没拿真实学科走过全程；真加学科时要按上面四步再验一次标注准确率。
- 产品里没有学科切换界面，只有一个首发学科；`setActiveBank` 只在测试里验证过。
- .docx 实际下载没点过（无头壳不接下载；按钮可用、预览、校验已验）——A-3 同款局限。
- 拖拽本身没在真浏览器里拖过（逻辑在单测 + 无头检查里验过）——A-3 同款局限；低置信度清单里“点备选换知识点”只在合成数据上单测过——A-2 同款局限。
- M2 真实老师 + 真实考试还没做；A-3、A-4、A-5 都还在本地，没推 GitHub，等用户一句话（推 A-3+A-4+A-5，还是一起攒到 A-6 再推）。
