# electron-demo
第一个 PR 偏功能和测试，第二个 PR 偏工程稳定性和问题定位。我希望通过这两个方向一起展示我对业务能力、边界行为和开发链路的理解。


feat(search): 增加整词匹配支持
## 修改背景

我在阅读 `plugin-search` 和 Electron demo 搜索栏实现时，发现搜索面板 UI 已经有 “By word / 整词匹配” 入口，但底层辅助函数 `findSearchMatches` 和 `replaceAllMatches` 还没有真正支持整词匹配。

这会导致：
- 插件 UI 能表达整词匹配能力
- 但外部复用这些辅助函数时，行为并不完整

## 本次修改

我主要做了三件事：

1. 在 `SearchOptions` 中补充 `wholeWord` 选项
2. 统一搜索正则构造逻辑，让查找和替换行为保持一致
3. 补充整词匹配相关测试，并把 Electron demo 搜索栏接上该能力

## 效果

例如搜索 `cat`：

`cat scatter cat cat_ cat-cat`

开启整词匹配后：
- 会命中独立出现的 `cat`
- 会命中 `cat-cat` 中被标点分隔的 `cat`
- 不会误命中 `scatter` 中的 `cat`
- 不会误命中 `cat_` 这种标识符片段

## 验证

我做了以下验证：

- 运行 `packages/plugin-search/test/plugin-search.test.ts`
- 检查 `packages/plugin-search` 的 TypeScript 类型
- 检查 `apps/electron-demo` 的 TypeScript 类型

## 说明

这个改动不涉及破坏性 API 变更，属于对现有搜索能力的补全，也让插件层和 demo 层的行为更一致。




fix(electron): 提升 dev-electron 启动稳定性
## 修改背景

在本地启动 Electron demo 过程中，我发现开发态启动流程有两个明显的不稳定点：

1. 启动 Electron 前只固定等待 2 秒，依赖 Vite 启动速度
2. 如果终端环境中存在 `ELECTRON_RUN_AS_NODE`，Electron 会退化成 Node 进程，导致应用无法正常拉起

这会让开发启动过程依赖本机环境和时序，稳定性比较差。

## 本次修改

我对 `dev-electron.mjs` 做了以下调整：

1. 将固定等待改为显式等待 Vite dev server 可访问
2. 启动 Electron 时从应用根目录进入，减少入口调用方式带来的脆弱性
3. 在启动前清理 `ELECTRON_RUN_AS_NODE` 等冲突环境变量
4. 增加子进程错误输出，方便定位启动失败原因

## 效果

修改后，Electron demo 的开发态启动流程：
- 不再依赖固定等待时间
- 对本地 shell 环境污染更有容错能力
- 启动问题更容易定位

## 验证

我做了以下验证：

- 启动 renderer dev server
- 运行 `npm run --prefix apps/electron-demo dev:electron`
- 确认 Electron 主进程和 renderer 进程能够正常拉起

## 说明

这个改动只影响开发态启动流程，不涉及公开 API，也不改变编辑器运行时逻辑。

