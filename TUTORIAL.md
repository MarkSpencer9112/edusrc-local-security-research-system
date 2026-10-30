# EduSRC 本地安全研究系统使用教程

本教程介绍如何先测试本地合成靶场，再启动明确授权的本地源码审计。系统默认离线，不执行远程主动测试。

## 1. 环境要求

- macOS、Linux 或 Windows 上可用的 Python 3.9 或更新版本。
- 不需要安装第三方 Python 软件包。
- 以下命令均从仓库根目录运行。

在终端进入项目目录：

```bash
cd "/你的本地路径/EduSRC-Local-Research-System"
```

## 2. 初始化并测试合成靶场

初始化脚本只创建缺失的运行目录和状态文件，不覆盖已有文件：

```bash
python3 EduSRC-Research/scripts/initialize.py
```

运行隔离靶场测试：

```bash
python3 EduSRC-Research/lab/run_lab.py
```

靶场使用仓库自带的合成样例，在临时副本中运行任务控制器和源码审计器；不启动 Web 服务、不访问网络，也不更改正式授权文件。结果写入 `EduSRC-Research/results/verification/local_lab.json`。预设样例命中只是检测器测试数据，不是实际漏洞。

## 3. 配置正式本地审计授权

没有正式授权时请停在靶场测试，不要对其他项目启动审计。需要正式审计时，先核实书面授权，再编辑：

```text
EduSRC-Research/scope/authorization.json
```

填写资产所有者、授权证明编号、带时区的有效起止时间、精确本地项目目录、`local_source_audit` 活动、禁止事项和测试限制。示意格式如下；尖括号内容必须替换为真实且核实过的信息：

```json
{
  "asset_owner": "<资产所有者>",
  "proof_id": "<授权证明编号>",
  "valid_from_utc": "<例如 2026-10-30T00:00:00Z>",
  "valid_until_utc": "<例如 2026-11-30T23:59:59Z>",
  "assets": ["<精确的本地项目绝对路径>"],
  "allowed_activities": ["local_source_audit"],
  "prohibited_activities": ["remote_active_testing", "destructive_testing", "real_user_data"],
  "test_limits": ["仅对该本地目录做只读源码检查"]
}
```

时间必须使用有效的 ISO 8601 时间并带时区。`assets` 必须包含将传给 `--source` 的精确本地目录，不得使用通配符或写入推测出的关联资产。程序实际强制校验的是 `authorization.json`；`scope/allowed_assets.txt` 只是人工查看用的补充清单。授权不完整、路径不匹配或已过期时，启动会被拒绝。

不要在授权文件里放密码、API Key、Cookie 或访问令牌。

## 4. 启动本地源码审计

只有在授权记录已填写并复核后，才运行：

```bash
python3 EduSRC-Research/scripts/task_controller.py start --source "/精确的/本地/项目路径"
```

任务控制器会检查状态和授权，建立任务编号并运行本地只读检查。扫描不会修改原始源码。可在另一终端查看状态：

```bash
python3 EduSRC-Research/scripts/task_controller.py status
```

停止或请求暂停：

```bash
python3 EduSRC-Research/scripts/task_controller.py stop
python3 EduSRC-Research/scripts/task_controller.py pause
```

恢复被停止或暂停的任务：

```bash
python3 EduSRC-Research/scripts/task_controller.py resume
```

恢复时会再次检查授权和源码检查点。停止与暂停是协作式机制；不能保证强行中断卡在系统调用中的操作。当前检查阶段中断后可能从该阶段重新开始，不支持逐文件续跑。

## 5. 查看发现并生成报告

审计发现位于 `EduSRC-Research/results/findings/`。生成 Markdown 草稿：

```bash
python3 EduSRC-Research/scripts/report_generator.py
```

草稿保存在 `EduSRC-Research/reports/drafts/security_audit.md`。模式命中默认为 `UNVERIFIED`（未验证），需要人工复核；不得仅根据静态模式命中声称漏洞已确认。分享报告前请再次检查敏感信息。

## 6. 当前功能边界

- 当前源码检查器只用简单模式匹配筛查 SQL 查询构造、动态 HTML 输出、不安全传输设置、疑似凭据字面量和调试开关。
- 项目清点器能识别常见依赖清单文件名，但不解析依赖版本或匹配漏洞数据库。
- 配置文件只作为普通文本接受上述模式检查，尚无完整配置审计规则引擎。
- `agents/` 下的文件是角色说明，不是已接入的自动智能体。
- 尚未实现远程测试、动态验证、联网情报更新、自动报告提交或逐文件检查点恢复。

请勿访问真实用户数据、执行破坏性测试、扩大授权范围或尝试规避监控。完整验收状态见 `EduSRC-Research/reports/INITIALIZATION_ACCEPTANCE.md`。
