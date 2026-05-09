# 房间签到管理系统 - 技术架构文档

## 1. 架构设计

```
┌─────────────────────────────────────────────────────────┐
│                    单HTML文件应用                        │
├─────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐ │
│  │   UI层      │  │  业务逻辑层  │  │    数据层       │ │
│  │  (DOM/CSS)  │  │  (JavaScript)│  │  (IndexedDB)    │ │
│  └─────────────┘  └─────────────┘  └─────────────────┘ │
│         ↕                ↕                  ↕          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              事件处理 & 状态管理                   │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## 2. 技术选型

| 类别 | 技术 | 说明 |
|------|------|------|
| 文件格式 | 单HTML文件 | 包含内联CSS和JavaScript |
| 数据存储 | IndexedDB | 浏览器本地存储，Dexie.js封装 |
| CSV解析 | PapaParse | 轻量级CSV解析库 |
| 图表库 | 无 | 纯表格展示，无外部依赖 |
| 图标 | Unicode符号 | 无外部图标依赖 |

## 3. 外部依赖（CDN）

```html
<!-- Dexie.js - IndexedDB封装 -->
<script src="https://cdn.jsdelivr.net/npm/dexie@3.2.4/dist/dexie.min.js"></script>

<!-- PapaParse - CSV解析 -->
<script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
```

## 4. 数据模型

### 4.1 IndexedDB数据库结构

```
数据库名: RoomCheckInDB (版本: 1)
├── rooms 表
│   ├── id (主键, string)
│   ├── name (索引, string)
│   ├── signer (string)
│   └── createdAt (number)
│
└── checkins 表
    ├── id (主键, string)
    ├── roomId (索引, string)
    ├── checkInDate (索引, string, YYYY-MM-DD格式)
    └── createdAt (number)
```

### 4.2 状态计算逻辑

```
距下次签到天数 = 下次签到日 - 今天

状态判定:
- 已签到: 距下次签到 >= 14天
- 明天需签到: 距下次签到 = 1天
- 后天需签到: 距下次签到 = 2天
- 未签到: 其他情况

下次签到日计算:
- 找到房间的最后一条签到记录
- 下次签到日 = 最后签到日 + 15天
```

## 5. 模块划分

### 5.1 核心模块

| 模块 | 职责 |
|------|------|
| `DB` | IndexedDB数据库操作封装 |
| `RoomManager` | 房间CRUD操作 |
| `CheckInManager` | 签到记录管理 |
| `StateCalculator` | 状态计算逻辑 |
| `UIController` | 界面渲染与交互 |
| `ExportManager` | CSV导出功能 |
| `ImportManager` | CSV导入功能 |
| `BatchProcessor` | 批量操作处理 |

### 5.2 主要函数

```javascript
// 数据库初始化
initDB() → Promise<IDBDatabase>

// 房间操作
createRoom(name, signer) → Promise<string>
updateRoom(id, data) → Promise<void>
deleteRoom(id) → Promise<void>
getAllRooms() → Promise<Room[]>

// 签到操作
checkIn(roomId, date) → Promise<void>
undoCheckIn(roomId, date) → Promise<void>
getLastCheckIn(roomId) → Promise<CheckIn | null>
getCheckInsByDate(date) → Promise<CheckIn[]>
getCheckInsByRange(roomId, startDate, endDate) → Promise<CheckIn[]>

// 状态计算
calculateStatus(roomId) → { status, daysUntil, nextDate }

// 导出功能
exportRoomsCSV() → void
exportCheckInsCSV(startDate, endDate) → void
exportFullBackup() → void

// 导入功能
importRoomsCSV(file) → Promise<{ success: number, failed: number }>
importFullBackup(json) → Promise<void>
```

## 6. 界面组件

| 组件 | 描述 |
|------|------|
| 顶部工具栏 | 标题、导出、导入按钮 |
| 操作工具栏 | 批量操作、快速查询、新增、CSV导入 |
| 主数据表格 | 房间列表， checkbox选择、状态颜色 |
| 单元格编辑 | 双击进入编辑模式 |
| 右键菜单 | 上下文菜单操作 |
| 模态对话框 | 新增房间、编辑、批量操作 |
| 日期选择器 | 快速查询日期选择 |
| 确认提示 | 删除等危险操作确认 |

## 7. 事件处理

| 事件 | 处理 |
|------|------|
| 点击签到按钮 | 今日签到或撤销签到切换 |
| 双击单元格 | 进入编辑模式 |
| 右键点击行 | 显示上下文菜单 |
| 批量选择 | checkbox多选 |
| 批量操作 | 执行批量签到/修改 |
| 导出点击 | 生成并下载CSV |
| 导入完成 | 刷新列表显示 |
