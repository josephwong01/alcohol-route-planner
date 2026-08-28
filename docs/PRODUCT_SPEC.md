# Product Spec — Alcohol Route Planner Skill v0.1

## 1. 产品目标

用户只需给出一个城市和大概活动区域，即可自动得到适合线下步行聚会的便利店 checkpoint 路线。

典型场景：

- “我在深圳福田，想在 COCO Park / 会展中心附近玩。”
- “4–6 人。”
- “希望走 3–5 km。”
- “大概经过 10–15 家便利店。”

Skill 应将模糊需求转化为：
**地理边界 → POI 搜索 → 候选路径 → 便利店筛选 → 路线评分 → 推荐 → 9:16 路线图。**

---

## 2. 必填输入

| 字段 | 含义 | 示例 |
|---|---|---|
| city | 城市 | 深圳 |
| district | 区域/行政区 | 福田区 |
| preferred_area | 期望活动片区 | COCO Park、领展中心城、会展中心 |
| party_size | 人数范围 | 4–6 |
| distance_or_duration | 总距离或耗时 | 3–5 km / 60–90 min |
| convenience_store_target | 便利店数量或密度 | 10–15 家 |

---

## 3. 可选输入

- start_hint：希望从哪里出发
- end_hint：希望在哪里结束
- loop_preference：环线 / 单向线 / 不限
- nightlife_priority：是否优先夜生活片区
- scenery_priority：是否优先城市夜景
- store_brands：偏好/排除的便利店品牌
- avoid_areas：希望避开的区域
- route_count：候选路线数量，默认 3
- map_aspect_ratio：默认 9:16
- language：默认 zh-CN

---

## 4. 输出结构

### 4.1 候选路线

每条路线至少包含：

- route_id
- route_name
- start
- end
- total_distance_km
- estimated_walk_minutes
- checkpoint_count
- checkpoints[]
- landmarks[]
- route_character
- density_score
- navigation_score
- nightlife_score
- overall_score

### 4.2 推荐结果

系统需明确说明：

- 推荐哪条
- 为什么推荐
- 与另外两条相比的优势
- 哪些条件下应改选另外两条

### 4.3 视觉图

默认生成 9:16。

图中必须有：

- 标题
- 起点 / 终点
- 北向或方向提示
- 实际道路 / 关键道路名
- 关键地标名
- 编号 checkpoint
- 路线连线
- 总距离 / 耗时 / 店数
- checkpoint 清单
- 图例
- “实际营业状态和步行导航以地图 App 为准”的提示

---

## 5. 路线规划原则

### P0：可走
路线必须基于真实可步行道路，而不是简单直线连接。

### P1：距离命中
优先满足用户给定距离/耗时范围。

### P2：便利店数量命中
目标数量允许默认 ±20% 浮动。

### P3：密度平衡
避免连续 50–100 米内出现过多 checkpoint。
默认希望平均 checkpoint 间距约 150–400 米。

### P4：区域体验
路线应尽量穿过用户指定的夜生活、商业、夜景或地标区域。

### P5：导航友好
减少不必要折返、复杂地下通道、商场内部穿越和难辨方向的小路。

### P6：终点价值
若用户指定夜生活区域，终点优先落在餐饮、酒吧、交通方便的地点附近。

---

## 6. 安全约束

Skill 的产品能力是“路线和 checkpoint 规划”，而不是强制饮酒计划。

- 不建议将每个 checkpoint 与固定酒量绑定。
- 输出中应保留“可跳过 / 可选无酒精饮品 / 量力而行”的说明。
- 不为驾驶者设计饮酒路线。
- 明确建议使用步行、公共交通或代驾离场。
- 不以“最大饮酒量”作为路线优化目标。

---

## 7. MVP 验收标准

满足以下条件即可进入 Codex 实现阶段：

1. 输入 Schema 可验证。
2. 能返回 3 条不同风格路线。
3. 每条路线有真实 POI。
4. 便利店总数在目标值 ±20% 内。
5. 距离在目标范围附近。
6. checkpoint 顺序与路线一致。
7. 输出包含导航友好度和推荐理由。
8. 能生成一张清晰的 9:16 路线图规范。
