# EduSRC 本地安全研究系统

面向明确授权资产的本地源码安全审查原型。默认离线，只读取指定源码；不执行远程主动测试。授权清单默认留空。

## 快速开始

需要 Python 3.9 或更新版本，无第三方依赖。先运行隔离靶场测试：

```bash
python3 EduSRC-Research/lab/run_lab.py
```

要审计真实本地项目，先依据正式授权填写 `EduSRC-Research/scope/authorization.json`，包括所有者、证明编号、有效期、精确本地路径和 `local_source_audit` 活动。之后运行：

```bash
python3 EduSRC-Research/scripts/task_controller.py start --source "/精确的/本地/项目路径"
```

授权缺失、过期、路径不匹配或任务类型不受支持时，程序会拒绝执行。完整说明见 [EduSRC-Research/README.md](EduSRC-Research/README.md)。

## 范围和限制

现有检查器以本地文本模式筛查少量源码和配置风险；依赖清单仅做文件识别，不进行漏洞版本匹配。扫描结果需要人工确认，不能把模式命中当作已验证漏洞。系统不读取真实用户数据、不破坏数据、不访问网络、不自动提交报告。

首次部署验收和未完成项见 [初始化验收报告](EduSRC-Research/reports/INITIALIZATION_ACCEPTANCE.md)。


