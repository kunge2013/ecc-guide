# Bug 修复 - 内存泄漏

这个场景展示了如何使用 Claude Code 识别、分析和修复内存泄漏问题。

## 🎯 场景概述

### 适用情况

- 应用内存使用持续增长
- 性能逐渐下降
- 偶发 Out of Memory 错误
- 长时间运行的应用

### 核心挑战

- 定位泄漏源头
- 区分真正的泄漏和正常缓存
- 验证修复效果
- 避免引入新问题

## 📋 使用场景

### 场景 1: 事件监听器泄漏

**问题：** 事件监听器未正确清理，导致内存泄漏

**症状：**
- 组件卸载后仍在执行
- 内存使用持续增长
- 性能下降

**修复步骤：**
1. 使用 Chrome DevTools Memory Profiler 识别泄漏
2. 分析 Retained Objects
3. 在组件卸载时移除监听器
4. 验证修复效果

---

### 场景 2: 闭包引用泄漏

**问题：** 闭包意外持有大对象引用

**症状：**
- 大对象无法被垃圾回收
- 内存占用异常高
- GC 频率增加

**修复步骤：**
1. 分析闭包引用链
2. 识别不必要的大对象引用
3. 使用 WeakMap 或 WeakRef
4. 重构代码避免闭包

---

### 场景 3: 定时器未清理

**问题：** setInterval 或 setTimeout 未清理

**症状：**
- 定时器持续执行
- 回调函数累积
- CPU 使用率高

**修复步骤：**
1. 使用 Agent 定位定时器泄漏
2. 在适当的生命周期清理定时器
3. 使用 clearTimeout/clearInterval
4. 添加清理逻辑到所有异步操作

## 🚀 典型工作流程

### 步骤 1: 问题诊断

```bash
# 使用 Agent 分析内存问题
Agent(planner, """
应用出现内存泄漏问题：
- 运行 24 小时后内存占用从 100MB 增长到 2GB
- CPU 使用率逐渐增加
- 偶发 Out of Memory 错误

请分析可能的原因并提供诊断方案：
1. 检查内存快照
2. 分析内存增长趋势
3. 定位可能的泄漏点
4. 提供修复建议
""")
```

### 步骤 2: 定位泄漏源

```bash
# 使用 Code Reviewer Agent 检查代码
Agent(code-reviewer, """
检查以下代码中的内存泄漏风险：

1. 事件监听器是否正确清理
2. 定时器是否正确清理
3. 闭包是否持有不必要的引用
4. 缓存是否有清理机制

重点检查：
- src/components/
- src/services/
- src/utils/
""")
```

### 步骤 3: 修复验证

```bash
# 修复后验证
Agent(e2eater, """
测试内存泄漏修复：
1. 运行应用 1 小时
2. 监控内存使用
3. 执行可能导致泄漏的操作
4. 验证内存是否稳定
""")
```

## 📊 诊断工具

### Chrome DevTools

```javascript
// Memory Profiler 使用
// 1. 打开 DevTools
// 2. 切换到 Memory 标签
// 3. 点击 "Take heap snapshot"
// 4. 执行可能导致泄漏的操作
// 5. 再次 "Take heap snapshot"
// 6. 对比两个快照，查看 "Retained Objects"

// 分析 Retained Objects
// 查找：
// - Detached DOM nodes
// - Event listeners
// - Closures
// - Timers
```

### Node.js 内存分析

```javascript
// file: memory-profiler.js

const v8 = require('v8');
const fs = require('fs');

function takeHeapSnapshot() {
  const snapshot = v8.getHeapSnapshot();
  const filename = `heap-snapshot-${Date.now()}.heapsnapshot`;
  fs.writeFileSync(filename, JSON.stringify(snapshot));
  console.log(`Heap snapshot saved to ${filename}`);
}

function getMemoryStats() {
  const stats = v8.getHeapStatistics();
  return {
    used_heap_size: stats.used_heap_size / 1024 / 1024 + ' MB',
    total_heap_size: stats.total_heap_size / 1024 / 1024 + ' MB',
    heap_size_limit: stats.heap_size_limit / 1024 / 1024 + ' MB'
  };
}

// 定期采集内存数据
setInterval(() => {
  console.log('Memory Stats:', getMemoryStats());

  // 内存使用超过阈值时采集快照
  const stats = v8.getHeapStatistics();
  if (stats.used_heap_size > stats.total_heap_size * 0.9) {
    takeHeapSnapshot();
  }
}, 60000); // 每分钟
```

## 🐛 Python 内存分析

```python
# memory_profiler.py

import psutil
import gc
import time
from memory_profiler import profile

def get_memory_stats():
    process = psutil.Process()
    return {
        'rss': process.memory_info().rss / 1024 / 1024,  # MB
        'vms': process.memory_info().vms / 1024 / 1024,  # MB
        'percent': process.memory_percent(),
        'num_objects': len(gc.get_objects())
    }

@profile
def monitor_memory(interval=60, threshold=500):
    """监控内存使用"""
    print("开始监控内存...")

    while True:
        stats = get_memory_stats()
        print(f"内存使用: {stats['rss']:.2f} MB")

        # 内存超过阈值时分析
        if stats['rss'] > threshold:
            print(f"⚠️ 内存使用超过阈值: {stats['rss']:.2f} MB")
            analyze_memory()

        time.sleep(interval)

def analyze_memory():
    """分析内存中的对象"""
    all_objects = gc.get_objects()
    print(f"总对象数: {len(all_objects)}")

    # 统计对象类型
    type_counts = {}
    for obj in all_objects:
        obj_type = type(obj).__name__
        type_counts[obj_type] = type_counts.get(obj_type, 0) + 1

    # 按数量排序
    sorted_types = sorted(type_counts.items(), key=lambda x: x[1], reverse=True)

    print("对象类型统计（前 10）:")
    for obj_type, count in sorted_types[:10]:
        print(f"  {obj_type}: {count}")

if __name__ == '__main__':
    monitor_memory()
```

## 💡 最佳实践

### 1. 事件监听器清理

```javascript
// ✅ 正确的监听器清理
class MyComponent {
  constructor() {
    this.listeners = [];
  }

  addEventListener(element, event, handler) {
    element.addEventListener(event, handler);
    this.listeners.push({ element, event, handler });
  }

  cleanup() {
    this.listeners.forEach(({ element, event, handler }) => {
      element.removeEventListener(event, handler);
    });
    this.listeners = [];
  }
}

// 使用
const component = new MyComponent();
component.addEventListener(document, 'click', handleClick);
// 使用完毕后
component.cleanup();
```

### 2. 定时器清理

```javascript
// ✅ 正确的定时器清理
class TimerManager {
  constructor() {
    this.timers = [];
  }

  setTimeout(callback, delay) {
    const timerId = setTimeout(callback, delay);
    this.timers.push(timerId);
    return timerId;
  }

  setInterval(callback, interval) {
    const timerId = setInterval(callback, interval);
    this.timers.push(timerId);
    return timerId;
  }

  clearAll() {
    this.timers.forEach(timerId => {
      clearTimeout(timerId);
      clearInterval(timerId);
    });
    this.timers = [];
  }
}

// 使用
const timerManager = new TimerManager();
timerManager.setTimeout(() => console.log('Hello'), 1000);
// 清理时
timerManager.clearAll();
```

### 3. 避免闭包泄漏

```javascript
// ❌ 错误：闭包持有大对象
function processData(data) {
  // 假设 data 是一个大对象
  return function() {
    // 这个闭包持有了 data 的引用
    console.log(data.length);
  };
}

// ✅ 正确：只持有需要的数据
function processData(data) {
  const length = data.length; // 只提取需要的数据
  return function() {
    console.log(length); // 不再持有 data
  };
}

// ✅ 或者使用 WeakMap
const dataCache = new WeakMap();

function processData(data) {
  dataCache.set(data, { processed: false });

  return function() {
    const cached = dataCache.get(data);
    if (cached) {
      console.log(cached.processed);
    }
  };
}
```

## 🔗 相关资源

- [Planner Agent](../../02-agents/01-planner/) - 问题分析
- [Code Reviewer Agent](../../02-agents/03-code-reviewer/) - 代码审查
- [Python Reviewer](../../02-agents/05-language-reviewers/) - Python 内存分析

---

**提示：** 内存泄漏诊断需要系统化的方法。使用内存分析工具和 Agent 协作可以快速定位和修复问题。
