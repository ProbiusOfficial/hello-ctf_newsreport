# axios投毒下openclaw更新行为的风险分析报告

## 1. 报告信息

- 报告名称：axios投毒下openclaw更新行为的风险分析报告
- 报告日期：2026-03-31
- 分析对象：`openclaw` 官方安装与更新路径
- 分析目标：判断在 `axios@1.14.1` 被下架前，安装或升级 OpenClaw 的行为是否会拉取到被投毒版本

## 2. 执行摘要

本次分析结论如下：

1. OpenClaw 官方推荐的全局安装与升级路径，明确使用 `openclaw@latest`。
2. OpenClaw 并未在自身逻辑中显式写死 `axios@latest`，但其传递依赖链会在安装时按 semver 规则解析 `axios` 的兼容最新版本。
3. 依赖链为：`openclaw` -> `@line/bot-sdk@^10.6.0` -> `axios@^1.7.4`。
4. `axios@^1.7.4` 覆盖 `1.14.1`，因此在恶意版本仍存在于 npm registry 的时间窗内，官方全局安装或升级 OpenClaw 的行为，高概率会解析并安装 `axios@1.14.1`。
5. 结合公开事件时间线，风险时间窗为北京时间 `2026-03-31 08:21` 至约 `2026-03-31 11:15`。
6. [openclaw issue #58140](https://github.com/openclaw/openclaw/issues/58140) 报告的安装异常现象，与本次 axios 投毒事件的行为特征高度吻合。

结论性判断：

在北京时间 `2026-03-31 08:21` 至约 `11:15` 之间，通过 OpenClaw 官方文档推荐方式执行安装或升级，存在较高概率拉取到被投毒的 `axios@1.14.1`。

## 3. 分析范围与问题定义

本报告围绕以下问题展开：

1. OpenClaw 的升级逻辑是否会拉取当时 registry 中的最新兼容依赖。
2. OpenClaw 对 `axios` 的依赖是否为直接依赖，还是通过 `@line/bot-sdk` 间接引入。
3. 在恶意 `axios@1.14.1` 上架但尚未下架的时间窗内，安装或升级 OpenClaw 是否可能命中恶意版本。
4. `issue #58140` 是否与本次事件存在时间和行为上的相关性。

## 4. 关键事实与证据

### 4.1 OpenClaw 官方更新行为

OpenClaw 官方更新文档显示：

- 官网安装器在全局安装场景下使用 `npm install -g openclaw@latest`
- 文档明确给出全局更新命令 `npm i -g openclaw@latest`
- 文档同时给出 `pnpm add -g openclaw@latest`
- `openclaw update` 在 npm 或 pnpm 安装场景下，会尝试通过包管理器执行更新

这说明其官方推荐路径并不是安装某个固定版本，而是始终以 `openclaw@latest` 为入口。

来源：

- [OpenClaw 更新文档](https://docs.openclaw.ai/zh-CN/install/updating)

### 4.2 OpenClaw 与 axios 的依赖链

公开包元数据显示，OpenClaw 相关发布版本依赖 `@line/bot-sdk ^10.6.0`。

而 `@line/bot-sdk@10.6.0` 的已发布 `package.json` 中声明：

```json
{
  "optionalDependencies": {
    "axios": "^1.7.4"
  }
}
```

这意味着：

- `axios` 并非 OpenClaw 顶层直接锁定依赖
- 它是由 `@line/bot-sdk` 通过 `optionalDependencies` 引入
- 只要安装时未显式 `--omit=optional`，npm 默认会尝试安装该依赖

来源：

- [openclaw 包元数据](https://npmx.dev/package/openclaw)
- [@line/bot-sdk@10.6.0 package.json](https://app.unpkg.com/%40line/bot-sdk%4010.6.0/files/package.json)
- [npm package.json 文档](https://docs.npmjs.com/cli/v11/configuring-npm/package-json/)

### 4.3 npm 对兼容版本的解析行为

npm 对依赖范围按 semver 规则解析。`^1.7.4` 会接受高于等于 `1.7.4` 且低于 `2.0.0` 的版本，因此 `1.14.1` 在该范围内。

对 fresh install 或全局安装场景而言，若未使用可发布的锁定机制来固定传递依赖，npm 会在安装时解析符合范围的最新版本。普通 `package-lock.json` 主要用于应用本地安装，不等同于发布到 registry 后对消费者安装过程的强制传递锁定；若要将依赖解析结果随包发布并影响消费者，通常需要 `npm-shrinkwrap.json`。

基于公开材料，未发现 OpenClaw 官方更新文档中存在任何 `--omit=optional`、依赖冻结、或传递依赖锁定保护措施。

来源：

- [npm package-locks 文档](https://docs.npmjs.com/cli/v6/configuring-npm/package-locks/)
- [npm shrinkwrap 文档](https://docs.npmjs.com/cli/v6/configuring-npm/shrinkwrap-json/)
- [npm package.json 文档](https://docs.npmjs.com/cli/v11/configuring-npm/package-json/)

### 4.4 axios 投毒事件时间线

根据 StepSecurity 对事件的公开分析：

- `axios@1.14.1` 于 `2026-03-31 00:21 UTC` 发布
- 约于 `2026-03-31 03:15 UTC` 被下架
- 下架后，`latest` 回退至 `1.14.0`

换算为北京时间：

- 发布时点：`2026-03-31 08:21`
- 下架时点：约 `2026-03-31 11:15`

同时，StepSecurity 指出恶意版本通过新增依赖及 `postinstall` 机制落地恶意载荷。

来源：

- [StepSecurity: axios compromised on npm](https://www.stepsecurity.io/blog/axios-compromised-on-npm-malicious-versions-drop-remote-access-trojan)

## 5. 技术分析

### 5.1 OpenClaw 是否“拉 latest”

需要区分两个层次：

1. OpenClaw 顶层安装入口是否拉 `latest`
2. 传递依赖 `axios` 是否会在安装时解析到“当时最新兼容版本”

对第一个问题，结论明确：是。官方文档使用的是 `openclaw@latest`。

对第二个问题，严格来说不是“代码里直接写 `axios@latest`”，但实际效果接近“拉当时兼容范围内的最新版本”。原因是：

- OpenClaw 依赖 `@line/bot-sdk`
- `@line/bot-sdk@10.6.0` 声明 `axios@^1.7.4`
- `^1.7.4` 会覆盖 `1.14.1`
- 在全局安装或升级场景中，包管理器会根据当前 registry 状态解析该范围

因此，从风险视角看，OpenClaw 的官方全局更新路径对 `axios` 呈现出“安装时跟随 registry 当前兼容最新版本”的行为特征。

### 5.2 风险是否来自仓库 lockfile

仓库中的 `pnpm-lock.yaml` 只对仓库源码安装和开发环境有较强参考意义。它并不能自动代表通过 `npm i -g openclaw@latest` 安装 npm 已发布包时，消费者环境中的传递依赖一定被锁死。

也就是说：

- 对源码仓库执行受 lockfile 约束的安装，风险相对较低
- 对官方文档推荐的全局安装与升级路径，风险显著更高

这也是为什么本报告重点判断官方安装器和全局更新行为，而不是仅凭仓库 lockfile 下结论。

### 5.3 optionalDependencies 是否会默认安装

会，除非显式省略。

`@line/bot-sdk` 中的 `axios` 不是普通 `dependencies`，而是 `optionalDependencies`。但根据 npm 官方文档，optional 依赖默认仍会安装，只有在使用 `--omit=optional` 等配置时才会跳过。

由于 OpenClaw 文档中未体现此类保护参数，因此应按“默认会安装”进行风险判断。

## 6. 风险窗口判断

### 6.1 时间窗口

按照公开时间线，本次风险窗口为：

- 开始：北京时间 `2026-03-31 08:21`
- 结束：约北京时间 `2026-03-31 11:15`

### 6.2 结论表

| 时间段（北京时间） | OpenClaw 官方安装/升级是否可能命中 `axios@1.14.1` | 判断 |
|---|---|---|
| `2026-03-31 08:21` 之前 | 否 | 恶意版本尚未发布 |
| `2026-03-31 08:21` 至约 `11:15` | 是，高概率 | semver 范围会覆盖 `1.14.1` |
| 约 `2026-03-31 11:15` 之后 | 常规情况下否 | 恶意版本已下架，解析结果回退 |

## 7. 分场景风险评估

### 7.1 高风险场景

- 通过官网安装器执行首次安装
- 使用 `npm i -g openclaw@latest`
- 使用 `pnpm add -g openclaw@latest`
- 执行 `openclaw update`，且当前安装方式为 npm 或 pnpm 全局安装

这些场景共同特点是：

- 以 `openclaw@latest` 为入口
- 发生新的依赖解析
- 没有看到对 `optionalDependencies` 的显式抑制

### 7.2 相对低风险场景

- 基于源码仓库、且严格遵循现有 lockfile 的安装
- 已在风险窗口之前完成安装，之后未重新解析依赖

但应注意：

- 如果源码路径执行了重新解析依赖的安装行为，仍不能绝对排除风险
- 若 lockfile 被刷新、删除，或安装命令未严格冻结，也可能重新命中恶意版本

## 8. 与 issue #58140 的相关性

[openclaw issue #58140](https://github.com/openclaw/openclaw/issues/58140) 报告称，用户运行官方 PowerShell 安装命令后，Windows Defender 识别到恶意批处理文件和启动项相关风险。

该 issue 与本次事件具有以下一致性：

- 时间发生在 `2026-03-31`
- 场景是官方安装命令
- 症状与恶意安装后持久化行为相符

因此，该 issue 可以视为本次风险链路的强支持性案例。

需要说明的是：

- 该 issue 能够支持“事件已经在 OpenClaw 安装场景中实际暴露”
- 但其本身并不能替代依赖解析链和时间窗分析
- 根因判断仍应以更新文档、依赖声明、npm 解析规则和 axios 事件时间线为主

## 9. 风险结论

综合以上证据，可以得出如下结论：

1. OpenClaw 官方升级逻辑并非显式固定安全版本，而是基于 `openclaw@latest` 执行安装或升级。
2. OpenClaw 对 `axios` 的引入来自 `@line/bot-sdk@10.6.0` 的 `optionalDependencies`，其版本范围为 `^1.7.4`。
3. 在 npm 默认行为下，该范围会在安装时解析到兼容范围内的最新版本。
4. `axios@1.14.1` 处于兼容范围内，且在北京时间 `2026-03-31 08:21` 至约 `11:15` 间曾短暂存在于 registry。
5. 因此，在该时间窗内使用 OpenClaw 官方推荐方式执行安装或升级，存在较高概率拉取到被投毒的 `axios@1.14.1`。

最终结论：

OpenClaw 的升级逻辑不是“直接拉 `axios@latest`”，但其实际效果在本次事件中等价于“在安装时解析到了 registry 中当时最新的兼容 axios 版本”，因此在恶意版本上线窗口内具有明确的供应链暴露风险。

## 10. 应急建议

建议立即对以下对象进行排查：

- 北京时间 `2026-03-31 08:21` 至约 `11:15` 期间安装或升级过 OpenClaw 的主机
- 同时间段内执行过自动化构建、CI、打包、部署的节点
- 通过官网安装器、PowerShell 安装命令、`npm i -g openclaw@latest`、`pnpm add -g openclaw@latest`、`openclaw update` 执行更新的设备

建议优先执行以下措施：

1. 检查是否安装过 `axios@1.14.1`、`axios@0.30.4`、`plain-crypto-js@4.2.1`
2. 对命中机器按“恶意 `postinstall` 可能已执行”处理
3. 检查启动项、计划任务、异常批处理文件和可疑持久化痕迹
4. 轮换可能暴露的凭据、令牌和会话
5. 对受影响机器开展主机取证与恶意文件清理

## 11. 分析边界与说明

本报告基于公开资料进行分析，结论针对“官方推荐安装与升级路径”的风险行为。

以下内容属于合理推断而非直接源码证明：

- `pnpm add -g openclaw@latest` 在风险窗口内的解析结果与 npm 全局安装保持同类风险特征
- 官方发布包未采用足以冻结该传递依赖解析结果的发布锁定机制

这些推断建立在官方文档命令、npm 解析规则、已发布依赖元数据和公开事件时间线的一致性基础上，具有较高可信度。

## 12. 参考资料

1. [OpenClaw 更新文档](https://docs.openclaw.ai/zh-CN/install/updating)
2. [OpenClaw 仓库](https://github.com/openclaw/openclaw)
3. [OpenClaw issue #58140](https://github.com/openclaw/openclaw/issues/58140)
4. [openclaw 包元数据](https://npmx.dev/package/openclaw)
5. [@line/bot-sdk@10.6.0 package.json](https://app.unpkg.com/%40line/bot-sdk%4010.6.0/files/package.json)
6. [npm package.json 文档](https://docs.npmjs.com/cli/v11/configuring-npm/package-json/)
7. [npm package-locks 文档](https://docs.npmjs.com/cli/v6/configuring-npm/package-locks/)
8. [npm shrinkwrap 文档](https://docs.npmjs.com/cli/v6/configuring-npm/shrinkwrap-json/)
9. [StepSecurity: axios compromised on npm](https://www.stepsecurity.io/blog/axios-compromised-on-npm-malicious-versions-drop-remote-access-trojan)
