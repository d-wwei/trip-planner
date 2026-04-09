---
name: road-trip-planner
description: End-to-end road trip and travel itinerary planner that produces navigation-ready guides. Use this skill whenever the user asks to plan a road trip, self-drive tour, multi-day driving itinerary, or travel route — even if they just mention a destination and dates. Also trigger when the user asks for driving directions between multiple stops, wants to optimize a travel route, or needs a day-by-day travel guide with navigation links. Covers route planning, driving time verification, holiday/closure checks, parking details, and generates both markdown documents and interactive HTML guides with embedded maps and Google Maps navigation links.
---

# Road Trip Planner

Plan multi-day driving itineraries from a vague idea ("I want to drive the California coast") to navigation-ready guides with real-time-verified driving times, point-to-point Google Maps links, interactive maps, and per-stop parking/timing details.

## When This Skill Applies

- User wants to plan a road trip or multi-day driving tour
- User asks for a driving itinerary between cities/attractions
- User wants route optimization (eliminate backtracking)
- User needs a travel guide with navigation links
- User mentions self-drive, rental car, or road trip anywhere in their request

## Tool Compatibility

This skill works across different Claude environments. Adapt tool usage based on what's available:

| Capability | Claude.ai | Claude Code |
|------------|-----------|-------------|
| Coordinate lookup | `places_search` | `web_search` (search "[place] GPS coordinates") |
| Map verification | `places_map_display` | Generate Leaflet HTML preview, or use `web_search` to verify route on Google Maps |
| Driving time | `web_search` | `web_search` / MCP browser tools |
| File output | `{output_dir}` | Project directory (ask user or use `./trip-output/`) |

**Degradation rule**: If a tool is unavailable, fall back to `web_search` for data and Leaflet HTML for visualization. Never skip a verification step just because the preferred tool is missing.

## Gather Requirements

Before planning, collect these inputs. Ask the user if not provided:

| Input | Required | Example |
|-------|----------|---------|
| Destination / region | Yes | "California coast", "Iceland ring road" |
| Departure city | Yes | "Toronto", "San Francisco" |
| Travel dates | Yes | "April 3-6, 2026" |
| Number of days | Yes | 4 days |
| Transport | Yes | Rental car / own car |
| Route type | Yes | One-way (A→B) / Loop (A→…→A) / Out-and-back |
| Interests | Helpful | Nature, tech, food, photography |
| Budget level | Helpful | Budget / mid-range / comfort |
| Existing bookings | Helpful | Flight times, hotel confirmations |
| Group size | Helpful | Solo / couple / family |
| Output language | Helpful | 中文 / English / bilingual (default: 中文) |

**Route type matters for optimization**:
- **One-way**: Optimize for a single geographic direction, no doubling back
- **Loop**: Ensure the final leg returning to origin is accounted for in time/fuel budget
- **Out-and-back**: Plan the return route separately — avoid repeating the same stops, suggest alternative roads for the return

## Eight-Step Workflow

Execute these steps **in order, for each day**. Do not skip steps — every shortcut taken here (especially steps 2-5) leads to a broken itinerary the user will discover mid-trip.

### Step 1: Route Framework

Design the day's route based on user interests and geography.

- Assign a theme to each day (e.g., "Silicon Valley tech tour", "coastal highway scenic drive")
- List all candidate stops with addresses
- Use `places_search` to get **exact coordinates** for every stop — never rely on memory for coordinates
- Decide on overnight accommodation locations

### Step 2: Geographic Verification — Eliminate Backtracking

This step catches route inefficiencies that waste 30+ minutes per occurrence.

- Plot all stops on a map using `places_map_display` to visually verify the sequence
- Check that the route flows in one geographic direction (north→south, etc.) without doubling back
- If hotel/accommodation is between two stops, consider checking in first to drop luggage
- Reorder stops if the visual map reveals backtracking

**Common trap**: Visiting a southern stop, going north to the next, then returning south to the hotel. Always verify against the map.

### Step 3: Driving Time Verification

LLM estimates for driving times are **systematically optimistic by 30-50%**. This step is non-negotiable.

- Use `web_search` to find the actual driving time for each segment
- Search queries like: `driving time [origin] to [destination]` or `[origin] to [destination] drive how long`
- Factor in **day of week and time of day** — Friday 5 PM rush hour is very different from Sunday 10 AM
- Record the realistic time range (e.g., "45-55 min") not just the optimistic number
- If total driving + activity time exceeds the available daylight, cut stops or restructure

**Red flags to search for**:
- Rush hour on the planned route (search "[city] rush hour traffic [day of week]")
- Known bottleneck roads (PCH, Bay Bridge, LA freeways)
- Construction or seasonal closures (especially mountain passes)

### Step 4: Holiday and Closure Check

Search for every stop's operating status on the travel date.

- Use `web_search` with queries like: `[attraction] hours [holiday name] [year]` or `[attraction] open [date]`
- Check for: national holidays, religious holidays (Easter, Christmas), local observances
- Check weekday-specific closures (many museums close Mondays)
- Check seasonal closures (some trails/roads close in winter)
- If a stop is closed, note it prominently and suggest an alternative

**Holiday calendar to check against**:
- US federal holidays, Easter weekend (Good Friday through Easter Monday), Christmas week
- Country-specific holidays for international trips
- "First Monday closed" pattern common at museums

### Step 5: Weather Forecast and Contingency Planning

Check the weather for each day's route and prepare alternatives.

- Use `web_search` with queries like: `[city] weather forecast [date]` or `[region] weather [month] [year]`
- For each day, record: temperature range, precipitation probability, wind conditions, sunrise/sunset times
- **Sunset time matters**: If a stop's highlight is sunset (e.g., Big Sur, Santa Monica), verify the exact sunset time and work backwards to plan arrival

**Weather-driven adjustments**:
- Rain forecast → swap outdoor hikes with indoor alternatives (museums, markets, aquariums), add "rain plan" note to the stop card
- Extreme heat (>35°C / 95°F) → schedule outdoor activities for early morning or late afternoon, note hydration stops
- Fog (common on Pacific Coast) → warn that coastal views may be obscured in the morning, suggest delaying departure
- Snow/ice (mountain passes) → check road closures, add chain requirement warnings

**Feed results into Step 8** (Pre-trip Checklist) for packing recommendations.

### Step 6: Generate Markdown Document

Create one `.md` file per day in `{output_dir}`. Follow this exact structure:

```
# Day N 详细攻略：[Route Summary]

> ⚠️ Holiday/special date notice if applicable

## Flight/Travel Info (if applicable)

## 🧭 Navigation Links Summary Table
| Segment | Route | Nav Link | Drive Time |
Top-level quick-reference with all segments

## [Time Range] | [Section Title]

### [Time] | [Stop Name] ([Duration])
- 🧭 **[Nav Link: Origin → Destination](Google Maps URL)**
- Address, hours, tickets
- Parking: location, cost, tips
- What to do (prioritized list)
- Safety warnings if any

## Day N Timeline Overview
| Time | Stop | Duration | Drive Time | Notes |

## Practical Info
- Weather: temperature, precipitation, sunset time
- Cost estimates (use structured budget template — see below)
- Emergency contacts
- Gas/fuel notes
- Rain plan alternatives (if applicable)
```

**Navigation link rules** (critical — validated through 4-day real-world testing):

1. **Destination-only format**: `https://www.google.com/maps/dir/?api=1&destination=PLACE+NAME+CITY+STATE&travelmode=driving`
2. **Never add `origin` parameter** — user may not be at the expected location, causing Google Maps to route from the wrong place
3. **Never add `waypoints` parameter** — waypoints can route users onto closed/restricted roads (e.g., Cañada Rd incident — 30+ min wasted)
4. **Prefer place names over coordinates** — coordinates are error-prone (NVIDIA photo spot, Stanford entrance). Only use coordinates when place name is genuinely ambiguous, and web_search the coordinates first
5. **Two-segment navigation for every stop**: always provide BOTH a driving link to the parking lot AND a walking link to the attraction entrance. Users need to know where to park separately from where to go
6. **Two layers of nav links**: summary table at top AND inline in each section
7. **Include skip links**: "If skipping stop X, click here to go directly from A to C"
8. **Include conditional rules**: "If you arrive at [Station X] after [TIME], skip [Station Y] to protect [hard deadline Z]"

### Step 7: Generate Interactive HTML Guide

Create one `.html` file per day in `{output_dir}`. Read `references/html-template.md` for the complete template code before writing.

Key requirements:
- **Leaflet.js map** with numbered color-coded pins and dashed route line
- **CARTO Voyager tiles** (not Google Maps iframe — it fails in restricted environments)
- **Blue navigation buttons** on every stop card that open Google Maps with coordinate URLs
- **Highlight the day's main event** with gold border (`.hl` class)
- **Optional stops** get dashed-outline numbers (`.sn-o` class)
- **Alert boxes**: yellow for cost warnings, red for safety warnings
- Mobile-optimized, max-width 480px
- Light theme (white cards on off-white background) for readability

### Step 8: Generate Pre-trip Checklist

Create a single `pre-trip-checklist.md` file in `{output_dir}`. This is generated **once for the entire trip**, not per day.

**Weather-driven packing section** (based on Step 5 forecast data):

```
# 行前清单 | Pre-trip Checklist

## 🌤️ 天气概览 & 行李建议
| 日期 | 地区 | 天气 | 温度 | 建议穿搭/装备 |
|------|------|------|------|---------------|
| Day 1 | [Region] | ☀️ 晴 | 12-22°C | 薄外套+防晒, 墨镜 |
| Day 2 | [Region] | 🌧️ 小雨 | 8-15°C | 防水外套, 防滑鞋 |
| ... | | | | |

### 必带行李
- [ ] 根据天气列出的衣物（具体到件）
- [ ] 防晒霜 SPF50+（户外日必备）
- [ ] 雨具（如有雨天）
- [ ] 舒适步行鞋
- [ ] 车载手机支架
- [ ] 充电线 + 车载充电器

## 📄 证件 & 预订
- [ ] 护照 / 身份证（有效期检查）
- [ ] 国际驾照 / 驾照翻译件（如需）
- [ ] 租车确认号：[号码] | 取车地址：[地址]
- [ ] 机票确认号：[号码]
- [ ] 酒店确认号：逐晚列出
- [ ] 保险单号 / 保险公司电话
- [ ] 信用卡（确认境外支付已开通）

## ⛽ 车辆 & 驾驶
- [ ] 租车公司电话：[电话]
- [ ] 道路救援电话：[电话]
- [ ] 加油类型：Regular / Premium
- [ ] 过路费准备（现金 / ETC / FasTrak）
- [ ] 了解当地交规（限速、右转规则、停车标志）

## 🆘 紧急信息
- [ ] 当地报警电话：[号码]
- [ ] 最近医院位置（每天路线附近各标一个）
- [ ] 大使馆/领事馆电话（国际旅行）
- [ ] 同行人紧急联系方式

## 📱 App & 离线准备
- [ ] Google Maps 离线地图已下载（覆盖全程区域）
- [ ] 翻译 App（如需）
- [ ] 加油站 App（GasBuddy 等）
- [ ] 酒店/民宿 App 已登录
```

**Rules**:
- Packing items must be **specific to the weather forecast**, not generic — "薄羽绒" vs "外套" makes a difference
- If any day has rain >50% probability, add rain gear to must-bring
- If temperature drops below 10°C at any point, add warm layers
- If visiting beaches/coastal areas, add swimwear + towel even if not explicitly planned
- Always include offline map download reminder — cell signal is unreliable on scenic routes

## Output Directory

Before generating any files, confirm the output directory:
- Claude.ai: use `/mnt/user-data/outputs/`
- Claude Code: ask the user, or default to `./trip-output/`
- Create the directory if it doesn't exist

## Output File Naming

```
Day1-[Region]-攻略.md          Day1-[Region]-攻略.html
Day2-[Region]-攻略.md          Day2-[Region]-攻略.html
pre-trip-checklist.md          (行前清单，整个行程一份)
budget-summary.md              (费用汇总，整个行程一份)
...
```

## Structured Budget Template

Generate a single `budget-summary.md` alongside the daily guides. Use this fixed structure:

```
# 💰 费用预估 | Trip Budget Summary

## 按类别汇总
| 类别 | 明细 | 费用 (USD) |
|------|------|-----------|
| ✈️ 机票 | [航班信息] | $XXX |
| 🚗 租车 | [天数] × [日租] + 保险 | $XXX |
| ⛽ 油费 | [总里程] ÷ [油耗] × [油价] | $XXX |
| 🅿️ 停车 | 逐站列出 | $XXX |
| 🏨 住宿 | 逐晚列出 | $XXX |
| 🍽️ 餐饮 | [天数] × [每日预算] | $XXX |
| 🎫 门票/活动 | 逐项列出 | $XXX |
| 🛣️ 过路费 | 逐段列出 | $XXX |
| 📱 通讯 | SIM卡 / 漫游 | $XXX |
| 🔄 杂费预留 | 10-15% buffer | $XXX |
| **合计** | | **$XXXX** |

## 按天汇总
| 日期 | 住宿 | 交通 | 餐饮 | 活动 | 小计 |
|------|------|------|------|------|------|
| Day 1 | $XX | $XX | $XX | $XX | $XXX |
| ... | | | | | |
| **合计** | | | | | **$XXXX** |
```

**Budget rules**:
- Always include a 10-15% buffer for unexpected costs
- If group size > 1, add a per-person column: `| 人均 | $XXX / person |`
- If group wants to split costs, add a "费用分摊" section with who-pays-what
- Use web search to verify current prices for tickets, parking, tolls — don't rely on memory
- Currency should match the destination country, with home-currency equivalent in parentheses if different

## Quality Checks Before Delivering

Run through this checklist for every day before presenting to the user:

- [ ] Every nav link uses destination-only format (no origin, no waypoints)
- [ ] Every stop has TWO nav links: drive-to-parking + walk-to-attraction
- [ ] Every driving time was web-searched, not estimated
- [ ] Route has no backtracking (verified on map)
- [ ] All stops confirmed open on the travel date (including holiday/weekday-specific checks)
- [ ] Weather checked for each day, rain plans noted where needed
- [ ] Timeline math adds up (arrival + duration + drive = next arrival)
- [ ] Timeline includes realistic buffers: +30min for car pickup, +15min for parking in cities, +50% for museums/markets
- [ ] Parking info included for every stop — specific parking lot name, not just "nearby parking"
- [ ] Skip-links provided for optional stops
- [ ] Conditional rules included: "if you arrive at X after TIME, skip Y to protect Z"
- [ ] "Optional" stops are ranked by cut priority (which to cut first when behind schedule)
- [ ] HTML map pins match the document's stop order
- [ ] Total day doesn't exceed reasonable hours (typically 8 AM - 10 PM)
- [ ] Hard deadlines noted (flight times, restaurant reservations, sunset times)
- [ ] Rental car pickup section includes anti-upsell checklist
- [ ] Pre-trip checklist generated with weather-specific packing list
- [ ] Budget summary generated with per-category and per-day breakdown
- [ ] Offline map download reminder included
