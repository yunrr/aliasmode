# AGENTS.md

本文件面向在本仓库工作的 AI Agent，记录**本 fork 特有的工作约定**。
上游项目自身的贡献规范见 [CONTRIBUTING.md](CONTRIBUTING.md)；两者冲突时以本文件为准（本地 fork 规则优先）。

## 本仓库的身份

- 本仓库是 `yunrr/aliasmode`，由 `aliasmode/aliasmode` fork 而来。
- `origin` = 自己的 fork（可写）。
- `upstream` = 上游原项目（**只读**，仅供拉取）。
- 工作分支：`main`。

```sh
git remote -v
# origin    https://github.com/yunrr/aliasmode.git
# upstream  https://github.com/aliasmode/aliasmode.git
```

## 铁律：只改本地 fork，不打扰上游

**所有修改只留在 fork 仓库，绝不向上游写入任何东西。**

明确禁止：

- `git push upstream`，或以任何方式向 `aliasmode/aliasmode` 推送分支、tag。
- 向上游仓库提交 Pull Request。
- 向上游提 Issue（含 bug 报告、功能建议）。
- `gh pr create --repo aliasmode/aliasmode`、`gh issue create --repo aliasmode/aliasmode` 等指向上游的写操作。
- 通过 API、网页或任何间接方式修改上游仓库。

允许：

- `git fetch upstream` 拉取上游最新代码。
- 在本地 `main` 上 merge / rebase 上游提交。
- 把合并或修改结果推送到 `origin`（自己的 fork）。

同步上游的推荐做法：

```sh
git fetch upstream
git merge upstream/main      # 或 rebase，按当次情况选择
git push origin main
```

## 本机环境注意事项

### 上游误提交的 `node_modules` 符号链接

上游在提交 `b7715a6` 中误将一个 Linux 符号链接当作普通文件提交，路径为 `node_modules`，内容只有一行：

```
/home/coder/projects/aliasmode/.claude/worktrees/fix-google-dbsc-logout/node_modules
```

它仅 84 字节，但会**在 Windows 上霸占 `node_modules` 这个名字**，导致 `bun install` 无法创建依赖目录。`.gitignore` 写的是 `node_modules/`（带斜杠只匹配目录），拦不住这个同名普通文件。

本 fork 已删除该文件。若重新克隆、或同步上游时让它复活，按以下步骤处理：

```sh
rm node_modules          # 走回收站，可恢复；不要用 rm -rf 或其它永久删除
bun install
```

删除后不要向上游提 PR 修正——那是上游的事，本仓库只管本地可用。

### `bun.lock` 会被本机镜像地址污染

本机 `~/.npmrc` 配置了 `registry=https://registry.npmmirror.com`。`bun install` 会把 lockfile 中每个包的解析地址从空字符串改写成 npmmirror 完整 URL，并新增 `configVersion` 字段，产生约 188 行的伪改动。

**这不是真实依赖变更，不要提交。** 每次 `bun install` 后检查并还原：

```sh
git status --porcelain
git checkout -- bun.lock
```

只有真实升级依赖时才让 lockfile 变更进入提交，并在提交信息里说明原因。

### 安装脚本被禁用

`bunfig.toml` 中有 `[install] ignoreScripts = true`，`bun install` 不执行包的 postinstall 脚本。若启动或构建时提示缺原生模块、缺下载的二进制，先怀疑这里，针对性放行所需包，不要全局关掉。

### 已验证的工具链

| 工具 | 版本 | 说明 |
| --- | --- | --- |
| Bun | 1.4.2 | CI 用 1.2.21，本地以本机为准 |
| TypeScript | 5.9.3 | 由 `bun install` 装入 |

## 常用命令

```sh
bun install            # 安装依赖（之后务必还原 bun.lock）
bun run typecheck      # tsc --noEmit
bun test               # bun test
bun run start          # 启动
bun run serve          # 启动服务
```

Windows 上用 PowerShell 执行时，先 `cd` 进仓库根目录。

## 提交约定

- 提交信息用英文，遵循上游风格（见 `git log`）。
- 只提交源码与配置。`node_modules/`、`dist/`、`target/`、profile 导出、`.env`、密钥、诊断产物一律不入库（见 `.gitignore`）。
- 不提交本机环境特有内容：绝对路径、用户名、镜像地址、token。
- 不与本次改动无关的重构混在一起。
