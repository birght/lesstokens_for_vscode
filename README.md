# lesstokens_for_vscode
基于caveman。针对vscode配置了全局提示词文件



# Copilot Chat 省 Token 提示词方案

> 基于 [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) 项目理念，适配 VS Code GitHub Copilot Chat 的中文版实现。  
> 核心原理：通过系统级提示词约束 LLM 输出风格，去掉废话和冗余表达，保留全部技术实质，节省 ~65% 输出 token。

---

## 一、文件清单

所有文件位于 **VS Code 用户级目录**，跨项目通用，不污染任何仓库。

```
%APPDATA%\Code\User\prompts\
├── caveman-lite.instructions.md   ← 🔄 自动注入（每次对话生效）
├── caveman-lite.prompt.md         ← 📋 按需选择：🪶 Lite
├── caveman-full.prompt.md         ← 📋 按需选择：🪨 Full
├── caveman-ultra.prompt.md        ← 📋 按需选择：🔥 Ultra
└── caveman-wenyan.prompt.md       ← 📋 按需选择：📜 文言文
```

> Windows 路径：`C:\Users\{用户名}\AppData\Roaming\Code\User\prompts\`

| 文件 | 类型 | 作用 |
|------|------|------|
| `caveman-lite.instructions.md` | 自动注入 | `applyTo: "**"` 匹配所有文件，**每次 Copilot 对话自动生效** |
| `caveman-lite.prompt.md` | 按需 | 🪶 去废话保语法，专业简练 |
| `caveman-full.prompt.md` | 按需 | 🪨 去冠词、片段式表达，经典穴居人风格 |
| `caveman-ultra.prompt.md` | 按需 | 🔥 电报式极限压缩，缩写+箭头因果链 |
| `caveman-wenyan.prompt.md` | 按需 | 📜 古典中文压缩，80-90% 字符缩减 |

**两种文件的区别**：
- `.instructions.md`：自动注入，通过 `applyTo` 匹配文件类型，无需手动触发
- `.prompt.md`：按需选择，在 Chat 输入框中主动引用才生效

---

## 二、配置步骤

### 步骤 1：创建目录

```powershell
mkdir "$env:APPDATA\Code\User\prompts"
```

### 步骤 2：创建自动注入文件（核心，做完即生效）

创建 `caveman-lite.instructions.md`：

```markdown
---
applyTo: "**"
---

回复精简高效。删除填充词（就是/其实/基本上/简单来说）、客套话（当然可以/很高兴帮你）、模棱两可措辞。句子简短有力，用短同义词。技术术语精确不变，代码块原样。模式：[事物] [动作] [原因]。[下一步]。安全警告和不可逆操作时恢复正常表达。
```

### 步骤 3（可选）：创建按需级别文件

将 4 个 `.prompt.md` 文件放入同一目录。文件内容见本文档附录。

### 步骤 4：工作区 settings.json 配置

在项目 `.vscode/settings.json` 中添加：

```jsonc
{
  // 启用用户级 prompts 目录中的 .instructions.md / .prompt.md
  "chat.promptFiles": true,
  // 启用 Agent Skills（支持 .prompt.md 按需选择）
  "chat.useAgentSkills": true,
  // Agent 模式单次会话最大请求数
  "chat.agent.maxRequests": 50
}
```

> `chat.promptFiles` 和 `chat.useAgentSkills` 也可以在用户级 settings.json 中配置，效果一致。

---

## 三、使用方式

### 默认行为（自动）

`caveman-lite.instructions.md` 已通过 `applyTo: "**"` 全局注入，**打开任何 Copilot Chat 对话即自动生效**，无需操作。

### `#` 命令引用提示词文件（按需升级）

在 Copilot Chat 输入框中，通过 `#` 命令可以引用 `.prompt.md` 文件：

```
# 操作步骤：
1. 在 Chat 输入框输入 #
2. 在弹出的列表中选择 "Prompt..."（或直接输入文件名模糊匹配）
3. 选择目标级别的 .prompt.md 文件
4. 输入你的问题，发送即可
```

**实际输入示例**：

```
#caveman-full 解释下 Vue 的 nextTick 原理
```

```
#caveman-ultra 这段代码有什么性能问题？
```

```
#caveman-wenyan pinia 和 vuex 区别
```

> `#` 引用的 `.prompt.md` 仅对**当次对话**生效，不影响其他对话。关闭对话后自动恢复默认 lite 级别。

### 操作速查表

| 场景 | 操作 | 效果 |
|------|------|------|
| **日常（默认）** | 无需操作 | `.instructions.md` 自动注入 lite 级别 |
| **升级到 Full** | 输入 `#` → 选 `caveman-full` | 当次对话切换到穴居人模式 |
| **升级到 Ultra** | 输入 `#` → 选 `caveman-ultra` | 电报式极限压缩 |
| **切换文言文** | 输入 `#` → 选 `caveman-wenyan` | 古典中文输出 |
| **停止** | 对话中说"正常模式"或"停止精简" | 恢复标准回复 |

---

## 四、效果对比

同一问题："Vue ref 和 reactive 区别"

| 级别 | 回复示例 |
|------|----------|
| **无 caveman** | "当然可以！`ref` 和 `reactive` 都是 Vue 3 组合式 API 中的响应式工具。简单来说，`ref` 主要用于基本类型的值，它会将值包装在一个带有 `.value` 属性的对象中……" |
| **🪶 Lite** | "`ref` 包装基本类型，访问用 `.value`。`reactive` 用于对象，直接访问属性。选择：基本类型用 ref，对象用 reactive。" |
| **🪨 Full** | "`ref`包基本类型，要`.value`。`reactive`用于对象，直接访问。基本类型→ref，对象→reactive。" |
| **🔥 Ultra** | "`ref`→基本类型+`.value`。`reactive`→obj直接访问。" |
| **📜 文言文** | "ref封基值，取以.value。reactive封物，直取其屬。" |

---

## 五、注意事项

1. **不要用 `settings.json` 的 `github.copilot.chat.codeGeneration.instructions`** — 该字段已被 VS Code 弃用，会报警告。正确方式是 `.instructions.md` 文件
2. `.instructions.md` 的 `applyTo: "**"` 匹配所有文件类型，全局自动生效
3. `.prompt.md` 不会自动注入，需要用户在 Chat 中主动 `#` 引用
4. 所有文件在用户级目录，**跨项目通用**，无需每个仓库重复配置
5. 安全警告、不可逆操作时会自动恢复正常表达，不影响关键信息传达

---

## 附录：各级别 .prompt.md 完整内容

### A. caveman-lite.prompt.md

```markdown
---
description: "🪨 Caveman Lite — 精简模式：去废话保语法，专业但无冗余"
mode: ask
---

以精简高效的方式回复。保留所有技术实质，只去掉废话。

## 规则

删除：填充词（就是/其实/基本上/简单来说/事实上）、客套话（当然可以/很高兴帮你/没问题）、模棱两可的措辞。
保留完整语法和冠词。句子简短有力。用短同义词（大 而非 大规模的，修 而非 实施一个解决方案）。
技术术语精确不变。代码块原样保留。错误信息精确引用。

模式：`[事物] [动作] [原因]。[下一步]。`

## 自动恢复清晰度

安全警告、不可逆操作确认、用户困惑时：暂停精简，恢复正常表达。完成后恢复。
```

### B. caveman-full.prompt.md

```markdown
---
description: "🪨 Caveman Full — 穴居人模式：去冠词、片段式、经典穴居风格"
mode: ask
---

像聪明的穴居人一样精炼回复。所有技术实质保留，只有废话消亡。

## 规则

删除：冠词（一个/这个/那个）、填充词、客套话、模棱两可措辞。
片段句OK。用短同义词。技术术语精确。代码块不动。

模式：`[东西] [动作] [原因]。[下一步]。`
```

### C. caveman-ultra.prompt.md

```markdown
---
description: "🔥 Caveman Ultra — 极限压缩：电报式、能缩写就缩写、箭头因果链"
mode: ask
---

极限压缩回复。缩写一切（DB/auth/config/req/res/fn/impl）。
砍连词。箭头表因果（X → Y）。一词够就一词。
```

### D. caveman-wenyan.prompt.md

```markdown
---
description: "📜 文言文模式 — 古典中文压缩：最高效的人类语言，80-90%字符缩减"
mode: ask
---

以文言文回复。技术实质全留，以古典中文最大压缩。
古典句式，动词前置，主语常省。用古典虚词（之/乃/為/其/則/故/以/而）。
现代技术术语、代码块、命令、路径、URL 原样保留不翻译。
```
