# Design Document: Floor Climbing Mode

## Overview

爬楼模式是 Linux.do 论坛助手的一个新功能模块，专门用于智能浏览运营反馈板块的高回复数帖子。该模式通过检测回复的阅读状态指示器（蓝色圆点），确保每条回复被完全阅读后才继续滚动，从而有效提升阅读量统计。

核心特性：
- 固定爬取运营反馈板块（https://linux.do/c/feedback/2）
- 智能检测阅读状态指示器（`.read-state.read` 内的蓝色圆点）
- 支持无限滚动加载（帖子列表和回复列表）
- 获取并验证阅读进度变化
- 防风控机制（随机延迟、模拟真实行为）

## Architecture

爬楼模式将作为 `Bot` 类的扩展功能实现，采用模块化设计。GUI 版本使用 Tkinter 界面，Bot 类负责浏览器自动化操作。

```
Bot (现有类)
├── 现有方法
│   ├── start()                     # 启动浏览器
│   ├── check_login()               # 检查登录
│   ├── get_level_info()            # 获取等级信息
│   └── run_session()               # 运行浏览会话
│
└── 新增爬楼模式方法
    ├── get_reading_progress()      # 获取阅读进度
    ├── fetch_feedback_topics()     # 获取运营反馈板块帖子
    ├── climb_topic()               # 爬楼单个帖子
    ├── detect_read_state()         # 检测阅读状态指示器
    ├── scroll_with_indicator()     # 带指示器检测的智能滚动
    ├── handle_infinite_scroll()    # 处理无限滚动
    ├── verify_progress()           # 验证进度变化
    └── run_floor_climbing_session() # 运行爬楼会话

GUI (现有类)
├── 现有界面元素
│   ├── 用户信息面板
│   ├── 升级进度面板
│   ├── 板块选择面板
│   ├── 日志面板
│   └── 控制按钮
│
└── 新增爬楼模式界面
    ├── 爬楼模式开关（Checkbutton）
    ├── 爬楼配置面板
    │   ├── 帖子数量配置
    │   ├── 滚动速度配置
    │   └── 检测间隔配置
    └── 爬楼统计显示
```

### 集成方式

爬楼模式将作为 `Bot` 的一个可选运行模式，通过 GUI 界面控制：

```python
# 在 CFG 中添加爬楼模式配置
CFG = {
    # ... 现有配置 ...
    "floor_climbing_enabled": False,
    "floor_climbing_topics": (3, 5),
    "floor_climbing_indicator_interval": (0.5, 1.0),
    "floor_climbing_reply_stay": (2, 5),
    "floor_climbing_scroll_distance": (300, 600),
}

# 在 Bot 类中添加爬楼模式入口
def run_floor_climbing_session(s):
    """运行爬楼模式会话"""
    pass

# 在 GUI 类中添加爬楼模式控制
def toggle_floor_climbing(s):
    """切换爬楼模式"""
    pass
```

## Components and Interfaces

### 1. Bot 类扩展

在现有的 `Bot` 类中添加爬楼模式方法：

```python
class Bot:
    """论坛自动化操作类（现有类）"""
    
    def __init__(s, cfg, cats, lg, update_info=None, update_progress=None):
        # ... 现有初始化代码 ...
        s.floor_climbing_stats = {
            "topics_climbed": 0,
            "replies_read": 0,
            "indicators_detected": 0,
            "progress_before": {},
            "progress_after": {},
            "errors": 0
        }
    
    # ========== 新增爬楼模式方法 ==========
    
    def get_reading_progress(s):
        """
        获取用户阅读进度
        
        Returns:
            dict: 包含已读话题数、阅读时长等信息
        """
        pass
    
    def fetch_feedback_topics(s, max_topics=50):
        """
        获取运营反馈板块的帖子列表
        
        Args:
            max_topics: 最多获取的帖子数量
            
        Returns:
            list: 帖子列表，每个帖子包含 url, title, reply_count
        """
        pass
    
    def detect_read_state(s):
        """
        检测当前可见回复的阅读状态指示器
        
        Returns:
            list: 指示器信息列表，包含 index, visible, has_indicator
        """
        pass
    
    def scroll_with_indicator(s):
        """
        带阅读状态指示器检测的智能滚动
        
        Returns:
            int: 已读回复数量
        """
        pass
    
    def handle_infinite_scroll(s, scroll_type="topic"):
        """
        处理无限滚动加载
        
        Args:
            scroll_type: "topic" 或 "reply"
            
        Returns:
            bool: 是否成功加载新内容
        """
        pass
    
    def climb_topic(s, topic_url, topic_title):
        """
        爬楼单个帖子
        
        Args:
            topic_url: 帖子 URL
            topic_title: 帖子标题
            
        Returns:
            bool: 是否成功完成爬楼
        """
        pass
    
    def verify_progress(s):
        """
        验证阅读进度变化
        
        Returns:
            dict: 进度差异信息
        """
        pass
    
    def run_floor_climbing_session(s):
        """
        运行一次爬楼会话
        
        Returns:
            bool: 是否成功完成
        """
        pass
```

### 2. GUI 类扩展

在现有的 `GUI` 类中添加爬楼模式界面元素：

```python
class GUI:
    """图形界面类（现有类）"""
    
    def __init__(s):
        # ... 现有初始化代码 ...
        s.floor_climbing_enabled = tk.BooleanVar(value=False)
        s.floor_climbing_topics_min = tk.IntVar(value=3)
        s.floor_climbing_topics_max = tk.IntVar(value=5)
    
    def create_floor_climbing_panel(s):
        """创建爬楼模式配置面板"""
        frame = ttk.LabelFrame(s.rt, text="爬楼模式", padding=10)
        
        # 爬楼模式开关
        ttk.Checkbutton(
            frame,
            text="启用爬楼模式（运营反馈板块）",
            variable=s.floor_climbing_enabled
        ).pack(anchor="w")
        
        # 帖子数量配置
        topics_frame = ttk.Frame(frame)
        topics_frame.pack(fill="x", pady=5)
        ttk.Label(topics_frame, text="爬楼帖子数:").pack(side="left")
        ttk.Spinbox(
            topics_frame,
            from_=1, to=10,
            textvariable=s.floor_climbing_topics_min,
            width=5
        ).pack(side="left", padx=5)
        ttk.Label(topics_frame, text="-").pack(side="left")
        ttk.Spinbox(
            topics_frame,
            from_=1, to=10,
            textvariable=s.floor_climbing_topics_max,
            width=5
        ).pack(side="left", padx=5)
        
        return frame
    
    def on_start_click(s):
        """开始按钮点击事件（修改）"""
        # ... 现有代码 ...
        
        # 检查是否启用爬楼模式
        if s.floor_climbing_enabled.get():
            # 运行爬楼模式
            threading.Thread(target=s.run_floor_climbing, daemon=True).start()
        else:
            # 运行普通模式
            threading.Thread(target=s.run_normal, daemon=True).start()
    
    def run_floor_climbing(s):
        """运行爬楼模式线程"""
        pass
```

### 3. 配置扩展

在 `CFG` 字典中添加爬楼模式配置：

```python
CFG = {
    # ... 现有配置 ...
    
    # 爬楼模式配置
    "floor_climbing_enabled": False,
    "floor_climbing_feedback_url": "https://linux.do/c/feedback/2",
    "floor_climbing_topics_min": 3,
    "floor_climbing_topics_max": 5,
    "floor_climbing_max_topics_to_fetch": 50,
    "floor_climbing_indicator_interval_min": 0.5,
    "floor_climbing_indicator_interval_max": 1.0,
    "floor_climbing_reply_stay_min": 2,
    "floor_climbing_reply_stay_max": 5,
    "floor_climbing_scroll_distance_min": 300,
    "floor_climbing_scroll_distance_max": 600,
    "floor_climbing_infinite_scroll_wait": 3,
    "floor_climbing_max_retries": 3,
}
```

### 4. 状态文件扩展

在状态 JSON 文件中添加爬楼模式记录：

```json
{
  "visited_topics": [...],
  "liked_posts": [...],
  "replied_topics": [...],
  "climbed_topics": [],
  "floor_climbing_stats": {
    "total_topics_climbed": 0,
    "total_replies_read": 0,
    "last_climb_time": null
  },
  "last_run": "2025-01-20T12:00:00",
  "total_stats": {...}
}
```

## Data Models

### Topic 数据结构

```python
# 使用字典表示帖子
topic = {
    "url": str,           # 帖子 URL
    "title": str,         # 帖子标题
    "reply_count": int    # 回复数量
}
```

### ReadingProgress 数据结构

```python
# 使用字典表示阅读进度
progress = {
    "topics_read": int,      # 已读话题数
    "reading_time": int,     # 阅读时长（分钟）
    "likes_given": int,      # 点赞数
    "replies_made": int,     # 回复数
    "timestamp": str         # 时间戳
}
```

### ReadStateIndicator 数据结构

```python
# 使用字典表示阅读状态指示器
indicator = {
    "index": int,            # 回复索引
    "visible": bool,         # 是否可见
    "has_indicator": bool,   # 是否有指示器
    "element_selector": str  # 元素选择器
}
```

### ClimbingStats 数据结构

```python
# 使用字典表示爬楼统计
stats = {
    "topics_climbed": int,           # 爬楼帖子数
    "replies_read": int,             # 已读回复数
    "indicators_detected": int,      # 检测到的指示器数
    "progress_before": dict,         # 爬楼前进度
    "progress_after": dict,          # 爬楼后进度
    "errors": int,                   # 错误次数
    "start_time": str,               # 开始时间
    "end_time": str                  # 结束时间
}
```

## Correctness Properties

*属性是系统应该在所有有效执行中保持为真的特征或行为——本质上是关于系统应该做什么的形式化陈述。属性作为人类可读规范和机器可验证正确性保证之间的桥梁。*

### Property 1: 帖子列表获取完整性

*对于任何* 运营反馈板块的访问，如果触发无限滚动直到没有新内容加载，则获取的帖子列表应包含板块中所有可用的帖子（或达到配置的最大数量）。

**Validates: Requirements 1.2, 1.4, 1.5**

### Property 2: 阅读状态指示器检测准确性

*对于任何* 包含阅读状态指示器的回复，当指示器的 DOM 元素 `.read-state.read` 存在且包含蓝色圆点 SVG 时，检测函数应返回指示器存在状态；当元素不存在或 SVG 不可见时，应返回指示器消失状态。

**Validates: Requirements 4.1, 4.2, 4.3**

### Property 3: 智能滚动等待策略

*对于任何* 检测到阅读状态指示器的回复，系统应暂停滚动并等待指示器消失后才继续滚动到下一条回复。

**Validates: Requirements 3.4, 3.5**

### Property 4: 无限滚动触发正确性

*对于任何* 滚动到页面底部的操作，如果页面支持无限滚动且有更多内容可加载，则系统应等待新内容加载完成后再继续操作。

**Validates: Requirements 7.1, 7.2, 7.4**

### Property 5: 进度验证一致性

*对于任何* 爬楼会话，如果成功爬楼了 N 个帖子，则会话后的阅读进度应大于会话前的阅读进度。

**Validates: Requirements 5.1, 5.2, 5.3**

### Property 6: 已爬楼帖子过滤

*对于任何* 帖子列表，如果某个帖子的 URL 已存在于已爬楼记录中，则该帖子应被过滤掉，不再进行爬楼。

**Validates: Requirements 1.6**

### Property 7: 错误恢复机制

*对于任何* 爬楼过程中的错误（页面加载超时、指示器检测失败等），系统应记录错误日志并继续处理下一个帖子，而不是终止整个会话。

**Validates: Requirements 8.1, 8.2, 8.5**

### Property 8: 防风控随机化

*对于任何* 两个连续的爬楼操作，它们之间的等待时间、滚动速度、停留时间应该是随机的，且在配置的范围内。

**Validates: Requirements 10.1, 10.2**

## Error Handling

### 错误类型和处理策略

| 错误类型 | 处理策略 | 重试次数 |
|---------|---------|---------|
| 页面加载超时 | 跳过当前帖子，继续下一个 | 1 |
| 阅读状态指示器检测失败 | 使用默认等待时间策略 | 0 |
| 无限滚动加载失败 | 判定内容已全部加载，继续 | 3 |
| 网络连接中断 | 暂停并等待网络恢复 | 5 |
| 登录状态失效 | 提示用户重新登录，终止会话 | 0 |
| 进度获取失败 | 记录警告，继续爬楼 | 2 |

### 错误日志格式

```python
{
    "timestamp": "2025-01-20 12:00:00",
    "error_type": "page_load_timeout",
    "topic_url": "https://linux.do/t/topic/12345",
    "message": "页面加载超时",
    "retry_count": 1,
    "action_taken": "skip_topic"
}
```

### 连续失败处理

当连续失败次数达到配置的最大重试次数（默认 3 次）时：
1. 记录严重错误日志
2. 停止当前爬楼会话
3. 保存已完成的统计数据
4. 提示用户检查网络或登录状态

## Testing Strategy

### 单元测试

使用 Python 的 `unittest` 框架进行单元测试：

1. **阅读状态指示器检测测试**
   - 测试指示器存在时的检测
   - 测试指示器消失时的检测
   - 测试 DOM 结构变化时的容错性

2. **无限滚动处理测试**
   - 测试滚动到底部触发加载
   - 测试加载完成的判定
   - 测试连续无新内容的判定

3. **进度计算测试**
   - 测试进度差异计算的准确性
   - 测试进度数据的序列化和反序列化

4. **状态管理测试**
   - 测试已爬楼帖子的记录和查询
   - 测试状态文件的保存和加载

### 属性测试

使用 `hypothesis` 库进行属性测试（每个测试至少 100 次迭代）：

1. **Property 1: 帖子列表获取完整性**
   ```python
   # Feature: floor-climbing-mode, Property 1: 帖子列表获取完整性
   @given(max_topics=st.integers(min_value=1, max_value=100))
   def test_fetch_topics_completeness(max_topics):
       # 生成随机的帖子数据
       # 验证获取的帖子数量不超过 max_topics
       # 验证所有帖子都有 url, title, reply_count
   ```

2. **Property 2: 阅读状态指示器检测准确性**
   ```python
   # Feature: floor-climbing-mode, Property 2: 阅读状态指示器检测准确性
   @given(has_indicator=st.booleans(), is_visible=st.booleans())
   def test_read_state_detection_accuracy(has_indicator, is_visible):
       # 模拟不同的指示器状态
       # 验证检测结果与实际状态一致
   ```

3. **Property 5: 进度验证一致性**
   ```python
   # Feature: floor-climbing-mode, Property 5: 进度验证一致性
   @given(topics_climbed=st.integers(min_value=1, max_value=10))
   def test_progress_verification_consistency(topics_climbed):
       # 模拟爬楼 N 个帖子
       # 验证进度增长 >= 0
   ```

4. **Property 6: 已爬楼帖子过滤**
   ```python
   # Feature: floor-climbing-mode, Property 6: 已爬楼帖子过滤
   @given(topics=st.lists(st.text(min_size=10), min_size=5, max_size=20))
   def test_climbed_topics_filtering(topics):
       # 随机标记部分帖子为已爬楼
       # 验证过滤后的列表不包含已爬楼帖子
   ```

5. **Property 8: 防风控随机化**
   ```python
   # Feature: floor-climbing-mode, Property 8: 防风控随机化
   @given(num_operations=st.integers(min_value=2, max_value=10))
   def test_anti_detection_randomization(num_operations):
       # 生成多次操作的等待时间
       # 验证所有等待时间都在配置范围内
       # 验证等待时间不完全相同（随机性）
   ```

### 集成测试

1. **完整爬楼流程测试**
   - 启动浏览器
   - 登录账号
   - 获取初始进度
   - 执行爬楼
   - 验证进度变化
   - 检查日志和统计数据

2. **错误恢复测试**
   - 模拟网络中断
   - 模拟页面加载失败
   - 验证系统能够恢复并继续

3. **防风控测试**
   - 运行多次爬楼会话
   - 验证操作间隔的随机性
   - 验证未触发异常检测

### 测试数据

使用模拟数据进行测试：

```python
MOCK_TOPICS = [
    {"url": "https://linux.do/t/topic/1", "title": "测试帖子1", "reply_count": 50},
    {"url": "https://linux.do/t/topic/2", "title": "测试帖子2", "reply_count": 30},
    {"url": "https://linux.do/t/topic/3", "title": "测试帖子3", "reply_count": 100},
]

MOCK_PROGRESS_BEFORE = {
    "topics_read": 100,
    "reading_time": 500,
    "likes_given": 50,
    "replies_made": 10
}

MOCK_PROGRESS_AFTER = {
    "topics_read": 103,
    "reading_time": 530,
    "likes_given": 50,
    "replies_made": 10
}
```

### 测试覆盖率目标

- 代码覆盖率：>= 80%
- 分支覆盖率：>= 70%
- 属性测试迭代次数：>= 100 次/属性
