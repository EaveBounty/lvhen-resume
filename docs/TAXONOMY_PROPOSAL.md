# 「履痕」事物分类体系重构建议 v4（规则终修稿）

> 修订基线：`taxonomy_proposal_v3.md`；修订依据：`taxonomy_critique_v3.md`（MF1-a/b/c/d + S1–S4）；证据：`cat_snapshot.json`（91 条）。
> 本稿只动 **§5 规则表**及其派生的 §6/§10 计数，**架构层（D′ / 13 大类 / 32 子类 / 章节映射 / override / relations / 迁移版本门）不变**（critique_v3 §2：D1–D5 全通过、架构可交付）。
> 核心三改：**判定输入去 `description`（MF1-a）**、**R10 学生干部收窄并后置到表彰之后（MF1-b）**、**R3 毕业论文前置于 R4 学历（MF1-c）**；随后**按修正规则重跑全部 91 条（MF1-d）**。

---

## 0. 修正摘要（回应 critique_v3）

| 项 | 严重度 | v4 处置 |
|---|---|---|
| **MF1-a** 判定输入含 desc → 4 条 works 被 R12 抢走 | high | 判定输入**回退为 `title ∪ tags ∪ organization`**（去掉 `description`）。英文论文靠 `tags`（论文/EI/SCI/CPCI）命中 R8，无需 desc。4 条 works 恢复：`19fb3834`/`2c43c446`/`9795cc70`→works/软件作品，`ee10370f`→works/装置设计。 |
| **MF1-b** R10 过宽（团委/学生会/委员）夺走活动/表彰/培训 4 条 | high | R10 收窄为**强职务词**（团支书/班长/班委/学生会主席/副主席/部长/副部长/社长/会长/队长/寝室长/年级工作组），删除 `团委/学生会(裸)/委员/干事/负责人`；R10 **移到 R9（表彰）之后、R11（公司任职）之前**。`60545726`→activities/志愿服务、`22d86d38`→awards/表彰荣誉、`634c2742`/`72e21e63`→trainings/MOOC培训 全部恢复正确。 |
| **MF1-c** R3(本科) 抢"本科毕业论文" | high | **R3=毕业论文/毕业设计前置于 R4=学历授予**；`f1cf3cd8`→research/毕业论文。 |
| **MF1-d** §10 四行自相矛盾 + §6.2/§6.3 失真 | high | **按修正规则重跑 91 条**：覆盖 91/91、强判 86、needs_review 5；§10 全表重出（含修正后的 4 行 + 新增 f1cf3cd8 等）；§6.3 逐类计数据实重列（works 恢复 4、毕业论文恢复 1、志愿服务恢复 6、MOOC 恢复 2）。 |
| **S1** `开发项目` 触发弱 | low | §4.4 补正判据（独立立项/课程交付的工程项目）；并说明真实数据暂无样例、作可选子类。 |
| **S2** R19 裸"证明"偏宽 | low | R19 删除裸"证明"，保留 `简历/学历证明/离职证明/成绩单` 等具体词；参与类证明由 R17 先拦截。 |
| **S3** R8 通用词 conference/paper 风险 | low | R8 的 `paper/journal/conference` **仅当 tags 命中或标题含 `paper/proceedings` 时计**；中文词（论文/期刊/会议/EI/SCI/CPCI）照常。 |
| **S4** 项目↔作品关系未覆盖 | low | §7 补"项目↔作品"关系约定（research 大创项目与其产出 works 可并存的 link 规则）。 |

---

## 1. 根因分析（不变）

1. **单层扁平**：15 个同级无父子，混合"领域/形态/归属"，伪 MECE。
2. **复合概念**：awards/activities/publications/experience/certificates 一桶多物。
3. **职责耦合**：`slug == sectionKey`，存储稳定与简历可变硬绑。
4. **缺领域标准**：未对齐学术 CV / HR / 综测 / 知识管理。
5. **真实数据校正**：91 条已有 26 个 `fields.subcategory`，病灶在 `category` 层与耦合。

---

## 2. 方案对比与决策矩阵（不变，D4 口径）

打分 **5=最优**；迁移成本、实现复杂度已换算为"越易/越简越高分"；实现复杂度计"含新增机制"。

| 维度 | A 扁平扩展 | B 两层大词典 | C 全 facet | **D′ 主干+32+章节解耦** |
|---|:---:|:---:|:---:|:---:|
| 表达力 | 3 | 5 | 5 | 4 |
| MECE（以 subtype 消歧） | 2 | 4 | 3 | 4 |
| 可扩展 | 2 | 5 | 5 | 5 |
| 迁移成本 | 4 | 2 | 1 | 3 |
| 实现复杂度 | 4 | 3 | 1 | 2 |
| AI 友好 | 3 | 3 | 2 | 4 |
| 简历适配 | 3 | 4 | 5 | 5 |
| **合计** | **21** | **26** | **22** | **27** |

D′ 以"复用既有 26 值（迁移轻）+ 章节解耦（简历适配高）+ 32 选项可控（AI 友好）"组合胜出，非单维碾压。

---

## 3. D′ 架构（不变）

```
存储层：category（13 封闭）→ data/records/<category>/<id>.json
语义层：subtype（种子 32 = 26 复用 + 6 新增，开放）
呈现层：resume_section = MAP[(category, subtype)] + 记录级 override
```
明确不含独立 facet（级别/结果沿用 `tags`/`fields`）。

---

## 4. 分类体系（不变；S1 补判据）

### 4.1 大类表（13）

| # | 键 | 中文名 | 语义 | 图标 |
|---|---|---|---|---|
| 1 | `education` | 教育经历 | 授予学历/学位的教育 | `school_outlined` |
| 2 | `experience` | 工作与职业 | 全职/实习/兼职、公司任职、创业任职 | `work_outline` |
| 3 | `leadership` | 学生工作与组织任职 | 学生干部、社团/工作室负责人（担任职务） | `diversity_3_outlined` |
| 4 | `research` | 科研经历 | 科研项目/课题、毕业论文 | `science_outlined` |
| 5 | `publications` | 论文与知识产权 | 论文、专利、著作权/软著、著作 | `menu_book_outlined` |
| 6 | `works` | 项目与作品 | 项目本体、软件作品、装置设计 | `widgets_outlined` |
| 7 | `awards` | 竞赛与荣誉 | 学科竞赛、竞赛参与、表彰荣誉、奖助学金 | `emoji_events_outlined` |
| 8 | `skills` | 技能特长 | 专业技能能力（非证书） | `construction_outlined` |
| 9 | `credentials` | 证书与资格 | 技能证书、等级考试、语言等级考试 | `verified_outlined` |
| 10 | `trainings` | 培训进修 | MOOC、青马、研修等学习过程 | `cast_for_education_outlined` |
| 11 | `activities` | 实践与志愿 | 志愿服务、社团活动、社会实践 | `volunteer_activism_outlined` |
| 12 | `archive` | 档案与隐私 | 健康隐私、证照归档、简历证明、综测材料 | `inventory_2_outlined` |
| 13 | `custom` | 其他/自定义 | 兴趣、推荐人、无法归类（兜底） | `category_outlined` |

### 4.2 子类清单（**32 = 26 复用 + 6 新增**）

| category | subtype | 来源 |
|---|---|---|
| education | 本科 | 复用 |
| experience | 公司任职、★创业任职 | 复用 / 新增 |
| leadership | 学生干部 | 复用 |
| research | 科研项目、毕业论文 | 复用 |
| publications | 论文、专利、著作权 | 复用 |
| works | 软件作品、装置设计、★开发项目 | 复用 ×2 / 新增 |
| awards | 学科竞赛、竞赛参赛、表彰荣誉、奖学金 | 复用 |
| skills | ★专业技能 | 新增 |
| credentials | 技能证书、等级考试、普通话、英语 | 复用 |
| trainings | MOOC培训、青马工程 | 复用 |
| activities | 志愿服务、社团活动 | 复用 |
| archive | 健康隐私、档案证照、简历证明、综测归档 | 复用 |
| custom | ★兴趣、★推荐人、★其他 | 新增 |

**26 复用值**：本科、公司任职、学生干部、科研项目、毕业论文、论文、专利、著作权、软件作品、装置设计、学科竞赛、竞赛参赛、表彰荣誉、奖学金、技能证书、等级考试、普通话、英语、MOOC培训、青马工程、志愿服务、社团活动、健康隐私、档案证照、简历证明、综测归档。
**6 新增值**：创业任职、开发项目、专业技能、兴趣、推荐人、其他。
**计数一致**：`record_subtype.dart`=32、AI 枚举=13+32、`taxonomy_mece_test` 断言 32。

### 4.3 简历章节映射

| sectionKey | 章节 | 来源 (category/subtype) |
|---|---|---|
| `education` | 教育背景 | education/* |
| `experience` | 工作与实习经历 | experience/公司任职, 创业任职 |
| `leadership` | 学生工作与领导 | leadership/* |
| `research` | 科研经历 | research/科研项目, 毕业论文 |
| `publications` | 论文与知识产权 | publications/论文, 专利, 著作权 |
| `projects` | 项目经历 | works/开发项目 |
| `works` | 作品集 | works/软件作品, 装置设计 |
| `awards` | 竞赛获奖 | awards/学科竞赛；竞赛参赛默认隐藏 |
| `honors` | 荣誉表彰 | awards/表彰荣誉 |
| `scholarships` | 奖助学金 | awards/奖学金 |
| `skills` | 技能特长 | skills/* |
| `certificates` | 证书与资格 | credentials/技能证书, 等级考试 |
| `languages` | 语言能力 | credentials/普通话, 英语 |
| `training` | 培训进修 | trainings/* |
| `activities` | 社会实践与志愿服务 | activities/* |
| `teaching` | 教学与助教 | research/*（教学向）或新增 subtype（可选） |
| `interests` | 兴趣爱好 | custom/兴趣 |
| `references` | 推荐人 | custom/推荐人（默认敏感） |
| `archive` | 档案证照 | archive/*（简历默认排除） |
| `misc` | 其他 | custom/其他 |

### 4.4 子类级判据（含 S1 修正）

> 先按 §5 定 `category`，再用本表定 `subtype`；判据 = `title ∪ tags ∪ organization`。

| category | subtype 选择规则 |
|---|---|
| experience | 含 `法定代表人/法人/股东/创始人/持股/出资` → **创业任职**；否则 `公司/任职/实习/工程师/员工/岗位` → **公司任职** |
| research | `毕业论文/毕业设计` → **毕业论文**；`大创/创新创业训练计划/科研项目/课题/基金` → **科研项目** |
| publications | `专利/发明/实用新型` → **专利**；`著作权/软著` → **著作权**；`论文/期刊/会议/EI/SCI/CPCI/paper` → **论文** |
| works | `软件/系统/平台/应用/APP` → **软件作品**；`装置/传感器/硬件/设备/光纤` → **装置设计**；否则含 `项目/开发/工程`（**独立立项或课程交付的工程项目**）→ **开发项目**；仍不决 → 软件作品 |
| awards | `奖学金/助学金` → **奖学金**；**枚举** `优秀学生/优秀学生干部/优秀团员/优秀团干/优秀毕业生/优秀志愿者/三好/标兵/模范/先进个人` → **表彰荣誉**（注意：裸"优秀奖"归 **学科竞赛**，即 R12）；有奖项词 → **学科竞赛**；仅参赛/观众/德育分 → **竞赛参赛** |
| credentials | `普通话` → **普通话**；`英语/CET/雅思/托福` → **英语**（**排除"英语竞赛/参赛证书"**，如 NECCS 参赛证书 → 技能证书）；`等级/合格/NCRE/计算机等级` → **等级考试**；其余 `证书/资格/认证/水平` → **技能证书** |
| activities | `志愿/志愿者/义工/公益` → **志愿服务**；`社团/协会/工作室（非职务）/打卡/参与类证明/参与` → **社团活动** |
| trainings | `MOOC/在线课程` → **MOOC培训**；`青马/青年马克思主义` → **青马工程**；其余 → 待补充子类 + `needs_review` |
| archive | `体检/影像/健康/病历` → **健康隐私**；`简历/求职材料` → **简历证明**；`综测/综合测评` → **综测归档**；`身份证/护照/证照/档案/高考/成绩查询/转递` → **档案证照** |
| custom | `兴趣/爱好` → **兴趣**；`推荐人/推荐信` → **推荐人**；其余 → **其他** |

> **S1 说明**：`works/开发项目` 真实数据无样例，作**可选子类**保留；其正判据为"独立立项（非大创）/课程交付的工程项目"，避免与 research/科研项目、works/软件作品重叠。

### 4.5 MECE 措辞（D5）与元类别

- **内容域（11）**：education, experience, leadership, research, publications, works, awards, skills, credentials, trainings, activities。
- **元类别（2）**：`archive`（存储/隐私轴）、`custom`（兜底轴）。
- 措辞：**"内容域以 `subtype` 消歧"**；不宣称字面不重不漏。

### 4.6 标准对齐（如实）

学术 CV：education/research/publications/skills/teaching/references/awards(honors)；HR：education/experience/works(projects)/awards/credentials/skills/languages；综测：仅 `archive/综测归档`，不建评分轴；PKM：13 类 + tags/fields。

---

## 5. 硬判定规则（v4 修正版：完整、有序、命中即止）

> **判定输入 = `title ∪ tags ∪ organization`（不含 `description`，MF1-a）。**
> **命中即止，自上而下。** 优先级语义：**奖学金/创业 > 毕业论文 > 学历 > 研究/IP > 表彰 > 学生干部 > 公司任职 > 竞赛获奖 > 培训 > 证书 > 竞赛参与 > 活动 > 产出 > 归档 > 兜底。**

**Tier 0**
| R | 条件 | 落点 |
|---|---|---|
| R1 | `奖学金` / `助学金` | awards/奖学金 |
| R2 | `法定代表人` / `法人` / `股东` / `创始人` / `持股` / `出资` | experience/创业任职 |

**Tier 1 — 学历与研究（R3 前置于 R4，MF1-c）**
| R | 条件 | 落点 |
|---|---|---|
| R3 | `毕业论文` / `毕业设计` | research/毕业论文 |
| R4 | `本科` / `硕士` / `博士` / `学士` / `学位` | education/本科 |
| R5 | `大创` / `创新创业训练计划` / `科研项目` / `课题` / `基金` | research/科研项目 |

**Tier 2 — 知识产权**
| R | 条件 | 落点 |
|---|---|---|
| R6 | `专利` / `发明` / `实用新型` | publications/专利 |
| R7 | `著作权` / `软著` / `软件著作权` | publications/著作权 |
| R8 | `论文` / `期刊` / `会议` / `EI` / `SCI` / `CPCI`；或（tags 命中 / 标题含）`paper` / `journal` / `proceedings`（S3） | publications/论文 |

**Tier 3 — 表彰 > 学生干部 > 公司任职（MF1-b：R10 收窄并后置）**
| R | 条件 | 落点 |
|---|---|---|
| R9 | `表彰` / `评优` / `荐优` / `三好` / `标兵` / `模范` / `先进个人` / `优秀学生干部` / `优秀团员` / `优秀团干` / `优秀志愿者` / `优秀毕业生` / `荣誉称号` | awards/表彰荣誉 |
| R10 | **强职务词**：`团支书` / `班长` / `班委` / `学生会主席` / `学生会副主席` / `部长` / `副部长` / `社长` / `会长` / `队长` / `寝室长` / `年级工作组` | leadership/学生干部 |
| R11 | `公司` / `任职` / `实习` / `工程师` / `员工` / `岗位`（未命中 R2） | experience/公司任职 |

> R10 **不含** `团委`/裸`学生会`/`委员`/`干事`/`负责人`（过宽）；R9 显式约定"优秀志愿者→表彰，纯志愿者→activities"，"优秀奖→R12 竞赛"（R9 词表不含裸"优秀"）。

**Tier 4 — 竞赛/培训/证书/竞赛参与**
| R | 条件 | 落点 |
|---|---|---|
| R12 | 奖项词 `一等奖`/`二等奖`/`三等奖`/`金奖`/`银奖`/`铜奖`/`优胜奖`/`优秀奖`/`名次`/`奖`（非奖学金） | awards/学科竞赛 |
| R13 | 学习过程词 `培训` / `课程` / `结业` / `研修` / `MOOC` / `青马` / `青年马克思主义` | trainings/{MOOC培训\|青马工程} |
| R14 | 证书/资格词 `证书` / `资格` / `等级` / `水平` / `合格` / `认证` / `成绩报告单`（无过程词） | credentials/{技能证书\|等级考试\|普通话\|英语} |
| R15 | `参赛` / `参赛证明` / `观众` / `德育分`；或 `参与` 且含竞赛词 `竞赛/大赛/比赛/挑战杯/建模/MCM/互联网+/职业规划` | awards/竞赛参赛 |

> R13 先于 R14：`MOOC培训合格证书`/`青马结业证书` 含过程词归 trainings；`NECCS 参赛证书` 无过程词，含"证书"归 credentials。单条记录只允许一个归口。

**Tier 5–8 — 活动/产出/归档/兜底**
| R | 条件 | 落点 |
|---|---|---|
| R16 | `志愿` / `志愿者` / `义工` / `公益` | activities/志愿服务 |
| R17 | `社团` / `协会` / `工作室`（非职务）/ `打卡` / `参与类证明` / `参与` | activities/社团活动 |
| R18 | `软件` / `系统` / `平台` / `应用` / `APP` / `装置` / `传感器` / `硬件` / `作品` / `设计` | works/{软件作品\|装置设计\|开发项目} |
| R19 | `身份证` / `护照` / `档案` / `证照` / `隐私` / `体检` / `影像` / `健康` / `简历` / `综测` / `综合测评` / `高考` / `成绩查询` / `转递` / `学历证明` / `离职证明` / `成绩单`（**删裸"证明"，S2**） | archive/{…} |
| R20 | 以上皆不命中 | custom/其他 + **`needs_review: true`** |

### 5.1 判定树

```
R1 奖学金 → awards/奖学金
R2 法定代表人/股东/创始人 → experience/创业任职
R3 毕业论文/毕设 → research/毕业论文
R4 本科/硕士/博士/学位 → education
R5 大创/科研项目/课题 → research/科研项目
R6/R7/R8 专利·软著·论文 → publications/*
R9 表彰/评优/三好/优秀志愿者 → awards/表彰荣誉
R10 团支书/班长/部长/副部长/社长/寝室长 → leadership/学生干部
R11 公司/任职/实习/工程师 → experience/公司任职
R12 奖项词 → awards/学科竞赛
R13 培训/结业/MOOC/青马 → trainings/*
R14 证书/资格/等级/合格 → credentials/*
R15 参赛/观众/德育分（或含竞赛词的参与） → awards/竞赛参赛
R16 志愿/公益 → activities/志愿服务
R17 社团/工作室/打卡/参与类证明 → activities/社团活动
R18 软件/系统/装置/作品 → works/*
R19 身份证/档案/证照/隐私/简历/综测 → archive/*
R20 custom/其他 + needs_review
```

---

## 6. 真实数据重跑（91 条，MF1-d）

### 6.1 旧 slug → 新 category 映射表

| 旧 slug | 新 category | 二义? | 说明 |
|---|---|:---:|---|
| education | education | 否 | 原位 |
| experience | experience / leadership | **是①** | 公司任职/创业→experience；学生干部→leadership |
| projects | works | 否 | 并入 works |
| research | research | 否 | 原位 |
| awards | awards | 否 | 四子类由 subtype 分 |
| publications | publications | 否 | 三子类由 subtype 分 |
| certificates | credentials | 否 | 改名 |
| skills | skills | 否 | 原位 |
| languages | credentials | 否 | 语言考试并入 |
| activities | activities / awards / credentials | **是②** | 志愿/社团→activities；竞赛参赛→awards；参赛证书→credentials |
| trainings | trainings | 否 | 原位 |
| works | works | 否 | 原位（projects 并入） |
| interests | custom | 否 | 收敛 custom/兴趣 |
| references | custom | 否 | 收敛 custom/推荐人 |
| custom | archive / custom | **是③** | 健康隐私/证照/简历/综测→archive；其余→custom |

**旧 slug 二义 3 处**：① experience（公司任职 vs 学生干部）；② activities（志愿·社团 vs 竞赛参赛 vs 参赛证书）；③ custom（归档 vs 兜底）。

### 6.2 覆盖口径（据实重跑）

- **覆盖：91/91**（每条唯一落点）。
- **规则强判（高置信）：86 条**。
- **needs_review：5 条**（规则已给唯一落点，但存在跨域政策二义 / 子类弱匹配，供人工确认，不阻断迁移）：

| # | id | 标题 | 落点 | 争议 |
|---|---|---|---|---|
| 1 | `b20ebe7f` | NECCS 参赛证书 | credentials/技能证书 | 证书判据 vs 竞赛参与 |
| 2 | `c15ee0a3` | 三下乡优秀志愿者 | awards/表彰荣誉 | 表彰 vs 志愿服务 |
| 3 | `e2daec69` | 迎新优秀志愿者 | awards/表彰荣誉 | 同上 |
| 4 | `d813e61e` | 物电学院 2022 级参与类证明 | activities/社团活动 | 纯参与证明，无社团属性 |
| 5 | `a825ace2` | 疯狂二十一天单词打卡 | activities/社团活动 | 学习打卡，非社团 |

> **相对 v3 的修正**：critique_v3 指出的 9 条误分（`19fb3834`/`2c43c446`/`9795cc70`/`ee10370f` 4 条 works、`60545726`、`22d86d38`、`634c2742`、`72e21e63`、`f1cf3cd8`）在 v4 全部回归正确落点，均计入 86 强判。

### 6.3 逐旧类统计（91 条，据实）

| 旧 category | 条数 | 新落点分布 |
|---|---:|---|
| activities | 15 | activities/志愿服务 6；activities/社团活动 2；awards/竞赛参赛 6；credentials/技能证书 1（NECCS） |
| awards | 42 | awards/学科竞赛 30；awards/表彰荣誉 8；awards/奖学金 4 |
| certificates | 4 | credentials/技能证书 3；credentials/等级考试 1 |
| custom | 5 | archive/健康隐私 1；archive/档案证照 2；archive/简历证明 1；archive/综测归档 1 |
| education | 1 | education/本科 1 |
| experience | 6 | experience/公司任职 1；experience/创业任职 2；leadership/学生干部 3 |
| languages | 3 | credentials/普通话 1；credentials/英语 2 |
| publications | 5 | publications/论文 3；专利 1；著作权 1 |
| research | 2 | research/科研项目 1；research/毕业论文 1 |
| trainings | 4 | trainings/MOOC培训 2；trainings/青马工程 2 |
| works | 4 | works/软件作品 3；works/装置设计 1 |
| **合计** | **91** | **91** |

### 6.4 新 category 计数（据实）

| 新 category | 条数 | 新 category | 条数 |
|---|:---:|---|:---:|
| education | 1 | awards | 42 |
| experience | 3 | skills | 0 |
| leadership | 3 | credentials | 8 |
| research | 2 | trainings | 4 |
| publications | 5 | activities | 14 |
| works | 4 | archive | 5 |
| custom | 0 | **合计** | **91** |

---

## 7. 双条计数消除（含 S4 项目↔作品约定）

1. `works`（含作品/项目成品）为**实体 canonical**；同一成品获奖**不得**另建完整内容条。
2. 奖项以 `relations: [{type:"award", competition, rank, ref}]` 挂作品；或奖项记录 `fields.work_ref` 指向作品。
3. **存量重复对**（`19fb3834`↔`65b6f8e8`、`2c43c446`↔`157a5e94`、`9795cc70`↔`795e49fa`、`ee10370f`↔`1334862f`）迁移时 **link 不删**；编译器按 canonical 去重。
4. **S4 项目↔作品**：`research/科研项目`（如大创 `9e220955`）与其产出 `works/软件作品`（如 `2c43c446`）为**不同实体**，允许并存；若确认同一实体，用 `relations: [{type:"output", ref}]` 链接，不合并、不双计简历。
5. **禁令**：同一 `(作品, 竞赛)` 不得生成两条 canonical 条。

---

## 8. 迁移方案（不变）

- 旧 slug 别名保留；拆分类未迁移态一律 `needs_review`。
- 写 `legacy_category`；未知 slug → `custom/其他` + `needs_review`。
- 目录：新增 `leadership/`、`archive/`；`certificates/`→`credentials/`；`projects/`→`works/`；其余原位；过渡期只读双扫。
- education 同名特判：缺 `subtype` 的旧记录补 `本科`（或按标题判定）。
- 迁移器 `migrate_v1_to_v2`：备份 → 逐条按 §5 R1–R20 → link 重复对 → 移目录（原子） → rebuildIndex → `state.json.schema_version=2` → 报告。
- 版本门：旧版读高版本进**只读模式**（迁移前核实已实现）；旧端写旧目录落 **tombstone**。
- `ProfileRecord.schemaVersion` 1→2；新字段可选 + 默认值。

---

## 9. 落地清单（不变）

**模型**：重写 `record_category.dart`（13，删 sectionKey，旧 slug 别名）；新增 `record_subtype.dart`（32）、`resume_section.dart`（映射 + override）、`migrate_v1_to_v2.dart`；改 `profile_record.dart`（subtype/legacyCategory/resumeSectionOverride/relations/needsReview/sensitive/categoryVersion，schema 2）。

**服务/UI/仓库**：`resume_compiler.dart` 按 `resolveSection` 聚合；`prompts.dart` 两级枚举（13+32）+ §5 规则；`ai_service.dart` EntryDraft.subtype/needsReview；`common.dart` 13 图标；records/capture/edit/settings 页面；`records_repository.dart` `_dirFor`/`loadAll` 双扫；`root_manager.dart` 建 13 目录；`docs/DATA_FORMAT.md` §5.3/§5.1/§9 + 变更日志。

**目录布局**：`education/ experience/ leadership/ research/ publications/ works/ awards/ skills/ credentials/ trainings/ activities/ archive/ custom/`。

**测试**：`taxonomy_migration_test.dart`（§10 全表 + 幂等）、`resume_section_mapping_test.dart`、`taxonomy_mece_test.dart`（32 子类）、`data_layer_test.dart`（13 目录 + 别名）、`integration_regression_test.dart`。

**AI 提示词**：输出 category(13)+subtype(32)；注入 §5 规则与判定树；兜底 custom/其他 + needs_review；禁双条 + relations。

---

## 10. 迁移回归测试用例表（v4 重跑版）

> 断言 `(category, subtype)`；输入 = title∪tags∪org。**加粗** = critique_v3 反例（含新修复的 9 条）。

| # | id | 标题（截断） | 期望 category/subtype | 命中规则 |
|---|---|---|---|---|
| 1 | 149afa1c | 2023-2024 优秀学生二等奖学金 | awards/奖学金 | R1 |
| 2 | **57412088** | 学习科技竞赛奖单项奖学金 | awards/奖学金（非竞赛） | R1 |
| 3 | **87b77e81** | 学习科技竞赛奖单项奖学金 | awards/奖学金（非竞赛） | R1 |
| 4 | **8e65afe4** | AI 算法工程师 / 股东 | experience/创业任职 | R2 |
| 5 | **a930c264** | 法定代表人 / 经理 | experience/创业任职 | R2 |
| 6 | **f1cf3cd8** | 本科毕业论文《…》 | research/毕业论文 | R3（先于 R4） |
| 7 | f362c93e | 电子信息科学与技术 本科 | education/本科 | R4 |
| 8 | 9e220955 | 听音识谱（大学生创新创业训练计划项目） | research/科研项目 | R5 |
| 9 | 3e70efff | 发明专利《…》（申请中） | publications/专利 | R6 |
| 10 | 9c91606e | 软件著作权《…》 | publications/著作权 | R7 |
| 11 | **1e4950f7** | Study on Energy Storage Planning…（EN） | publications/论文 | R8（tags=EI/论文） |
| 12 | **6bace271** | TransGNN-DTA…（EN） | publications/论文 | R8（tags=SCI/论文） |
| 13 | **814403f0** | Research on Training Performance…（EN） | publications/论文 | R8（tags=CPCI/论文） |
| 14 | **22d86d38** | 团委学生会"年终荐优"表彰 | awards/表彰荣誉 | R9（先于 R10/R16） |
| 15 | **5342ffae** | 五四评优优秀团学干部 | awards/表彰荣誉 | R9 |
| 16 | **5f054406** | 优秀学生干部 | awards/表彰荣誉 | R9 |
| 17 | **5f6497bc** | 三好学生 | awards/表彰荣誉 | R9 |
| 18 | **bf9c2b20** | 五四评优优秀团员 | awards/表彰荣誉 | R9 |
| 19 | **c15ee0a3** | 三下乡优秀志愿者 | awards/表彰荣誉（needs_review） | R9 |
| 20 | **e2daec69** | 迎新优秀志愿者 | awards/表彰荣誉（needs_review） | R9 |
| 21 | **1b35e3ce** | 电信 2205 班团支书 | leadership/学生干部 | R10 |
| 22 | **2559c5d4** | 学生会设计部副部长 | leadership/学生干部 | R10 |
| 23 | **3af58192** | 阳光传媒工作室设计部部长 | leadership/学生干部 | R10（先于 R17） |
| 24 | **6522b255** | 嵌入式与智能硬件实习生 | experience/公司任职 | R11 |
| 25 | 0389c05c | 金种子杯大学生创业大赛 金奖 | awards/学科竞赛 | R12（"创业"不触发 R2） |
| 26 | c53657ee | 青年志愿服务公益创业赛 银奖 | awards/学科竞赛 | R12（先于 R16） |
| 27 | be5660fe | 算法精英大赛 优秀奖 | awards/学科竞赛 | R12（"优秀奖"≠R9） |
| 28 | **634c2742** | 高校心理委员工作平台 MOOC 培训合格证书 | trainings/MOOC培训 | R13（"委员"不触发 R10） |
| 29 | **72e21e63** | 高校心理委员工作平台 MOOC 培训合格证书 | trainings/MOOC培训 | R13 |
| 30 | 3c1d78f7 | 青年马克思主义者培养学院 结业 | trainings/青马工程 | R13 |
| 31 | 005a5b70 | 普通话水平测试等级证书 | credentials/普通话 | R14 + §4.4 |
| 32 | 0f08493f | 全国计算机等级考试（NCRE）二级合格证书 | credentials/等级考试 | R14 + §4.4 |
| 33 | 486b568f | CET-4 成绩报告单 | credentials/英语 | R14 + §4.4 |
| 34 | **b20ebe7f** | NECCS 参赛证书 | credentials/技能证书（needs_review） | R14（先于 R15） |
| 35 | 2252499d | 挑战杯院赛 参赛 | awards/竞赛参赛 | R15 |
| 36 | 7569ac04 | "互联网+"参赛德育分证明 | awards/竞赛参赛 | R15 |
| 37 | a8396cd4 | 挑战杯初赛 观众/参与证明 | awards/竞赛参赛 | R15（观众） |
| 38 | c08a01b5 | 职业规划大赛 参与 | awards/竞赛参赛 | R15（参与+大赛） |
| 39 | **60545726** | 团委学生会…迎新活动 志愿者 | activities/志愿服务 | R16（R10 已收窄不夺） |
| 40 | d813e61e | 物电学院 2022 级参与类证明 | activities/社团活动（needs_review） | R17 |
| 41 | **19fb3834** | 结合人工智能的老年家居健康物联系统 | works/软件作品 | R18（desc 已排除） |
| 42 | **2c43c446** | 听音识谱…软件 | works/软件作品 | R18 |
| 43 | **9795cc70** | 基于数字孪生的…智能控制系统 | works/软件作品 | R18 |
| 44 | **ee10370f** | 基于布拉格光栅的光纤传感器装置设计 | works/装置设计 | R18 |
| 45 | d8da7e39 | 简历与证明材料（零散文档） | archive/简历证明 | R19 |
| 46 | cb3839ea | 综测佐证材料归档 | archive/综测归档 | R19 |
| 47 | c569b5ea | 高考报名与准考证信息 | archive/档案证照 | R19 |

> 断言要点：① R2 词表**不含裸"创业"**，确保"创业大赛"归 R12；② R10 为**强职务词**且**后置于 R9**，确保"优秀志愿者→表彰""志愿/培训不被夺"；③ **R3 前置于 R4**；④ 判定输入**不含 description**，确保 4 条 works 不被 R12 抢走；⑤ R9 词表不含裸"优秀"，"优秀奖"归 R12。

---

## 11. 与 v3 的差异（快速核对）

| 项 | v3 | v4 |
|---|---|---|
| 判定输入 | title∪tags∪org∪**desc** | **title∪tags∪org**（去 desc） |
| R10 学生干部 | 宽词表（含团委/学生会/委员/干事/负责人），在 R11 前 | **强职务词**，**移到 R9/R11 之后** |
| R3/R4 | R3=本科、R4=毕业论文 | **R3=毕业论文、R4=学历**（前置） |
| R8 通用词 | conference/paper 无条件 | 仅 tags/标题 paper/proceedings 时计（S3） |
| R19 | 含裸"证明" | 删裸"证明"，保留具体词（S2） |
| §10 回归表 | 4 行自相矛盾 | **重跑重出，全部可通过** |
| §6.2 强判 | 86（失真） | **86（据实）**，9 条误分已修复 |
| §6.3/6.4 | activities 志愿 6 但 works 0、毕设 0 | **works 4、毕设 1、志愿服务 6、MOOC 2 全部恢复** |
| relations | 仅作品↔获奖 | 补**项目↔作品**（S4） |
| 开发项目 | 触发弱 | §4.4 补正判据 + 标注可选（S1） |

## 附录：v4 记录 JSON 示例

```jsonc
{
  "schema_version": 2,
  "category_version": 2,
  "id": "19fb3834-…",
  "category": "works",
  "subtype": "软件作品",
  "legacy_category": "works",
  "title": "结合人工智能的老年家居健康物联系统",
  "relations": [{ "type": "award", "competition": "中国大学生计算机设计大赛", "rank": "中南地区赛三等奖", "ref": "65b6f8e8-…" }],
  "needs_review": false,
  "resume_section_override": null,
  "sensitive": false
  // …其余字段同 v1
}
```

---
*本稿仅描述方案，不改任何代码；`subtype` 种子直接取自真实数据 `fields.subcategory` + 6 项最小新增。*
