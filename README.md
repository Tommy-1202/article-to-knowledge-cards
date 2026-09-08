# make-knowledge-cards Skill

把一篇文章压缩成 5–8 张**只讲一个知识点**的学习卡片,纯 Markdown 输出。

---

## 1. 项目解决什么问题

读一篇技术博客、长文报告或讲义后,大多数人只能记住其中一两个核心概念,而真正"学进去"的内容往往更少。这个 Skill 用来替代手写笔记:

- **自动提取真正重要的知识点**,而不是照抄段落。
- **强制每张卡片只讲一件事**,避免一张卡片里塞三四个论点导致哪个都记不住。
- **不强行凑数量** —— 原文只有 3 个有价值的点,就输出 3 张卡片;绝不为了凑到 5 张而把感谢名单、营销套话、个人轶事抬升成"知识点"。
- **不编造原文没有的内容**,例子优先复用原文给出的,自测题也基于原文事实。

适合谁用:学生、自学者、技术博主、需要把长输入文档快速转成复习材料的工程师 / 研究人员。

## 2. 主要功能

| 功能 | 说明 |
|---|---|
| **输入** | (a) 聊天里直接粘贴的文章; (b) 本地 `.md` 或 `.txt` 文件的绝对路径 |
| **输出** | 一份 Markdown 卡片组,每张卡片包含:`标题` / `核心知识` / `解释` / `例子 或 自测` |
| **目标数量** | 5–8 张卡片;原文信息密度低时输出更少并标注 ⚠️,绝不凑数 |
| **抽取原则** | 一卡一知识点 / 去重 / 不编造 / 优先复用原文例子 |
| **质量自检** | 写完每张卡片都对照原文核验:核心知识是否准确?例子 / 自测是否 grounded?是否有两张卡片重复? |
| **明确的拒绝列表** | 不支持:URL / PDF / DOCX / EPUB / 图片 / Anki `.apkg` / HTML 页面 / 幻灯片 |

同时附带 `agents/openai.yaml`,可在 OpenAI 兼容的 agent 平台直接加载,把 system prompt / 卡片 schema / 抽取规则一次性注入。

## 3. 安装方法

这个项目没有运行时依赖,只有两个产物:

### 产物 A: Skill 包(Claude / WorkBuddy)

把 `skills/make-knowledge-cards/` 整个目录拷到目标客户端能扫描到的位置:

| 客户端 | 推荐位置 |
|---|---|
| Claude Code | `~/.claude/skills/make-knowledge-cards/` |
| WorkBuddy | 当前 workspace 下的 `skills/make-knowledge-cards/`(即本仓库结构) |
| 其他支持 Skills 协议的客户端 | `<config_dir>/skills/make-knowledge-cards/` |

拷贝完成后目录结构应该是:

```
skills/make-knowledge-cards/
├── SKILL.md
└── agents/
    └── openai.yaml
```

### 产物 B: Agent 配置(OpenAI / 其他兼容平台)

把 `skills/make-knowledge-cards/agents/openai.yaml` 作为 agent 定义文件导入平台即可。该文件已经包含 `name` / `description` / `model` / `temperature` / `system_prompt` / `tools` / `metadata` 等完整字段,无需修改。

### 验证安装

```bash
python <skill-creator>/scripts/quick_validate.py skills/make-knowledge-cards
```

预期输出:

```
Skill is valid!
```

## 4. 使用方法

加载完成后,在对话里直接描述需求即可。Skill 会按 `SKILL.md` 的 Workflow 自动完成:读源 → 评估知识点数 → 起草 → 自检 → 输出。

### 模式一:粘贴文章

> 把下面这段文章做成知识卡片:
>
> 在 Git 中整合不同分支的修改主要有两种方式:`merge` 和 `rebase`……

### 模式二:本地 Markdown / TXT 文件

> 用 `make-knowledge-cards` 处理 `~/notes/transformer.md`

> 把 `D:/workbuddy/article.txt` 转成知识卡片

### 不支持的形式(直接被拒绝)

| 输入 | 处理方式 |
|---|---|
| URL | 拒绝,提示"不支持网页抓取" |
| PDF / DOCX / EPUB | 拒绝,提示"不支持二进制格式,请先转 `.md` / `.txt`" |
| 图片 / 扫描件 | 拒绝,提示"不支持图像输入" |
| 要求输出 Anki `.apkg` / CSV | 拒绝,提示"只输出 Markdown" |
| 要求生成 HTML 页面 / 幻灯片 | 拒绝,提示"只输出 Markdown" |

## 5. 输入输出示例

### 示例 1:技术教程(完整 fixture 见 `test_fixtures/test_tech.md`)

**输入节选**:

> 在 Git 中整合不同分支的修改主要有两种方式:`merge` 和 `rebase`。`git merge` 会保留两个分支的所有提交历史,会在合并点创建一个新的 "merge commit"……`git rebase` 的核心思想是"重放提交"……永远不要对已经推送到公共仓库的提交做 rebase……

**输出节选**(完整版见 `test_outputs/out_tech.md`,共 6 张):

```markdown
# 知识卡片: Git Rebase 与 Merge 的区别

> 来源: skills/make-knowledge-cards/test_fixtures/test_tech.md · 共 6 张卡片

### 3. Rebase 的工作机制

**核心知识**: `git rebase` 把当前分支的所有提交"摘下来",在目标分支的最新提交之上依次重新应用,使历史呈现为一条直线。

**解释**: 应用 rebase 之后,你分支上的 commit 看起来就像是在目标分支的最新状态之后直接写的。commit 内容不变,但 hash 会全部重新生成……

**例子 / 自测**:
- 自测: 把 feature 分支 rebase 到 main 之后,feature 分支上的 commit hash 会发生变化还是保持不变?为什么?

### 4. Rebase 的铁律

**核心知识**: 永远不要对已经推送到公共共享仓库的提交执行 rebase。

**解释**: 因为 rebase 会重写 commit hash,导致其他协作者的本地历史和远程不一致……
```

### 示例 2:短文不凑数(`test_short.txt`,原文约 120 字)

原文只支撑 3 张卡片。Skill 输出 3 张并显式标注:

```markdown
# 知识卡片: 番茄工作法

> 来源: skills/make-knowledge-cards/test_fixtures/test_short.txt · 共 3 张卡片

> ⚠️ 原文信息密度较低,只生成了 3 张卡片。
```

### 示例 3:致辞 / 感谢演讲(实际案例 `out_yao_speech.md`)

原文 ~5500 字,但属于个人致辞,真正可作为"知识点"提取的只有 5 处(CBA 奖杯命名、NBA 中国球员第一人、汤姆贾诺维奇的"冠军之心"、范甘迪的"机会哲学"、演讲的"三镜"主题)。Skill 输出 5 张,明确标注:

> ⚠️ 原文为个人致辞/感谢演讲,信息密度较低,只提取出 5 张可作为"知识点"的卡片;个人叙事、感谢名单与轶事未纳入。

每张卡片的所有事实都直接对应原文某一行,未编造任何信息。

## 目录结构

```
.
├── README.md                                  # 本文件
└── skills/
    └── make-knowledge-cards/
        ├── SKILL.md                           # Skill 主文件(YAML frontmatter + 指令)
        ├── agents/
        │   └── openai.yaml                    # OpenAI 兼容的 agent 配置
        ├── test_fixtures/                     # 测试输入
        │   ├── test_tech.md                   #   技术教程
        │   ├── test_concept.md                #   概念科普
        │   ├── test_long.md                   #   长文摘要
        │   └── test_short.txt                 #   短文(测试不凑数)
        └── test_outputs/                      # 对应的卡片输出
            ├── out_tech.md
            ├── out_concept.md
            ├── out_long.md
            ├── out_short.md
            └── out_yao_speech.md              #   实际使用案例(姚明名人堂演讲)
```

## 验证

```bash
python <skill-creator>/scripts/quick_validate.py skills/make-knowledge-cards
```

预期输出:

```
Skill is valid!
```

四条 fixture 的实际运行结果保存在 `skills/make-knowledge-cards/test_outputs/`,可作为预期输出参考。

## License

MIT