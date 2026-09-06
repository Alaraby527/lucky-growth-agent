# 工具定义：意图识别（intent_recognition）

> 工具类型：Dify 自定义工具 / LLM 推理工具
> 调用方式：Agent 在 ReAct 循环中自动调用
> 实现方式：方式A - 由 LLM 基于系统提示词直接推理（推荐，零代码）；方式B - 独立模型服务

---

## 工具描述

分析用户的多维度上下文信号（时间、天气、位置、行为、社交），识别用户当前的消费意图类型，输出意图分类、置信度和判断依据。

这是 Agent 感知层的核心工具，所有后续决策都基于本工具的输出。

---

## 输入参数

| 参数名 | 类型 | 必填 | 说明 | 示例 |
|--------|------|------|------|------|
| user_id | string | 是 | 用户唯一标识 | "U001" |
| current_time | string | 是 | 当前时间（含星期和时段） | "周一 8:45" |
| weather | string | 否 | 天气和温度 | "晴 21°C" |
| location | string | 否 | 用户当前位置 | "公司楼下" |
| recent_orders | array | 否 | 最近5笔订单（产品+时间） | [{"product":"美式大杯","time":"昨天8:50"}] |
| monthly_frequency | integer | 否 | 近30天消费杯数 | 22 |
| preference_tags | object | 否 | 偏好标签及占比 | {"美式":0.85,"拿铁":0.15} |
| social_signals | object | 否 | 社交信号（拼单/分享/社群） | {"group_buy_count":2,"shared":false} |
| member_level | string | 否 | 会员等级 | "黑金" |
| special_date | string | 否 | 特殊日期标记 | "发薪日"/"节日"/"无" |

---

## 输出格式

```json
{
  "intent_type": "习惯复购",
  "intent_category": "routine",
  "confidence": 92,
  "confidence_level": "high",
  "lifecycle_stage": "成熟用户",
  "reasoning": [
    "工作日早晨8:45，与历史下单时段8:50高度吻合",
    "位置在公司楼下，符合日常到店自取模式",
    "连续3天同一时段点美式大杯，行为一致性极高",
    "月消费22杯，黑金会员，高频成熟用户",
    "美式大杯占比85%，偏好极其明确"
  ],
  "signal_weights": {
    "time": 0.30,
    "location": 0.25,
    "behavior": 0.20,
    "weather": 0.15,
    "social": 0.10
  },
  "secondary_intent": null,
  "conflict_signals": []
}
```

### 输出字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| intent_type | string | 六大意图之一：提神刚需/社交分享/放松享受/尝鲜探索/性价比驱动/习惯复购/无法确定 |
| intent_category | string | 意图分类英文标识：caffeine/social/relax/explore/value/routine/unknown |
| confidence | integer | 置信度分数（0-100） |
| confidence_level | string | high(≥80)/medium(50-79)/low(<50) |
| lifecycle_stage | string | 新用户/成长用户/成熟用户/预警流失 |
| reasoning | array | 3-5条关键判断依据 |
| signal_weights | object | 各信号维度的权重（用于可解释性） |
| secondary_intent | string/null | 次要意图（混合意图场景） |
| conflict_signals | array | 冲突信号列表（如"偏好冰饮但天气寒冷"） |

---

## 意图判断规则（内置逻辑）

### 信号权重
- 时间维度：30%
- 位置维度：25%
- 行为维度：20%
- 环境维度：15%
- 社交维度：10%

### 快速判断规则

| 意图类型 | 核心判断条件（满足2条以上） |
|----------|---------------------------|
| 提神刚需 | 早晨/深夜时段 + 工作地点 + 高咖啡因偏好 + 近期加班/熬夜信号 |
| 社交分享 | 下午/周末 + 多人/商场位置 + 拼单/分享历史 + 特调偏好 |
| 放松享受 | 周末/假期 + 家/休闲场所 + 高端款/甜品偏好 + 慢节奏时段 |
| 尝鲜探索 | 新品上市期 + 历史新品购买率>50% + 社群活跃 + 无固定偏好 |
| 性价比驱动 | 优惠券使用率>80% + 低价产品偏好 + 大促/囤券行为 + 学生/新客 |
| 习惯复购 | 固定时段+固定产品+固定位置 + 月消费>10杯 + 行为一致性>80% |

### 置信度计算
- 信号完整度（40%）：必填参数是否齐全
- 判断一致性（30%）：多维度信号是否指向同一意图
- 历史匹配度（20%）：当前场景与历史行为的匹配程度
- 分类清晰度（10%）：该意图与其他意图的区分度

---

## 调用示例

### 示例1：高置信度习惯复购

**输入：**
```json
{
  "user_id": "U001",
  "current_time": "周一 8:45",
  "weather": "晴 21°C",
  "location": "公司楼下",
  "recent_orders": [
    {"product": "美式大杯", "time": "昨天8:50"},
    {"product": "美式大杯", "time": "前天8:48"}
  ],
  "monthly_frequency": 22,
  "preference_tags": {"美式大杯": 0.85, "拿铁大杯": 0.15},
  "member_level": "黑金"
}
```

**输出：** 见上方输出格式示例（intent_type=习惯复购, confidence=92）

### 示例2：低置信度新用户

**输入：**
```json
{
  "user_id": "U007",
  "current_time": "周二 12:00",
  "weather": null,
  "location": null,
  "recent_orders": [],
  "monthly_frequency": 0,
  "preference_tags": {}
}
```

**输出：**
```json
{
  "intent_type": "无法确定",
  "intent_category": "unknown",
  "confidence": 30,
  "confidence_level": "low",
  "lifecycle_stage": "新用户",
  "reasoning": [
    "无任何消费记录和偏好数据",
    "天气、位置等场景信号缺失",
    "仅有注册行为，无法判断消费意图"
  ],
  "secondary_intent": null,
  "conflict_signals": []
}
```

---

## 实现方式

### 方式A：LLM 直接推理（推荐，零代码）
不需要单独开发工具，在系统提示词中明确意图分类体系和判断规则，由 Agent 的 LLM 直接基于用户输入进行推理。这是本项目采用的方式。

**优点**：零开发成本、灵活、可解释性强
**缺点**：依赖 LLM 能力，结果有一定随机性

### 方式B：独立分类模型
训练一个专门的意图分类模型（如 BERT/FastText），部署为 API 服务，在 Dify 中注册为自定义工具。

**优点**：速度快、成本低、结果稳定
**缺点**：需要训练数据和开发成本、更新不灵活

### 方式C：规则引擎 + LLM 兜底
用规则引擎处理高置信度的明确场景（如固定习惯复购），LLM 处理复杂和边缘场景。

**优点**：兼顾速度和灵活性
**缺点**：规则维护成本较高

---

## 版本记录

| 版本 | 日期 | 变更 |
|------|------|------|
| v1.0 | 2026-08-21 | 初始版本，六大意图分类+置信度分级 |
