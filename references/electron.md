# Electron

## 先认清 ASAR

- `app.asar` 只是归档格式，任何人都能解包查看全部 JS。主进程、preload 和渲染进程的代码都按公开代码对待：不放密钥，核心逻辑放服务端。
- 打包前确认哪些文件进了 ASAR。测试文件、`.env`、source map、私钥都不应出现在里面（检查命令见 self-audit.md）。

## 用 Fuses 防篡改和注入

用 `@electron/fuses` 在打包时设置（Electron Forge 有 fuses 插件，electron-builder 也提供对应配置）：

| Fuse | 建议 | 作用 |
| --- | --- | --- |
| `RunAsNode` | 关闭 | 阻止通过 `ELECTRON_RUN_AS_NODE` 把应用当作 Node 运行 |
| `EnableNodeOptionsEnvironmentVariable` | 关闭 | 阻止通过 `NODE_OPTIONS` 注入参数 |
| `EnableNodeCliInspectArguments` | 关闭 | 阻止通过 `--inspect` 等参数附加调试器 |
| `EnableEmbeddedAsarIntegrityValidation` | 开启 | 启动时校验 ASAR 完整性（macOS、Windows） |
| `OnlyLoadAppFromAsar` | 开启 | 只从 ASAR 加载应用代码 |

这些设置防的是有人修改你的应用，或借它在用户机器上注入代码；它们挡不住阅读。用 `npx @electron/fuses read --app <应用路径>` 核对发布版的实际状态。

## 安全配置

- 保持 `contextIsolation: true`、`sandbox: true`、`nodeIntegration: false`。新版本默认就是这样，不要为了方便关掉。
- preload 通过 `contextBridge` 暴露的 API 越少越好，每个函数只做一件具体的事，不要把通用的 `ipcRenderer.send` 或 `invoke` 直接暴露出去。
- 主进程处理每个 IPC 时校验发送方（`event.senderFrame` 的来源）和参数。通道名可以被枚举出来，要假设任何通道都可能收到构造过的参数。

## 分发与更新

- macOS 签名并公证，Windows 做代码签名，让用户和系统能识别被篡改的副本。
- 自动更新走 HTTPS，并校验更新包签名（electron-updater 等方案支持）。

## 可选的减速带

- 把 JS 编译成 V8 字节码（如 bytenode）或使用 V8 snapshot，可以增加阅读成本，但会带来版本兼容和调试成本，也不改变核心逻辑放服务端的原则。
