# Route Generation & Scoring Logic v0.1

## 1. 数据准备

### 1.1 Geocoding
将：
- city
- district
- preferred_area
- start_hint
- end_hint

转换为坐标和搜索边界。

### 1.2 POI Search
搜索目标区域内：
- convenience_store
- supermarket（可选）
- nightlife
- landmark
- metro_station

便利店至少保留：
- name
- brand
- lat
- lon
- address
- opening_hours（若数据源可得）
- poi_id

---

## 2. 候选起终点

优先顺序：

起点：
1. 用户 start_hint
2. preferred_area 边缘交通节点
3. 商圈 / 地铁站 / 地标

终点：
1. 用户 end_hint
2. nightlife cluster
3. 大型商圈
4. 地铁站附近

---

## 3. 路线形态

至少生成三种候选：

A. Scenic / CBD Loop
- 环境好
- 地标多
- 便利店中等
- 导航容易

B. Challenge / Dense
- 便利店较多
- 街区变化大
- 距离略长

C. Straight / Easy Navigation
- 沿主干道或简单折线
- 几乎无折返
- 最适合第一次组织

---

## 4. Checkpoint 选择

### 4.1 初筛
保留位于候选 walking corridor 一定缓冲范围内的便利店。

推荐 corridor buffer：
- 80–150 m

### 4.2 去重
若两个 POI：
- 名称高度相同
- 坐标 < 30m
则视为重复候选。

### 4.3 密度控制
默认 checkpoint 间距：
- minimum preferred gap = 120m
- ideal gap = 180–350m
- maximum preferred gap = 500m

若局部过密：
- 保留更靠近主路线的店
- 保留营业信息更可靠的店
- 优先品牌明确的店

### 4.4 数量命中
设目标店数为 N：
- hard acceptable range: 0.8N – 1.2N
- preferred range: 0.9N – 1.1N

---

## 5. 路线评分

总分 100：

- Distance Fit：25
- Store Count Fit：25
- Navigation Simplicity：15
- Nightlife / Destination Fit：15
- Landmark / Experience：10
- Store Spacing Balance：10

### 示例

score =
0.25 * distance_fit +
0.25 * store_fit +
0.15 * navigation +
0.15 * nightlife +
0.10 * experience +
0.10 * spacing

每项先归一化到 0–100。

---

## 6. 关键指标

### distance_fit

若目标区间为 [Dmin, Dmax]：
- 区间内 = 100
- 每偏离 10% 逐步扣分

### store_fit

target = N

store_fit = max(0, 100 - abs(actual-N)/N*100)

### spacing_balance

计算相邻 checkpoint 距离：
- median
- std
- min
- max

若大量间距 <100m，扣分；
若大量间距 >600m，扣分。

### navigation

根据：
- 转弯次数
- 折返距离
- 商场内部穿越
- 道路等级
- 地下通道复杂度

进行评分。

---

## 7. 输出图生成逻辑

地图数据层：
1. basemap
2. walking polyline
3. checkpoint markers
4. start / finish
5. landmarks
6. road labels

版式层：
- 9:16
- 顶部：路线名称 + 三项核心指标
- 左/中：主地图
- 右/下：checkpoint 清单
- 底部：特色、难度、注意事项

最终图必须能让用户在手机上：
- 看懂大方向
- 看清道路名
- 知道下一个 checkpoint 在哪里

但图本身不替代正式导航。
