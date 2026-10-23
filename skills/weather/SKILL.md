---
name: weather
description: "通过 wttr.in 或 Open-Meteo 获取当前天气和预报。适用场景：用户询问任意地点的天气、温度或预报。不适用：历史天气数据、恶劣天气预警或详细气象分析。无需 API key。"
homepage: https://wttr.in/:help
metadata:
  {
    "brigade":
      {
        "emoji": "☔",
        "requires": { "bins": ["curl"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "curl",
              "bins": ["curl"],
              "label": "安装 curl (brew)",
            },
          ],
      },
  }
---

# 天气技能

获取当前天气状况和预报。

## 何时使用

✅ **以下情况使用此技能：**

- “天气怎么样？”
- “今天/明天会下雨吗？”
- “[城市]的温度”
- “一周天气预报”
- 出行规划时的天气查询

## 何时不使用

❌ **以下情况不要使用此技能：**

- 历史天气数据 → 使用天气档案/API
- 气候分析或趋势 → 使用专业数据源
- 超本地微气候数据 → 使用本地传感器
- 恶劣天气预警 → 查看官方 NWS 来源
- 航空/海洋天气 → 使用专业服务（METAR 等）

## 地点

天气查询中始终包含城市、地区或机场代码。

## 命令

### 当前天气

```bash
# 一行摘要
curl "wttr.in/London?format=3"

# 详细的当前天气状况
curl "wttr.in/London?0"

# 指定城市
curl "wttr.in/New+York?format=3"
```

### 预报

```bash
# 3 天预报
curl "wttr.in/London"

# 一周预报
curl "wttr.in/London?format=v2"

# 指定日期（0=今天，1=明天，2=后天）
curl "wttr.in/London?1"
```

### 格式选项

```bash
# 单行输出
curl "wttr.in/London?format=%l:+%c+%t+%w"

# JSON 输出
curl "wttr.in/London?format=j1"

# PNG 图片
curl "wttr.in/London.png"
```

### 格式代码

- `%c` — 天气状况 emoji
- `%t` — 温度
- `%f` — “体感温度”
- `%w` — 风速
- `%h` — 湿度
- `%p` — 降水量
- `%l` — 地点

## 快速响应

**“天气怎么样？”**

```bash
curl -s "wttr.in/London?format=%l:+%c+%t+(feels+like+%f),+%w+wind,+%h+humidity"
```

**“会下雨吗？”**

```bash
curl -s "wttr.in/London?format=%l:+%c+%p"
```

**“周末预报”**

```bash
curl "wttr.in/London?format=v2"
```

## 注意事项

- 无需 API key（使用 wttr.in）
- 有速率限制；不要频繁请求
- 适用于大多数全球城市
- 支持机场代码：`curl wttr.in/ORD`