# 工具定义：用户画像查询（user_profile_query）

> 工具类型：Dify 自定义工具 / API 工具
> 调用方式：Agent 在需要用户历史数据时自动调用
> 实现方式：方式A - 模拟数据（作品集演示用）；方式B - 对接真实 CRM/会员系统 API

---

## 工具描述

根据用户ID查询用户的完整画像数据，包括基础信息、消费统计、偏好标签、行为特征、会员状态等。Agent 在进行意图识别和策略决策前，需要调用本工具获取用户上下文。

---

## 输入参数

| 参数名 | 类型 | 必填 | 说明 | 示例 |
|--------|------|------|------|------|
| user_id | string | 是 | 用户唯一标识 | "U001" |

---

## 输出格式

```json
{
  "user_id": "U001",
  "basic_info": {
    "name": "王磊",
    "age": 28,
    "city": "上海",
    "gender": "男",
    "member_level": "黑金",
    "register_date": "2025-06-15",
    "register_days": 432
  },
  "consumption_stats": {
    "total_spent": 3200,
    "total_cups": 200,
    "monthly_frequency": 22,
    "avg_order_value": 16,
    "avg_cups_per_week": 5.5,
    "first_order_date": "2025-06-16",
    "last_order_date": "2026-08-20",
    "days_since_last_order": 1
  },
  "preference_tags": {
    "favorite_products": [
      {"product": "美式大杯", "ratio": 0.85},
      {"product": "拿铁大杯", "ratio": 0.15}
    ],
    "taste_preference": ["苦", "清爽", "低卡"],
    "temperature_preference": "冰",
    "sweetness_preference": "无糖",
    "milk_preference": "无",
    "price_sensitivity": "低",
    "coupon_usage_rate": 0.08
  },
  "behavior_patterns": {
    "usual_order_time": "工作日 8:30-9:00",
    "usual_location": "公司楼下门店",
    "usual_channel": "到店自取",
    "delivery_ratio": 0.05,
    "group_buy_ratio": 0.0,
    "new_product_try_rate": 0.1,
    "light_food_pairing_rate": 0.05,
    "max_consecutive_days": 45
  },
  "social_signals": {
    "community_joined": true,
    "community_active_level": "低",
    "group_buy_count_30d": 0,
    "share_count_30d": 0,
    "is_koc": false
  },
  "lifecycle": {
    "stage": "成熟用户",
    "stage_days": 380,
    "churn_risk": "低",
    "churn_risk_score": 5
  },
  "recent_5_orders": [
    {"date": "2026-08-20", "time": "08:50", "product": "美式大杯", "price": 13.6, "channel": "到店自取"},
    {"date": "2026-08-19", "time": "08:48", "product": "美式大杯", "price": 13.6, "channel": "到店自取"},
    {"date": "2026-08-18", "time": "08:52", "product": "美式大杯", "price": 13.6, "channel": "到店自取"},
    {"date": "2026-08-15", "time": "09:15", "product": "拿铁大杯", "price": 16.15, "channel": "到店自取"},
    {"date": "2026-08-14", "time": "08:45", "product": "美式大杯", "price": 13.6, "channel": "到店自取"}
  ]
}
```

---

## 输出字段说明

### basic_info（基础信息）
| 字段 | 类型 | 说明 |
|------|------|------|
| name | string | 用户昵称（脱敏处理） |
| age | integer | 年龄 |
| city | string | 所在城市 |
| member_level | string | 会员等级：小蓝豆/银卡/金卡/黑金 |
| register_date | string | 注册日期 |
| register_days | integer | 注册天数 |

### consumption_stats（消费统计）
| 字段 | 类型 | 说明 |
|------|------|------|
| total_spent | float | 累计消费金额（元） |
| total_cups | integer | 累计消费杯数 |
| monthly_frequency | integer | 近30天消费杯数 |
| avg_order_value | float | 平均客单价 |
| days_since_last_order | integer | 距上次消费天数 |

### preference_tags（偏好标签）
| 字段 | 类型 | 说明 |
|------|------|------|
| favorite_products | array | 偏好产品及占比（Top5） |
| taste_preference | array | 口味标签（苦/甜/清爽/丝滑等） |
| temperature_preference | string | 温度偏好（热/冰/常温） |
| sweetness_preference | string | 甜度偏好（无糖/三分/五分/七分/全糖） |
| price_sensitivity | string | 价格敏感度（低/中/高） |
| coupon_usage_rate | float | 优惠券使用率（0-1） |

### behavior_patterns（行为模式）
| 字段 | 类型 | 说明 |
|------|------|------|
| usual_order_time | string | 常下单时段 |
| usual_location | string | 常下单位置 |
| usual_channel | string | 常用渠道（到店自取/配送） |
| new_product_try_rate | float | 新品尝试率 |
| max_consecutive_days | integer | 最长连续打卡天数 |

### lifecycle（生命周期）
| 字段 | 类型 | 说明 |
|------|------|------|
| stage | string | 生命周期阶段 |
| churn_risk | string | 流失风险等级（低/中/高） |
| churn_risk_score | integer | 流失风险评分（0-100） |

---

## 实现方式

### 方式A：模拟数据（作品集演示用）
在 Dify 中使用"代码节点"或"HTTP请求工具"，内置10个模拟用户画像（见 `data/user-personas.md`），根据 user_id 返回对应数据。

**Dify 代码节点示例（Python）：**
```python
def main(user_id: str) -> dict:
    # 模拟用户数据库
    profiles = {
        "U001": {"name": "王磊", "member_level": "黑金", "monthly_frequency": 22, ...},
        "U002": {"name": "李思雨", "member_level": "金卡", "monthly_frequency": 8, ...},
        # ... 更多用户
    }
    return profiles.get(user_id, {"error": "用户不存在"})
```

### 方式B：对接真实 API
在 Dify 中注册自定义工具，对接品牌的 CRM/会员系统 API：
- 接口：`GET /api/v1/user/profile?user_id={user_id}`
- 认证：API Key / OAuth
- 超时：3秒
- 缓存：同一用户5分钟内缓存，减少重复调用

---

## 调用时机

Agent 在以下情况下应调用本工具：
1. 收到包含 user_id 的用户请求时，首先调用本工具获取画像
2. 进行意图识别前，需要用户历史数据作为判断依据
3. 进行产品推荐前，需要用户偏好数据
4. 计算优惠力度前，需要会员等级和价格敏感度

---

## 数据隐私说明

- 用户姓名等敏感信息在输出时做脱敏处理（如"王*"）
- 手机号、身份证号等PII数据不返回给Agent
- 所有数据访问记录日志，支持审计
- 符合《个人信息保护法》要求
