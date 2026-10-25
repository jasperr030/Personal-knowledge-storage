# 家里与公司 Codex 共用交接说明

更新日期：2026-10-26。本文件随私人知识仓库版本控制，不依赖聊天记录。

## 给另一端 Codex 的起始提示

> 请阅读本仓库 AGENTS.md、CONTEXT.md 和 docs/CODEX-HANDOFF.md。这个知识库同时收录生活、学习、兴趣、技术和个人工作经验。先检查 Git 状态和现有 MCP 连接，再继续任务。重要的新结论请整理到对应 Markdown，明确报告哪些内容已推送、哪些已在家里索引。

在其他工程中工作时，将本仓库的实际路径提供给 Codex，并要求显式读取这些文件。
`AGENTS.md` 是项目约定文件，不等于全账号或所有聊天的自动共享记忆。
参考：[OpenAI Docs：AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md)。

## 架构与固定地址

Markdown/Git → Python 解析与 CPU Embedding → Qdrant + SQLite FTS5 → 只读 MCP → 两端 Codex。
内容领域不限于技术；现有文本索引可以处理生活知识，不需要为每类知识创建独立服务。

| 项目 | 值 |
| --- | --- |
| 公司 MCP URL | `https://laptop-du55soua.tail2da209.ts.net/mcp` |
| 公司接入端口 | **443**，HTTPS 默认端口 |
| 健康检查 | `https://laptop-du55soua.tail2da209.ts.net/health` |
| 家里本机 | `http://127.0.0.1:8000/mcp` |
| 网络 | 两端 Tailscale，私人网络；未启用 Funnel |
| 工具 | `search_knowledge`、`read_knowledge` |
| Git | `git@github.com:jasperr030/Personal-knowledge-storage.git` |
| HTTPS Git | `https://github.com/jasperr030/Personal-knowledge-storage.git` |
| 家里笔记 | `F:\PersonalAIKB\outputs\personal-notes` |
| 家里服务工程 | `F:\PersonalAIKB\outputs\personal-ai-kb` |
| Docker 数据盘 | `F:\PersonalAIKB\docker-disk` |

旧的 `C:\Users\jaspe\Documents\Codex\2026-10-14\https-chatgpt-com-share-6acf8252-f970\outputs` 路径通过目录联接指向 F 盘，原脚本和快捷方式继续有效。不要分别移动联接及其目标。

## 公司端如何获取最新背景

首次没有笔记仓库时克隆上述 Git 仓库；已有仓库时在仓库目录执行：

```bash
git status --short
git pull --ff-only
```

有本地未提交内容时先保留或提交自己的修改，不要用 reset/强制覆盖。遇到分支分叉应检查差异。
拉取后，在该目录重新启动 Codex，或明确要求当前 Codex重新读取本次更新的说明文件。
公司配置完成后可执行：

```bash
python3 ~/.config/personal-ai-kb/check.py
```

客户端凭据保存在 `~/.config/personal-ai-kb/token`；不要展示或提交。
首次安装说明仍在家里工程的 `company-client/HANDOFF.md` 及私密接入包中。
本仓库不包含接入 Token，复制说明文件并不能代替首次安装客户端。

## 写笔记与更新检索

1. 任一端把新知识整理为 Markdown；用户的长期偏好与架构变动同步到 `CONTEXT.md`。
2. 检查 diff，提交需要分享的文件并 push。
3. 另一端工作前 pull，获取相同版本的说明和笔记。
4. 公司 push 后，家里的笔记 checkout 还需要手动拉取，随后运行中的服务最多五分钟内增量索引。

家里在服务工程目录运行：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Sync-Knowledge.ps1 -Repository ..\personal-notes
```

**目前没有定时 git pull。** MCP 搜索读取的是最近成功索引的快照，不是 GitHub 实时内容。
MCP 没有写入工具，不能靠一次聊天或搜索自动保存笔记。
两端可以共用文件和索引，但聊天记录、未写入文件的临时讨论不会自动互通。

## 家里启停

在 `F:\PersonalAIKB\outputs\personal-ai-kb` 运行：

```powershell
# 启动知识库
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Start-Knowledge.ps1
# 游戏前停止知识库并退出 Docker
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Start-Gaming.ps1
```

桌面“宽带自动重拨开关”单独控制检查：出门开启，在家关闭；五分钟一次，无终端弹窗。
上述服务启停脚本不等于自动切换宽带检查。当前聊天中用户说“打开/关闭”时，两项一起处理。
家里需开机、联网且用户已登录；游戏期间远程检索暂停。

## 排查

- 无法解析域名或超时：检查两端 `tailscale status`、家庭拨号网络。
- 502：检查家里 Docker、知识服务与 Tailscale Serve。
- 401：检查私有凭据是否匹配，不把 Token 贴进聊天或仓库。
- 有连接但新笔记搜不到：确认家里已 pull，查看 `/health` 的索引状态并等下一轮扫描。
- 需要重启 Windows Tailscale 服务时，管理员 PowerShell 可运行 `Restart-Service Tailscale`；普通进程可能没有权限。

动态在线状态以实际探测为准。本文件不承诺电脑始终在线或宽带断线必然恢复。
