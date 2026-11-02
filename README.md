# dsh-super-agent

`dsh-super-agent` 是运行在 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（DSH）上的常驻 Agent：它监听 GitHub（可选 Feishu），把被允许的请求变成工作项并在自己的会话和目录里完成，最后在原对话中给出结果，例如一个 Pull Request；Web 面板会显示哪些事情需要人来处理。

本仓库目前只包含这份 README；可运行的源码是其上游研究原型 [dsh-superagent-prototype](https://github.com/deepseek-ai/dsh-superagent-prototype)，下面的步骤按该项目编写。

## 快速开始

在本地跑起一个完整的 dsh-super-agent 需要四样东西：DSH CLI、项目源码、一个独立的 DSH profile（本指南中名为 `superagent`）以及一份机器配置。

### 1. 环境要求

| 依赖 | 版本 | 用途 |
| --- | --- | --- |
| Node.js | 22.19+ 或 24+ | 运行 DSH 与项目 |
| pnpm | 11.7（见项目 `packageManager`） | 安装依赖与构建 |
| DeepSeek Harness CLI | 0.2.1-rc.3，须满足 `engines.dsh` 的 `^0.2.1-alpha.2` | 启动 profile |
| git | 2.32+ | 获取源码 |
| Python 3.8+、tmux | — | `scripts/dsh-superagent` 管理脚本 |
| GitHub CLI（`gh`） | 2.48+ | 可选，让 Agent 以独立账号操作 GitHub |

### 2. 安装 DSH CLI

```sh
npm install -g @deepseek-ai/dsh@0.2.1-rc.3
dsh --version   # 0.2.1-rc.3
```

### 3. 获取源码并构建

```sh
git clone https://github.com/deepseek-ai/dsh-superagent-prototype.git dsh-superagent
cd dsh-superagent
pnpm install
pnpm build
```

### 4. 创建独立 profile 并安装 bundle

```sh
dsh superagent --from-default-profile web --dump-config > /dev/null
dsh plugin --profile superagent add "link:$PWD"
```

第一条命令从 DSH 的 Web 模板创建 `~/.dsh/profiles/superagent`（`DSH_HOME` 可改变 `~/.dsh` 的位置）；profile 已存在时不要执行它，它会替换 profile 并删除已有的 `cordis.patch.yml`。第二条命令把当前目录以 link 方式装进该 profile。

### 5. 写入机器配置

```sh
cp examples/profile.cordis.patch.yml ~/.dsh/profiles/superagent/cordis.patch.yml
```

编辑该文件并替换其中所有 `<placeholder>`：`stateDir` 必须是绝对路径，数据库、锁、命令 shim 与工作目录都放在这里（升级前先备份）；`webserver.port` 默认是 `3082`。需要 Feishu 时按注释启用 `dsh-superagent-feishu`。

### 6. 启动、检查与停止

```sh
scripts/dsh-superagent start
scripts/dsh-superagent status   # 0 运行正常；1 未运行或无响应；2 某部分报错；3 等待配置
scripts/dsh-superagent stop
```

`start` 会在 tmux 会话 `dsh-superagent` 中运行 `dsh --profile superagent`。

### 7. 打开面板

浏览器访问 <http://127.0.0.1:3082>，在侧边栏中选择 **Superagent**。

完整的账号、授权名单与 Feishu 配置见上游项目的 [SETUP.md](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/main/SETUP.md) 与 `docs/006-deployment.md`。

## 开发

```sh
pnpm check   # 类型检查 + 测试 + 构建
```

[![](https://img.shields.io/badge/powered_by-dsh-4D6BFE?style=flat-square&logo=deepseek&logoColor=white)](https://github.com/deepseek-ai/deepseek-harness)
