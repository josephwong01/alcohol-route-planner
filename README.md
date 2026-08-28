# Alcohol Route Planner Skill — Spec v0.1.0

> 这是一个交给 Codex 继续实现和测试的中间规格包，不是最终可执行 Skill。

## 目标

根据用户提供的城市、区域、目标活动片区、人数、距离/耗时和便利店数量偏好，
规划 3 条候选步行路线，并选择 1 条推荐路线，最终输出一张适合手机查看的 9:16 路线图。

核心体验参考“沿路线步行 + 便利店作为连续 checkpoint”的玩法，但 Skill 本身只负责路线规划、
便利店 POI 组织、距离/耗时估算和地图视觉化，不应把“每一家必须饮酒”作为强制规则。

## MVP 输入

1. 城市
2. 行政区 / 所在区域
3. 期望活动的大概区域 / 夜生活片区
4. 人数范围
5. 总距离范围，或预计耗时范围
6. 期望便利店数量 / 密集程度

## MVP 输出

- 3 条候选路线
- 每条路线：
  - 起点 / 终点
  - 总距离
  - 预计步行耗时
  - 便利店数量
  - 便利店顺序清单
  - 每两个 checkpoint 之间的距离
  - 主要地标
  - 路线特色
  - 难度 / 密度评分
- 1 条推荐路线
- 1 张 9:16 手机竖版路线图
- 可选：GeoJSON / Google Maps / 高德 / 百度地图可导入的坐标点列表

## 目录

- `docs/PRODUCT_SPEC.md`：产品与功能规格
- `docs/ROUTE_ALGORITHM.md`：路线生成与评分逻辑
- `schemas/input.schema.json`：输入 Schema
- `schemas/output.schema.json`：输出 Schema
- `prompts/route_planner_prompt.md`：给 Agent/LLM 的核心规划 Prompt
- `examples/shenzhen_futian.example.json`：深圳福田示例
- `examples/shenzhen_futian.expected.md`：示例期望输出结构

## 推荐实现路径

1. Codex 建 Skill 工程骨架。
2. 接入地图/POI 数据源。
3. 将便利店 POI 过滤、排序与路径计算实现为确定性代码。
4. LLM 只负责理解用户意图、解释路线和生成视觉图规范。
5. 对深圳福田案例做回归测试。
6. 再测试 3–5 个城市，验证泛化能力。
