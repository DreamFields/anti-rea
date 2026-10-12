# 移动端：Android、iOS、Flutter、React Native

## Android

- 开启 R8：`isMinifyEnabled = true`、`isShrinkResources = true`（AGP 8 起默认使用 full mode）。
- `mapping.txt` 只上传到 Play Console 或崩溃上报平台，不随包发布，也不提交到公开仓库。
- 不要在 `strings.xml`、`BuildConfig`、`assets`、`res/raw` 里放密钥。
- NDK 原生库同样可以被分析，不能当保险箱用；去符号等做法参照 native-desktop.md。
- 敏感操作接入 Play Integrity API：由服务端验证完整性判定结果，再决定是否放行。

## iOS

- 从 App Store 分发的程序同样可以被分析，不要指望平台机制保护代码。
- 发布构建去符号，dSYM 只上传到崩溃上报平台。
- 敏感操作接入 App Attest（DeviceCheck 框架），由服务端验证请求来自未被修改的正版 App。
- 用户相关的密钥存进 Keychain，不要硬编码在代码或 `Info.plist` 中。

## Flutter

- 发布时使用 `--obfuscate --split-debug-info=<目录>`，符号目录只保存在内部。
- Dart 编译出的原生库依然可以被分析，核心逻辑放服务端。

## React Native 与 Expo

- Hermes 字节码只增加阅读成本。
- `EXPO_PUBLIC_*` 变量和打进包里的配置都是公开的，密钥走后端。

## 网络与运行环境

- 证书固定（certificate pinning）能保护用户免受中间人攻击，也提高了分析流量的成本，但它不是安全边界。要准备备用证书和轮换方案，避免证书更新导致应用无法使用。
- root、越狱、调试器和模拟器检测，适合作为风险信号上报服务端参与风控评分，不要当作唯一的拦截手段。
- 可以参考 OWASP MASVS 的 MASVS-RESILIENCE 类别，按应用价值决定做到什么程度。
