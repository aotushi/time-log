# time-log

个人时间记录系统，基于 ActivityWatch + Cloudflare Worker + GitHub 构建。自动采集电脑活动数据，结合每日手动补录，生成完整的时间使用记录。

---

## 系统架构

```
PC端 ActivityWatch
│  自动记录窗口/应用/浏览器活动
│  AFK + window 双重校正，过滤虚假时间
│
├── sync.py（每小时自动运行）
│   ├── 采集当天数据 → data/raw/
│   └── 每天早上生成昨日报告 → reports/daily/
│
网页表单（Cloudflare Worker）
│  手动补录离线活动（吃饭/睡觉/散步等）
│  每日反思（专注质量/状态/明日计划）
└── 提交 → data/manual/
```

---

## 仓库结构

```
time-log/
├── data/
│   ├── raw/                  # PC 自动采集数据（每日 JSON）
│   │   └── YYYY-MM-DD.json
│   └── manual/               # 表单手动数据（每日 JSON）
│       └── YYYY-MM-DD.json
└── reports/
    └── daily/                # 每日 Markdown 报告（自动生成）
        └── YYYY-MM-DD.md
```

---

## 数据格式

### raw 数据（自动采集）

```json
{
  "date": "2026-03-29",
  "synced_at": "2026-03-29T07:00:00Z",
  "computer": {
    "active_minutes": 73.9,
    "afk_minutes": 360.0,
    "active_intervals": [
      { "start": "09:00", "end": "12:00" },
      { "start": "13:30", "end": "18:00" }
    ],
    "apps": [{ "app": "msedge.exe", "minutes": 37.9 }],
    "sites": [{ "site": "claude.ai", "minutes": 6.2 }]
  }
}
```

### manual 数据（手动填写）

```json
{
  "date": "2026-03-29",
  "submitted_at": "2026-03-29T23:30:00Z",
  "wake_time": "07:10",
  "sleep_time": "23:30",
  "activities": [
    { "name": "吃午饭", "start": "12:00", "end": "12:45" },
    { "name": "散步", "start": "18:30", "end": "19:10" }
  ],
  "focus_quality": 3,
  "mood": "状态还不错",
  "tomorrow": "早点睡"
}
```

---

## 技术栈

| 组件                                        | 用途                 |
| ------------------------------------------- | -------------------- |
| [ActivityWatch](https://activitywatch.net/) | PC端活动自动采集     |
| Python + PyGithub                           | 数据处理与GitHub同步 |
| Windows 任务计划程序                        | 定时触发sync.py      |
| Cloudflare Worker                           | 网页表单托管与API    |
| GitHub                                      | 数据存储与版本管理   |

---

## 数据质量保障

**AFK校正**：鼠标键盘无操作超过10分钟标记为离开，过滤短暂离开（<10分钟）避免时间段碎片化。

**Window校正**：网站使用时长与对应浏览器前台时间取交集，解决浏览器最小化但标签页仍计时的问题。

**手动优先原则**：表单填写的时间段优先于自动采集数据，解决"电脑播放视频但人已离开"等场景的冲突。

---

## 设计原则

- **自动为主，手动为辅**：电脑端全自动，每天只需3-5分钟填写表单补录离线活动
- **数据只增不改**：历史数据永远保留，报告可随时重新生成
- **原型优先**：当前为验证阶段，待数据积累后统一迁移至 Flutter App

---

## 路线图

- [x] PC端 ActivityWatch 数据采集
- [x] AFK + Window 双重校正
- [x] GitHub 自动同步（每小时）
- [x] 网页表单（Cloudflare Worker）
- [x] 每日 Markdown 报告自动生成
- [ ] 手机端数据采集（Flutter App）
- [ ] 周/月汇总报告
- [ ] 数据可视化面板
- [ ] 多设备冲突解决（CRDT）

---

## 隐私说明

本仓库为**私有仓库**，包含个人时间使用、浏览记录等隐私数据，请勿设为公开。

