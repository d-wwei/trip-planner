# Road Trip Planner Skill

A Claude Code skill that turns a vague travel idea into navigation-ready road trip guides — with verified driving times, interactive maps, weather-aware packing lists, and one-tap Google Maps navigation.

## What It Does

Give it a destination and dates, get back a complete self-drive itinerary:

| Output | Format | Description |
|--------|--------|-------------|
| Daily guide | `.md` | Detailed per-stop breakdown with nav links, parking, timing, meals |
| Interactive map | `.html` | Leaflet map + navigation buttons + stop cards (mobile-optimized) |
| Pre-trip checklist | `.md` | Weather-driven packing list, documents, emergency info, offline prep |
| Budget summary | `.md` | Per-category and per-day cost breakdown |

## 8-Step Workflow

1. **Route Framework** — Theme each day, list stops, get exact coordinates
2. **Geographic Verification** — Plot on map, eliminate backtracking
3. **Driving Time Verification** — Web search every segment (LLM estimates are 30-50% too optimistic)
4. **Holiday & Closure Check** — Confirm every stop is open on travel date
5. **Weather Forecast** — Check forecast, plan rain/heat alternatives, note sunset times
6. **Generate Markdown** — Daily guide with dual-layer navigation links (coordinate format only)
7. **Generate HTML** — Leaflet map + CARTO tiles + nav buttons + offline fallback
8. **Pre-trip Checklist** — Weather-specific packing, documents, vehicle prep, offline maps

## Key Design Decisions (Learned from 4-Day Real-World Testing)

- **Destination-only nav links, no origin/waypoints** — origin param routes from the wrong place; waypoints can send you onto closed roads (lost 30+ min on Cañada Rd)
- **Every stop gets TWO nav links** — one for driving to the parking lot, one for walking to the entrance
- **Prefer place names over coordinates** — coordinates are error-prone; only use them when names are genuinely ambiguous
- **Driving times are web-searched, never estimated** — LLM memory is systematically optimistic by 30-50%
- **Routes are verified on a map** — reading lat/lng numbers doesn't catch backtracking
- **Conditional skip rules** — "if you arrive at X after TIME, skip Y to protect Z" prevents cascading delays
- **Timeline buffers built in** — +30min car pickup, +15min city parking, +50% for museums/markets
- **HTML uses Leaflet + CARTO, not Google Maps iframe** — iframe fails in restricted environments
- **Offline fallback built in** — scenic routes often have no cell signal

## Installation

### Claude Code (CLI / Desktop / IDE)

Copy the `.skill` file to your Claude Code skills directory, or reference the skill files directly.

### Claude.ai (Artifacts)

Upload `SKILL.md` as a project file. The skill will use `places_search` and `places_map_display` natively.

## File Structure

```
road-trip-planner/
├── SKILL.md                    # Main skill definition (8-step workflow)
└── references/
    └── html-template.md        # Complete HTML/CSS/JS template for interactive guides
```

## Example Input

> "I want to drive from San Francisco to LA along the coast, April 3-6, rental car, interested in tech companies and ocean views, mid-range budget, solo trip"

## Example Output

- `Day1-SiliconValley-攻略.md` + `.html`
- `Day2-BigSur-攻略.md` + `.html`
- `Day3-SantaBarbara-攻略.md` + `.html`
- `Day4-LA-攻略.md` + `.html`
- `pre-trip-checklist.md`
- `budget-summary.md`

## Compatibility

| Environment | Coordinate Lookup | Map Verification | File Output |
|-------------|------------------|-----------------|-------------|
| Claude.ai | `places_search` | `places_map_display` | `/mnt/user-data/outputs/` |
| Claude Code | `web_search` | Leaflet HTML preview | Project directory |

## License

MIT

---

# Road Trip Planner Skill（自驾旅行规划技能）

一个 Claude Code 技能，把模糊的旅行想法变成导航级自驾攻略 — 包含实时校准的驾车时间、交互式地图、天气驱动的行李清单、一键 Google Maps 导航。

## 功能概览

输入目的地和日期，输出完整的自驾行程：

| 输出 | 格式 | 说明 |
|------|------|------|
| 每日攻略 | `.md` | 每站详细信息：导航链接、停车、时间、餐饮 |
| 交互式地图 | `.html` | Leaflet 地图 + 导航按钮 + 卡片式 UI（移动端优化） |
| 行前清单 | `.md` | 基于天气的穿搭建议、证件、紧急信息、离线准备 |
| 费用预估 | `.md` | 按类别 + 按天的双维度费用汇总 |

## 8 步工作流

1. **路线框架** — 每天分配主题，列出站点，获取精确坐标
2. **地理校验** — 地图上绘制路线，消除回头路
3. **驾车时间校准** — Web 搜索每一段实际驾车时间（LLM 估算普遍偏乐观 30-50%）
4. **节假日/闭馆检查** — 确认旅行日期内各景点正常营业
5. **天气预报** — 查天气预报，规划雨天/高温备选方案，标注日落时间
6. **生成 Markdown** — 每天一份攻略，双层导航链接（仅坐标格式）
7. **生成 HTML** — Leaflet 地图 + CARTO 瓦片 + 导航按钮 + 离线降级
8. **行前清单** — 天气驱动的行李清单、证件汇总、车辆准备、离线地图

## 核心设计决策（4 天实测总结）

- **导航链接只设 destination，不加 origin/waypoints** — origin 会从错误位置导航；waypoints 可能导入封闭道路（Cañada Rd 事件浪费 30+ 分钟）
- **每站生成两条导航** — 一条驾车到停车场，一条步行到景点入口
- **优先用地名而非坐标** — 坐标容易出错，仅在地名有歧义时使用
- **驾车时间必须搜索验证** — LLM 估算系统性偏乐观 30-50%
- **路线必须在地图上验证** — 看经纬度数字无法发现回头路
- **内置条件跳过规则** — "如果到达 X 站超过某时间，跳过 Y 以保护 Z"，防止连锁延误
- **时间缓冲内置** — 取车 +30min，城市停车 +15min，博物馆/市场 +50%
- **HTML 用 Leaflet + CARTO，不用 Google Maps iframe** — iframe 在受限环境加载失败
- **内置离线降级** — 风景公路常常没有手机信号

## 安装

### Claude Code（CLI / 桌面 / IDE）

将 `.skill` 文件复制到 Claude Code 技能目录，或直接引用技能文件。

### Claude.ai（Artifacts）

将 `SKILL.md` 作为项目文件上传。技能会原生使用 `places_search` 和 `places_map_display`。

## 许可证

MIT
