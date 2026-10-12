# anti-rea

防御 AI 辅助逆向的 Agent Skill：判断你自己的网站、桌面程序和 App 在 [rea](https://github.com/morluto/rea) 这类工具面前暴露了什么，并按「架构优先、加固其次」给出可执行的防护方案。

An agent skill for hardening your own software and websites against AI-agent-assisted reverse engineering.

## 背景

rea 把 Ghidra、Hopper、IDA、JADX、Chrome 等逆向工具封装成 MCP，让 AI agent 直接调用。逆向的门槛和时间成本因此大幅下降，「代码压缩过就没人看得懂」的假设不再成立。本 skill 的立场是：交付给用户的代码默认会被读懂，防护的重点放在让读懂的人拿不走有价值的东西。

## 它会做什么

- 按交付形态（网站、Electron、原生程序、.NET、Go、Android、iOS、Flutter、React Native、小程序）梳理攻击者能看到的内容
- 优先给出架构调整：核心逻辑、密钥、权限判断移到服务端
- 给出各平台发布构建的加固配置，并说明每项措施的代价
- 有构建产物时实际运行自查，输出带证据、按严重度排序的评估报告
- 补充泄露检测手段和法律边界说明

## 安装

Claude Code（个人级）：

```bash
git clone https://github.com/DreamFields/anti-rea.git ~/.claude/skills/anti-rea
```

项目级安装时，克隆到项目的 `.claude/skills/anti-rea`。其他支持 Agent Skills 的工具，把整个目录放进对应的 skills 目录即可。

## 使用示例

- 我的 Vite + React 网站怎么防止被人扒代码？
- 帮我检查 dist 目录有没有泄露 source map 和密钥
- 我们的 Electron 客户端值得上加固吗？
- 这个 Android App 的付费功能怎么防破解？

## 目录结构

```text
anti-rea/
├── SKILL.md                     入口：核心判断、工作流程、输出格式
└── references/
    ├── threat-model.md          AI 辅助逆向能看到什么
    ├── architecture.md          架构层防线
    ├── web.md                   网站、前端、小程序
    ├── electron.md              Electron
    ├── native-desktop.md        原生程序、.NET、Go、Java、Python
    ├── mobile.md                Android、iOS、Flutter、React Native
    ├── self-audit.md            发布前自查命令
    └── detection-and-legal.md   检测追踪与法律边界
```

## 适用范围

本 skill 只用于保护你自己拥有或获得授权的软件，不提供绕过他人软件保护的帮助。

## 资料来源

- rea 仓库 README 与文档（2026-10-12 查阅）
- 各框架与平台的官方文档

## License

[MIT](LICENSE)
