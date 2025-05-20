## 一、开发语言与框架

| 类型    | 技术方案                                                | 理由                                  |
| ----- | --------------------------------------------------- | ----------------------------------- |
| 语言    | Swift                                               | Apple 原生开发语言，安全、高效、文档完善，Cursor 支持良好 |
| UI 框架 | SwiftUI                                             | 声明式 UI，简化代码结构，适合非专业开发者              |
| 状态管理  | `@State` / `@ObservedObject` / `@EnvironmentObject` | SwiftUI 原生状态管理，简单易上手                |
| 网络请求  | URLSession + `async/await` （或 `Alamofire`）          | 调用 Supabase REST API / 设计中间服务       |

---

## 二、后端推荐：Supabase (PostgreSQL + REST + Auth)

| 功能模块      | 技术栈                            | 描述                |
| --------- | ------------------------------ | ----------------- |
| 用户认证      | Supabase Auth                  | 支持邮箱、手机号、OAuth 登录 |
| 数据存储      | Supabase Database (PostgreSQL) | 数据结构明确，可 SQL/定时分析 |
| 云函数       | Supabase Edge Functions        | 自定义后端逻辑，用于计算、联合查询 |
| 图片/文件存储   | Supabase Storage               | 云存储，用于输出报表/图片上传   |
| 数据同步      | Swift + 本地缓存 + Supabase        | 支持无线操作，联网后同步      |
| 推送提醒 (扩展) | iOS 本地通知 + Edge Function       | 支持时段提醒，后期增加推送服务   |

> 如需支持中国大陆访问，可后期部署转发 API Gateway / Cloudflare Tunnel 实现网络兼容

---

## 三、图表可视化方案

| 图表类型      | 推荐技术                   | 用途                |
| --------- | ---------------------- | ----------------- |
| 折线图 / 柱状图 | Charts（iOS 第三方库）       | 展示断食时长、热量差、体重趋势等  |
| 饮图 / 堆叠图  | Charts / SwiftUI Shape | 展示萤养比例、碳循环执行比例    |
| 数据交互      | 手势滑动 + tooltip         | 支持用户查看具体数值，提升交互体验 |

---

## 四、数据结构映射（Supabase 数据表）

| 表名                 | 描述                 |
| ------------------ | ------------------ |
| `users`            | 用户个人信息，如身高、体重、断食设置 |
| `fasting_records`  | 断食记录，包含开始/结束时间、时长等 |
| `meal_records`     | 饮食记录，包含热量、碳水化合物等   |
| `activity_records` | 运动记录，包括类型、耗能、时长    |
| `weight_records`   | 体重记录，用于趋势图生成       |
| `plan_configs`     | 用户设置的碳循环、断食、运动计划   |
| `notifications`    | 本地提醒或用户提醒设置        |

所有数据表建议包含至少三个字段：`user_id`，`date`，`created_at`，并建立复合索引便于查询

---

## 五、页面结构与模块拆分（Cursor 中使用）

| 页面模块    | 文件建议命名                     | 功能描述                         |
| ------- | -------------------------- | ---------------------------- |
| 首页（今日）  | `TodayView.swift`          | 汇总显示断食状态、热量差、碳循环状态、运动计划完成情况等 |
| 记录页     | `RecordView.swift`         | 四个 tab：体重、断食、饮食、运动；支持增删改查    |
| 数据页     | `DataView.swift`           | 展示趋势图表，支持切换数据视图和时间周期         |
| 我的页     | `MyProfileView.swift`      | 个人资料、计划设置、语言切换等              |
| 添加饮食    | `AddMealView.swift`        | 表单录入每餐饮食信息，支持单位转换            |
| 开始/结束断食 | `FastingControlView.swift` | 快捷计时/手动补录                    |
| 添加运动    | `AddActivityView.swift`    | 录入运动信息，区分有氧与无氧类型             |
| 添加体重    | `AddWeightView.swift`      | 输入每日体重，触发趋势图更新               |
| 设置计划    | `PlanSettingsView.swift`   | 设置碳循环与锻炼周期、高碳日选择等            |
| 目标设定    | `GoalSettingView.swift`    | 设置目标体重、目标热量差等                |

---

## 六、Cursor Prompt 使用范例

```swift
// 生成“今日”页：汇总展示断食计时器、热量差、碳循环提醒、体重趋势
Create a SwiftUI view named TodayView.
It should include:
- A calorie summary (e.g., Intake: 1800 / Expenditure: 2300 / Deficit: -500)
- A real-time fasting timer with countdown toward 16 hours
- A message like "Today is a low-carb day (<50g), have you reached the goal?"
- A button bar with "+ Add Meal", "+ Start/End Fasting", "+ Add Activity"
```

```swift
// 生成记录页：支持四个tab记录体重、断食、饮食、运动
Create a SwiftUI tabbed view with four sections: Weight, Fasting, Meal, and Activity.
Each section should show a summary and a list of past records. Add buttons to add or edit entries.
```

---

## 七、开发部署流程

| 阶段    | 操作                                              | 工具                  |
| ----- | ----------------------------------------------- | ------------------- |
| 环境准备  | 注册 Apple 开发者账号 + Supabase 账号                    | Apple + Supabase    |
| UI 开发 | 使用 Cursor 创建 SwiftUI 页面                         | Cursor IDE          |
| 后端配置  | 初始化 Supabase 项目，创建数据表，启用 Auth                   | Supabase 控制台        |
| 本地测试  | Xcode 运行 + iOS 模拟器调试                            | Xcode + Simulator   |
| 功能联调  | 使用 URLSession 调用 Supabase REST 或 Edge Functions | Swift + async/await |
| 上架准备  | 填写隐私条款、配置证书、打包 IPA                              |                     |
