# Requirements Document

## Introduction

本文档定义了 Linux.do 论坛助手的"爬楼模式"功能需求。爬楼模式专门针对"运营反馈"板块（https://linux.do/c/feedback/2）的官方帖子进行智能阅读。该板块的帖子回复数较多，适合爬楼。根据 Linux.do 的规则，爬楼（浏览回复）也计入阅读量，但需要等待每条回复的蓝色圆点（read-state）消失后才算有效阅读。

## Glossary

- **System**: 爬楼模式系统
- **Topic**: 论坛帖子
- **Reply**: 帖子的回复内容
- **Read_State_Indicator**: 回复的阅读状态指示器，DOM 元素为 `<div class="read-state read">` 内的蓝色圆点 SVG 图标
- **Feedback_Category**: 运营反馈板块，URL 为 https://linux.do/c/feedback/2
- **Reading_Progress**: 阅读进度，包括已读话题数、阅读时长等统计数据
- **Infinite_Scroll**: 无限滚动，帖子列表和回复列表滚动到底部时自动加载更多内容
- **Scroll_Speed**: 滚动速度，控制页面滚动的快慢

## Requirements

### Requirement 1: 获取运营反馈板块帖子列表

**User Story:** 作为用户，我希望系统能自动获取运营反馈板块的所有帖子，以便进行爬楼阅读。

#### Acceptance Criteria

1. THE System SHALL 访问运营反馈板块 URL（https://linux.do/c/feedback/2）
2. WHEN 页面加载完成 THEN THE System SHALL 获取当前可见的所有帖子信息
3. THE System SHALL 记录每个帖子的标题、URL、回复数
4. WHEN 滚动到页面底部 THEN THE System SHALL 触发无限滚动加载更多帖子
5. THE System SHALL 持续滚动直到加载所有可用帖子或达到配置的最大帖子数
6. THE System SHALL 过滤已爬楼过的帖子（基于本地记录）
7. WHEN 没有新帖子可爬 THEN THE System SHALL 提示用户并结束

### Requirement 2: 获取初始阅读进度

**User Story:** 作为用户，我希望在开始爬楼前获取当前的阅读进度，以便后续验证爬楼效果。

#### Acceptance Criteria

1. WHEN 用户启动爬楼模式 THEN THE System SHALL 访问用户统计页面获取初始阅读进度
2. THE System SHALL 记录以下初始数据：已读话题数、阅读时长、点赞数、回复数
3. THE System SHALL 将初始进度数据保存到本地文件
4. WHEN 无法获取进度数据 THEN THE System SHALL 记录错误日志并提示用户
5. THE System SHALL 在日志中显示初始进度的详细信息

### Requirement 3: 智能爬楼滚动

**User Story:** 作为用户，我希望系统能智能地滚动浏览回复，确保每条回复的阅读状态指示器消失后才继续，以保证阅读有效性。

#### Acceptance Criteria

1. WHEN 进入帖子页面 THEN THE System SHALL 等待页面完全加载
2. WHEN 检测到回复区域 THEN THE System SHALL 开始滚动浏览
3. WHILE 滚动过程中 THE System SHALL 监测当前可见回复的阅读状态指示器
4. WHEN 检测到阅读状态指示器（蓝色圆点）存在 THEN THE System SHALL 暂停滚动并等待其消失
5. WHEN 阅读状态指示器消失 THEN THE System SHALL 继续滚动到下一条回复
6. THE System SHALL 支持配置指示器检测间隔时间（默认 0.5-1 秒）
7. WHEN 滚动到页面底部 THEN THE System SHALL 触发无限滚动加载更多回复
8. WHEN 所有回复加载完毕 THEN THE System SHALL 标记该帖子爬楼完成
9. THE System SHALL 在每条回复停留时间为 2-5 秒的随机值

### Requirement 4: 阅读状态指示器检测

**User Story:** 作为用户，我希望系统能准确检测阅读状态指示器的存在和消失状态，确保阅读被正确统计。

#### Acceptance Criteria

1. THE System SHALL 使用 CSS 选择器 `.read-state.read` 检测阅读状态指示器元素
2. WHEN 指示器元素存在且包含蓝色圆点 SVG THEN THE System SHALL 返回指示器存在状态
3. WHEN 指示器元素不存在或 SVG 不可见 THEN THE System SHALL 返回指示器消失状态
4. THE System SHALL 检测 SVG 元素的 `use` 标签是否引用 `#circle`
5. WHEN 无法检测指示器状态 THEN THE System SHALL 使用默认等待时间策略
6. THE System SHALL 支持检测多个回复的指示器状态

### Requirement 5: 验证阅读进度

**User Story:** 作为用户，我希望在爬楼结束后验证阅读进度是否增加，以确认爬楼模式的有效性。

#### Acceptance Criteria

1. WHEN 爬楼任务完成 THEN THE System SHALL 再次获取用户阅读进度
2. THE System SHALL 计算进度差异：已读话题增量、阅读时长增量
3. WHEN 进度有增长 THEN THE System SHALL 在日志中显示增长数据
4. WHEN 进度无增长 THEN THE System SHALL 记录警告日志并提示可能的问题
5. THE System SHALL 生成爬楼报告，包含：爬楼帖子数、总回复数、进度变化

### Requirement 6: 爬楼模式配置

**User Story:** 作为用户，我希望能够配置爬楼模式的各项参数，以适应不同的使用场景。

#### Acceptance Criteria

1. THE System SHALL 固定使用运营反馈板块（https://linux.do/c/feedback/2）
2. THE System SHALL 支持配置每次爬楼的帖子数量（默认 3-5）
3. THE System SHALL 支持配置滚动速度（慢速/中速/快速）
4. THE System SHALL 支持配置指示器检测间隔（默认 0.5-1 秒）
5. THE System SHALL 支持配置每条回复停留时间范围（默认 2-5 秒）
6. THE System SHALL 支持启用/禁用爬楼模式
7. WHEN 配置参数无效 THEN THE System SHALL 使用默认值并记录警告

### Requirement 7: 无限滚动处理

**User Story:** 作为用户，我希望系统能正确处理帖子列表和回复列表的无限滚动，确保加载所有内容。

#### Acceptance Criteria

1. WHEN 在帖子列表页滚动到底部 THEN THE System SHALL 等待新帖子加载
2. WHEN 在帖子详情页滚动到底部 THEN THE System SHALL 等待新回复加载
3. THE System SHALL 检测页面是否正在加载新内容
4. WHEN 检测到加载指示器 THEN THE System SHALL 暂停滚动并等待加载完成
5. WHEN 连续 3 次滚动到底部无新内容加载 THEN THE System SHALL 判定内容已全部加载
6. THE System SHALL 在触发无限滚动后等待 2-3 秒

### Requirement 8: 错误处理与恢复

**User Story:** 作为用户，我希望系统在遇到错误时能够妥善处理，不影响整体运行。

#### Acceptance Criteria

1. WHEN 页面加载超时 THEN THE System SHALL 跳过当前帖子并继续下一个
2. WHEN 检测蓝点失败 THEN THE System SHALL 使用默认等待策略继续
3. WHEN 网络连接中断 THEN THE System SHALL 暂停并等待网络恢复
4. WHEN 登录状态失效 THEN THE System SHALL 提示用户重新登录
5. THE System SHALL 记录所有错误到日志文件
6. THE System SHALL 在连续失败 3 次后停止爬楼模式

### Requirement 9: 日志与统计

**User Story:** 作为用户，我希望看到详细的爬楼日志和统计信息，了解爬楼效果。

#### Acceptance Criteria

1. THE System SHALL 记录每个爬楼帖子的详细信息：标题、回复数、耗时
2. THE System SHALL 实时显示当前爬楼进度
3. THE System SHALL 在爬楼完成后生成统计报告
4. THE System SHALL 统计报告包含：总耗时、爬楼帖子数、总回复数、进度增量
5. THE System SHALL 将统计数据保存到本地文件供后续分析

### Requirement 10: 防风控机制

**User Story:** 作为用户，我希望爬楼模式具有防风控机制，避免被系统检测为异常行为。

#### Acceptance Criteria

1. THE System SHALL 在帖子之间添加随机等待时间（3-8 秒）
2. THE System SHALL 随机化滚动速度和停留时间
3. THE System SHALL 限制单次会话的爬楼帖子数量（最多 10 个）
4. THE System SHALL 模拟真实用户的鼠标移动和页面交互
5. WHEN 检测到异常响应 THEN THE System SHALL 暂停并延长等待时间
