# PHASE-00-BASELINE（中文）

## 目标
建立前端、自动化、路由与场景资产之间的可追溯基线绑定。

## 输入
- 前端仓库路径与默认分支
- Java BDD Playwright 仓库路径与测试命令
- 初始业务路由与目标场景

## 输出
- 按项目实际填写的 `config/project_binding.example.yaml`
- 初始 `route_map` 与 `locator_inventory` 产物
- 一次基线执行证据包

## 准出标准
- 任一 locator 都可追溯到 scenario、step、page object、route、前端文件。
- 至少一个冒烟场景可运行并产出证据。

## 非本阶段范围
- 自动自愈与补丁生成。
