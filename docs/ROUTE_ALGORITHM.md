# Route Generation & Scoring Logic v0.1

## 1. 数据准备

正式算法必须按以下顺序执行，不能先画图再补 POI：

1. 用高德地理编码/POI 搜索确定起点和终点坐标，并保留 `poi_id`、地址和名称。
2. 先调用高德步行路径规划 API 获取 A→B 主路线，保存完整步行 polyline、道路步骤和距离。
3. 以主路线 polyline 建立走廊缓冲区，并在走廊及左右扩展区域切分网格；逐格调用高德周边/关键词搜索，合并并去重便利店 POI。
4. 对每个候选 checkpoint 做插入测试：分别计算前一点→候选点→后一点的步行路线，比较新增距离、偏离距离、方向进展和是否重复路线段。
5. 只从通过插入测试的真实 POI 中组装路线，再进行数量、距离、间距和导航评分；便利店数量不足时减少数量，不得强行加入远点。
6. 对每条最终候选路线逐段调用高德步行 API，并保存每段 polyline；最后才生成路线图，图上的店名、道路、编号和连线必须来自这些结果。

### 1.1 网格搜索

- 搜索范围应覆盖起终点之间的走廊，并向左右扩展，以允许顺路绕行。
- 网格大小和搜索半径按城市密度调整；每个网格独立查询并记录来源。
- 合并结果时按 `poi_id` 优先去重；无 `poi_id` 时按名称、地址和坐标近似去重。
- 网格搜索只产生候选，不代表店铺一定在最终步行路线旁。
- 网格核验图必须展示每个网格内检索到的全部便利店候选，包含名称、地址、坐标和来源网格；不得在此阶段按“顺路”删店。
- POI 检索至少覆盖“便利店、士多、7-11、全家、罗森、美宜佳、超市”等关键词，并逐页读取结果；不同关键词和网格的结果按 `poi_id` 或名称+地址+坐标去重。

### 1.2 路线走廊与插入校验

- 主路线是事实基线；候选店优先选择距主路线步行偏离较小、且位于 A→B 前进方向的 POI。
- `增量距离 = (前点→候选点 + 候选点→后点) - (前点→后点)`，优先选择增量较小的候选。
- 若候选点使路线反向进入已经过的区域、重复主要道路段，或显著增加总距离，则淘汰。
- 便利店目标是软约束；真实路线连续性、可导航性和终点正确性优先级更高。

### 1.3 逐点动态选择

路线生成必须按以下循环执行：

1. 固定并核验起点 A、终点 B 的 POI 坐标。
2. 在 A 与 B 周边建立 3×3 网格，逐格搜索并标记全部便利店 POI。
3. 当前位置设为 A；从当前位置可达且尚未使用的候选 POI 中，选择最适合的下一点：优先考虑朝向 B 的方向进展、较小的步行增量、与上一点合理的间距，以及对目标数量的贡献。
4. 到达该点后，将当前位置更新为该点，重新计算“当前位置→B”的方向和剩余可用 POI，再选择下一点。
5. 若候选点要求回到已经过的区域、重复道路段、明显远离 B，或会使总距离超过上限，则淘汰。
6. 直到继续前往 B 比增加 checkpoint 更合理，或达到目标数量/距离上限；最后调用高德步行 API 验证当前位置→B 的终段。

这是一种动态的逐点路线选择，不允许把全量 POI 按距离一次性排序后直接连线。

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
