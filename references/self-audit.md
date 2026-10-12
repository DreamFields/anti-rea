# 发布前自查

用这些检查确认攻击者拿到构建产物后能看到什么。只对用户自己的项目运行。命令中的 `dist`、`<应用路径>` 等替换成实际路径。

## 报告规则

- 找到疑似密钥时，只报告文件位置、类型和打码后的值，完整的值不写进对话或报告。
- 已经发布过的真实密钥，提醒用户立即到服务商处轮换。

## 网站与前端

是否带 source map：

```bash
find dist -name "*.map"
```

```bash
grep -rln "sourceMappingURL" dist
```

线上再确认一次：对一个实际的 JS 地址加上 `.map` 请求，应返回 404 或 403。

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://<你的域名>/assets/<文件名>.js.map
```

构建产物里有没有密钥（需要安装 gitleaks）：

```bash
gitleaks dir dist
```

源码里用到的公开前缀环境变量，只列变量名、不打印值，逐个确认可以公开：

```bash
grep -rhoE "(VITE|NEXT_PUBLIC|REACT_APP|EXPO_PUBLIC)_[A-Z0-9_]+" src | sort -u
```

产物里出现的域名，检查有没有测试环境和内网地址：

```bash
grep -rhoE "https?://[A-Za-z0-9.-]+" dist | sort -u
```

接口返回字段需要手动检查：在浏览器开发者工具的 Network 面板打开主要页面，对比每个接口返回的字段和页面实际显示的字段，多出来的字段都有泄露风险。

## Electron

ASAR 路径：macOS 在 `<应用>.app/Contents/Resources/app.asar`，Windows 在安装目录的 `resources\app.asar`。

查看 ASAR 里打包了哪些文件：

```bash
npx @electron/asar list <ASAR 路径>
```

解包后扫描密钥：

```bash
npx @electron/asar extract <ASAR 路径> asar-audit
```

```bash
gitleaks dir asar-audit
```

核对 Fuses 是否按 electron.md 设置：

```bash
npx @electron/fuses read --app <应用路径>
```

## 原生程序

是否去掉了符号。Linux 上 `file` 输出 `not stripped` 说明还带符号表：

```bash
file <程序>
```

macOS 上能在输出里看到自己的函数名，说明没去干净：

```bash
nm -U <程序> | head -20
```

可读字符串中有没有敏感线索：

```bash
strings -n 8 <程序> | grep -iE "secret|token|password|api[_-]?key|internal|staging|debug"
```

发布目录里不应出现调试符号文件：

```bash
find <发布目录> -name "*.pdb" -o -name "*.dSYM"
```

## .NET 与 Go

- .NET：确认发布方式是 NativeAOT 还是普通 IL 程序集，并确认发布目录里没有 `.pdb`（命令同上）。
- Go：查看嵌入的构建信息里有没有不想公开的路径和版本控制信息：

```bash
go version -m <程序>
```

## Android

解包 release APK 后扫描密钥，同时扫描源码中的资源目录：

```bash
unzip -q -o app-release.apk -d apk-audit
```

```bash
gitleaks dir apk-audit
```

```bash
gitleaks dir app/src/main
```

另外确认 release 构建开启了 R8，`mapping.txt` 没有进入安装包或公开仓库。

## 用 rea 看攻击者视角（可选）

用户想直观看到攻击者能拿到什么时，可以用 rea 分析自己的构建产物。rea 的免责声明明确支持审计自己拥有的软件。

```bash
npx -y rea-agents@latest analyze-javascript-application /绝对路径/dist --json
```

- 命令行运行时结果留在本地；通过 AI agent 调用时，结果会发送给模型服务商，涉及保密代码时先和用户确认。
- 原生程序分析需要先配置 Hopper、Ghidra 或 IDA，参见 rea 的安装文档。
