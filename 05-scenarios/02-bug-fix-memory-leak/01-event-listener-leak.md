# 使用场景 1: 事件监听器内存泄漏

## 问题描述

应用在运行 24 小时后内存占用从 100MB 增长到 2GB，最终 Out of Memory 崩溃。

## 诊断过程

### 步骤 1: 使用 Planner Agent 分析

```bash
Agent(planner, """
应用出现严重的内存泄漏问题：
- 启动时内存：100MB
- 运行 24 小时后：2GB
- 偶发 Out of Memory 错误

请分析可能的原因：
1. 事件监听器泄漏
2. 定时器泄漏
3. 闭包引用泄漏
4. 缓存无限增长

并提供诊断方案和修复建议。
""")
```

**Planner 输出：**

```markdown
# 内存泄漏诊断方案

## 可能原因分析

### 1. 事件监听器泄漏（可能性：高）
**症状：** DOM 节点移除后，事件监听器仍在执行
**特征：** 事件监听器数量持续增长
**常见位置：** React 组件、事件绑定

### 2. 定时器泄漏（可能性：中）
**症状：** setInterval/setTimeout 未清理
**特征：** 活动定时器数量增长
**常见位置：** 轮询、动画、延迟执行

### 3. 闭包引用泄漏（可能性：中）
**症状：** 闭包意外持有大对象引用
**特征：** 对象无法被垃圾回收
**常见位置：** 回调函数、事件处理

### 4. 缓存无限增长（可能性：低）
**症状：** 缓存条目持续增加
**特征：** 数据结构大小增长
**常见位置：** 数据缓存、会话存储

## 诊断步骤

### 1. 采集内存快照
```javascript
// 在 DevTools Memory 标签中
// 1. 点击 "Take heap snapshot" (基准)
// 2. 执行可能导致泄漏的操作（切换页面 100 次）
// 3. 再次点击 "Take heap snapshot"
// 4. 对比两个快照
```

### 2. 分析 Retained Objects
```javascript
// 检查：
// - Detached DOM nodes
// - Event listeners
// - Timers
// - 闭包
```

### 3. 定位代码位置
```javascript
// 查找：
// - addEventListener 但没有 removeEventListener
// - setInterval/setTimeout 但没有 clearTimeout/clearInterval
// - 持有外部引用的闭包
```

## 修复建议

### 1. 事件监听器
- 在组件卸载时移除所有监听器
- 使用事件委托
- 使用 WeakMap 存储监听器引用

### 2. 定时器
- 在组件卸载时清理所有定时器
- 使用定时器管理器
- 避免嵌套定时器

### 3. 闭包
- 只闭包必要的数据
- 使用 WeakMap/WeakRef
- 避免在闭包中持有大对象

### 4. 缓存
- 设置最大大小
- 实现 LRU 淘汰策略
- 定期清理过期数据
```

### 步骤 2: 使用 Code Reviewer 检查代码

```bash
Agent(code-reviewer, """
检查以下代码中的内存泄漏风险：

重点检查：
1. 事件监听器是否正确清理
2. 定时器是否正确清理
3. 闭包是否持有不必要的引用
4. 缓存是否有清理机制

文件范围：
- src/components/
- src/services/
- src/utils/
""")
```

### 步骤 3: 修复验证

```bash
Agent(e2eater, """
测试内存泄漏修复：

1. 创建新页面
2. 执行可能的泄漏操作（切换页面 100 次）
3. 监控内存使用
4. 验证内存稳定
5. 检查没有泄漏

使用 Chrome DevTools Performance Memory 进行验证。
""")
```

## 修复实现

### 问题代码

```javascript
// file: src/components/DataGrid.js

class DataGrid {
  constructor(container, data) {
    this.container = container;
    this.data = data;
    this.listeners = [];

    this.render();
    this.attachEvents();
    this.startAutoRefresh();
  }

  render() {
    this.container.innerHTML = `
      <div class="datagrid">
        ${this.data.map(item => `
          <div class="row" data-id="${item.id}">
            ${item.name}
          </div>
        `).join('')}
      </div>
    `;
  }

  // ❌ 问题：事件监听器没有清理
  attachEvents() {
    const rows = this.container.querySelectorAll('.row');
    rows.forEach(row => {
      const handler = () => {
        const id = row.dataset.id;
        console.log('Row clicked:', id);
      };

      row.addEventListener('click', handler);
      // ❌ 没有保存 handler 和 removeEventListener 的引用
    });
  }

  // ❌ 问题：定时器没有清理
  startAutoRefresh() {
    // ❌ 没有保存 timer ID，无法清理
    setInterval(() => {
      this.refreshData();
    }, 5000);
  }

  // ❌ 问题：闭包持有大对象引用
  refreshData() {
    // ❌ 闭包持有了 this.data 的引用
    fetch('/api/data')
      .then(response => response.json())
      .then(newData => {
        // ❌ 闭包意外持有了 this.data
        const diff = this.calculateDiff(this.data, newData);
        this.data = newData;
        this.render();
      });
  }

  // ❌ 问题：缓存无限增长
  cache = new Map();

  getCachedData(id) {
    // ❌ 没有清理机制
    if (!this.cache.has(id)) {
      const data = this.fetchData(id);
      this.cache.set(id, data);
    }
    return this.cache.get(id);
  }

  destroy() {
    // ❌ 没有清理监听器
    // ❌ 没有清理定时器
    // ❌ 没有清理缓存
  }
}
```

### 修复代码

```javascript
// file: src/components/DataGrid.js (已修复）

class DataGrid {
  constructor(container, data) {
    this.container = container;
    this.data = data;
    this.listeners = [];
    this.timers = [];
    this.cache = new Map();
    this.maxCacheSize = 1000;

    this.render();
    this.attachEvents();
    this.startAutoRefresh();
  }

  render() {
    this.container.innerHTML = `
      <div class="datagrid">
        ${this.data.map(item => `
          <div class="row" data-id="${item.id}">
            ${item.name}
          </div>
        `).join('')}
      </div>
    `;
  }

  // ✅ 修复：正确管理事件监听器
  attachEvents() {
    const rows = this.container.querySelectorAll('.row');
    rows.forEach(row => {
      const handler = () => {
        const id = row.dataset.id;
        console.log('Row clicked:', id);
      };

      row.addEventListener('click', handler);
      // ✅ 保存监听器引用
      this.listeners.push({
        element: row,
        event: 'click',
        handler
      });
    });
  }

  // ✅ 修复：使用事件委托
  attachEventsWithDelegation() {
    const handler = (event) => {
      if (event.target.classList.contains('row')) {
        const id = event.target.dataset.id;
        console.log('Row clicked:', id);
      }
    };

    this.container.addEventListener('click', handler);
    // ✅ 只保存一个监听器
    this.listeners.push({
      element: this.container,
      event: 'click',
      handler
    });
  }

  // ✅ 修复：正确管理定时器
  startAutoRefresh() {
    // ✅ 保存 timer ID
    const timerId = setInterval(() => {
      this.refreshData();
    }, 5000);
    this.timers.push(timerId);
  }

  // ✅ 修复：避免闭包持有大对象
  refreshData() {
    // ✅ 只提取需要的数据
    const currentIds = new Set(this.data.map(item => item.id));

    fetch('/api/data')
      .then(response => response.json())
      .then(newData => {
        // ✅ 只比较 ID，不持有完整数据
        const newIds = new Set(newData.map(item => item.id));
        const diff = this.calculateDiffIds(currentIds, newIds);

        this.data = newData;
        this.render();
      });
  }

  // ✅ 修复：添加缓存清理机制
  getCachedData(id) {
    if (this.cache.has(id)) {
      return this.cache.get(id);
    }

    // ✅ 检查缓存大小
    if (this.cache.size >= this.maxCacheSize) {
      this.cleanupCache();
    }

    const data = this.fetchData(id);
    this.cache.set(id, {
      data,
      timestamp: Date.now()
    });
    return data;
  }

  cleanupCache() {
    // ✅ 移除最老的条目
    const entries = Array.from(this.cache.entries());
    entries.sort((a, b) => a[1].timestamp - b[1].timestamp);

    const toRemove = Math.ceil(this.maxCacheSize * 0.2); // 移除 20%
    for (let i = 0; i < toRemove; i++) {
      this.cache.delete(entries[i][0]);
    }
  }

  // ✅ 修复：正确清理资源
  destroy() {
    // 清理所有事件监听器
    this.listeners.forEach(({ element, event, handler }) => {
      element.removeEventListener(event, handler);
    });
    this.listeners = [];

    // 清理所有定时器
    this.timers.forEach(timerId => {
      clearInterval(timerId);
    });
    this.timers = [];

    // 清理缓存
    this.cache.clear();

    // 清空容器
    this.container.innerHTML = '';
  }
}
```

## 验证修复

### 使用 Chrome DevTools

```javascript
// 1. 打开 DevTools Memory 标签
// 2. 点击 "Take heap snapshot" (基准)

// 3. 执行可能泄漏的操作
for (let i = 0; i < 100; i++) {
  const dataGrid = new DataGrid(container, mockData);
  // 执行一些操作
  dataGrid.destroy(); // 确保调用 destroy
}

// 4. 等待垃圾回收
// 点击 "Collect garbage" 按钮

// 5. 再次点击 "Take heap snapshot"

// 6. 对比两个快照
// 检查 Retained Objects 中是否有：
// - Detached DOM nodes
// - Event listeners
// - Timers
// - 闭包
```

### 使用 Node.js 内存分析

```javascript
// file: test/memory-leak.js

const v8 = require('v8');

function takeHeapSnapshot() {
  const snapshot = v8.getHeapSnapshot();
  const filename = `heap-snapshot-${Date.now()}.heapsnapshot`;
  require('fs').writeFileSync(filename, JSON.stringify(snapshot));
  console.log(`Heap snapshot saved: ${filename}`);
}

function getMemoryStats() {
  const stats = v8.getHeapStatistics();
  return {
    used: stats.used_heap_size / 1024 / 1024, // MB
    total: stats.total_heap_size / 1024 / 1024, // MB
    limit: stats.heap_size_limit / 1024 / 1024 // MB
  };
}

// 测试内存泄漏
console.log('初始内存:', getMemoryStats());
takeHeapSnapshot();

// 创建和销毁组件
const iterations = 1000;
for (let i = 0; i < iterations; i++) {
  const dataGrid = new DataGrid(container, mockData);
  dataGrid.destroy();

  if (i % 100 === 0) {
    console.log(`迭代 ${i}:`, getMemoryStats());
  }
}

console.log('最终内存:', getMemoryStats());
takeHeapSnapshot();

// 触发垃圾回收
global.gc(); // 需要 --expose-gc 标志
console.log('GC 后内存:', getMemoryStats());
```

## 关键学习点

### 1. 内存泄漏模式

| 模式 | 症状 | 检查方法 |
|------|------|----------|
| 事件监听器 | DOM 节点移除后仍在执行 | Detached DOM nodes |
| 定时器 | 定时器持续执行 | Active timers |
| 闭包 | 大对象无法回收 | Retained closures |
| 缓存 | 缓存大小增长 | Data structures |

### 2. 预防策略

1. **生命周期管理**
   - 创建时分配资源
   - 销毁时释放资源
   - 实现明确的 destroy 方法

2. **引用管理**
   - 只保留必要引用
   - 使用 WeakMap/WeakRef
   - 避免循环引用

3. **资源清理**
   - 定期清理缓存
   - 清理监听器
   - 清理定时器

4. **监控和报警**
   - 监控内存使用
   - 设置内存阈值
   - 触发警报

### 3. 工具使用

- Chrome DevTools Memory Profiler
- Node.js v8 模块
- heapdump 分析
- memory-leak 检测库

---

**提示：** 内存泄漏诊断需要系统化的方法。使用内存分析工具和 Agent 协作可以快速定位和修复问题。
