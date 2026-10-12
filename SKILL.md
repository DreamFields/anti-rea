---
name: anti-rea
description: 保护用户自己的软件和网站，降低被 AI 辅助逆向读懂、复刻或滥用的风险（典型工具是 morluto/rea，一个让 agent 调用 Ghidra、IDA、JADX、浏览器等工具的逆向 MCP）。当用户问怎样防止自己的网站、前端代码、Electron 或桌面客户端、原生程序、.NET/Go 程序、Android/iOS/Flutter/React Native App、小程序被逆向、破解、扒代码或抄功能，想做发布前加固，想检查构建产物是否泄露 source map、密钥、内部接口，或想评估混淆、加固方案值不值得时，都应使用本 skill，即使用户没有提到 rea。Defensive hardening of your own apps and websites against AI-agent-assisted reverse engineering, covering threat model, server-side architecture, per-platform build hardening, release self-audit, detection and legal notes.
---

# anti-rea：防御 AI 辅助逆向

## 核心判断

rea（https://github.com/morluto/rea）把 Ghidra、Hopper、IDA、JADX、Chrome 浏览器和 source map 解析等现成工具封装成 MCP，Claude Code、Codex、Cursor 等 agent 可以直接调用。这些能力本身早就存在，变化在于成本：过去要逆向专家花几天的分析，现在用一句自然语言就能启动；大模型读压缩、改名后的代码和反编译伪代码也很在行。

本 skill 的基本立场：

- **交付给用户的东西默认会被完全读懂**，包括 JS、APK、IPA、exe、Electron 包和小程序代码包。防护目标定为：即使读懂了，也拿不走有价值的东西。
- **架构是防线，混淆和加固是减速带。** 核心逻辑、密钥和权限判断放在服务端才真正安全。客户端措施只能提高成本，推荐时要连同代价一起说明。
- **工具的免责声明不构成防护。** rea 声明不支持破解授权、复制专有软件，这只是社区规则，没有技术约束力。

## 适用范围

本 skill 用于保护用户自己拥有或获得授权的软件与网站。如果请求实际上是绕过别人软件的保护、破解授权或复制他人代码，这部分不提供帮助。

## 工作流程

1. **弄清保护对象。** 确认值钱的是什么（算法、数据、付费功能、接口额度、防仿冒），以什么形态交付给用户，能否访问项目代码或构建产物。能从项目里读到的信息（构建配置、环境变量用法、目录结构）先自己读，不要反复追问用户。
2. **对照威胁模型。** 读 [references/threat-model.md](references/threat-model.md)，按交付形态列出攻击者能直接看到什么，再标出其中哪些是用户真正在乎的。
3. **先给架构建议。** 读 [references/architecture.md](references/architecture.md)。把值钱的东西移出客户端，通常能一次解决大部分问题，效果超过任何加固。
4. **再按平台加固。** 只读与交付形态相关的文档：

   | 交付形态 | 阅读 |
   | --- | --- |
   | 网站、前端、小程序 | [references/web.md](references/web.md) |
   | Electron 桌面应用 | [references/electron.md](references/electron.md)，前端部分另读 [references/web.md](references/web.md) |
   | 原生程序、.NET、Go、Java、Python 打包程序 | [references/native-desktop.md](references/native-desktop.md) |
   | Android、iOS、Flutter、React Native | [references/mobile.md](references/mobile.md) |

5. **能验证就实际检查。** 拿得到构建产物或项目代码时，按 [references/self-audit.md](references/self-audit.md) 运行检查，用结果说话；拿不到时把对应的检查清单交给用户。
6. **补充检测与法律。** 涉及泄露追踪、维权，或用户问到法律边界时，读 [references/detection-and-legal.md](references/detection-and-legal.md)。

## 输出格式

完整评估使用下面的结构；用户只问一个具体问题时，直接回答相关部分即可。

```markdown
# 防逆向评估：<项目名或交付形态>

## 结论
一两句话：最大的风险是什么，最该先做哪件事。

## 发现
按严重度（高 / 中 / 低）排列。每条写明位置或证据、为什么是风险、怎么修。
没有实际检查过的条目标注「未验证」。

## 架构调整
需要移到服务端的逻辑、密钥和权限判断。

## 加固清单
- 今天就能做：
- 近期：
- 视价值决定：

## 代价与残余风险
混淆、加固等措施的性能、兼容性和维护成本，以及做完之后仍然存在的风险。
```

## 写作原则

- **区分已验证和建议。** 运行检查得出的结论附上证据；推断和通用建议要标明。
- **如实说明客户端措施的效果。** 每推荐一项混淆或加固，都说清它能拖延什么、拦不住什么。
- **发现真实密钥时**，报告里只写位置、类型和打码后的值（如 `sk-…9f3a`），并提醒用户立即到服务商处轮换。已经发布出去的密钥，从下一个版本删掉并不能让它失效。
- **按收益排序。** 关闭公开的 source map、清理密钥、精简接口返回字段通常比引入商业加固划算，先推荐这些。
- **用 rea 自查时**只分析用户自己的产物，并提醒：通过 agent 调用时，分析结果会发送给模型服务商。

## 资料来源

威胁模型依据 rea 仓库 README 和文档（2026-10-12 查阅）。rea 迭代很快，描述它的具体能力前，可以重新查看其 README 中 "What you can analyze" 一节确认。
