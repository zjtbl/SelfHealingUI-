# PHASE-03-FRONTEND-AST-GRAPH（中文）

## 目标
建立前端静态结构理解，并映射到测试资产。

## 输入
- React/AntD 源码
- 路由配置
- 已有 route-element-scenario 映射

## 输出
- 组件图谱快照
- route->component->element 绑定
- locator 脆弱性早期信号

## 准出标准
- 变更文件可映射到受影响路由与元素。
- 能识别测试契约缺口（`data-testid`、role/label 质量）。

## 非本阶段范围
- 业务逻辑语义正确性的全自动判断。
