# 网站、前端与小程序

## Source map

公开的 source map 能还原原始文件结构、模块名和注释，是收益最高的修复点。

- 生产环境不对外提供 `.map` 文件。需要线上错误堆栈时，生成 hidden source map，只上传到 Sentry 等错误监控平台，不部署到站点：
  - Vite：`build.sourcemap: 'hidden'`
  - webpack：`devtool: 'hidden-source-map'`
  - Next.js：保持 `productionBrowserSourceMaps: false`（默认值）
- 在 CDN 或网关上让 `*.map` 返回 404，防止误部署。

## 构建产物清理

- 去掉 `console` 和 `debugger`。例如 Vite 使用 esbuild 压缩时可配置 `esbuild: { drop: ['console', 'debugger'] }`，terser 使用 `compress.drop_console`；具体写法以所用版本的文档为准。
- 检查注释、测试账号、测试环境地址、内网域名有没有进入产物（命令见 self-audit.md）。
- 生产和测试环境分别构建，不要在同一个包里靠运行时判断切换地址。

## 逻辑放在服务端渲染层

- 敏感逻辑放在 SSR、React Server Components 或 API Route 中执行，这些代码不会下发到浏览器。
- Server Actions 本质上是可以被直接调用的 POST 接口，函数内部同样要做鉴权和参数校验。
- Next.js 中给只应在服务端运行的模块加上 `import 'server-only'`，一旦被客户端组件引用，构建就会报错。

## 混淆与 WebAssembly

- JavaScript 混淆器（如 javascript-obfuscator）会增大体积、降低性能、增加排错难度，而大模型去混淆的能力还在增强。只对少量确实需要拖延的代码使用。
- 把算法编译成 WebAssembly 也只是减速带，WASM 可以被转换成可读的文本格式（WAT）。

## 数据与抓取

- 页面能显示的数据，攻击者都能拿到。限制抓取要靠登录、限流、bot 防护和按用户划分的数据可见范围，前端加密起不到作用。
- 接口返回字段与页面显示字段保持一致，参见 architecture.md 的「接口按会被直接调用来设计」一节。

## 小程序

- 小程序代码包会下发到用户设备，原则与网站相同：不放密钥，核心逻辑放在云函数或自有后端，接口在服务端鉴权。
- 例如微信小程序的 AppSecret 只能保存在服务端，换取登录态的 `code2Session` 调用也在服务端完成。
