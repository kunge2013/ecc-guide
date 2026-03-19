# 场景 2: 性能分析和优化

**类型：** 性能审查
**工具：** Lighthouse, Web Vitals
**优先级：** HIGH (用户体验)

## 场景目标

分析应用程序性能并识别优化机会：
1. 加载性能
2. 运行时性能
3. 资源优化
4. 渲染性能

## Lighthouse 性能审计

### 问题场景

```javascript
// file: src/app.js

// ❌ 问题 1: 未压缩的大 JavaScript 文件
import * as _ from 'lodash'; // 全量导入
import moment from 'moment';

class App {
  constructor() {
    // ❌ 问题 2: 同步阻塞操作
    this.data = this.loadDataSync();  // 阻塞事件循环

    // ❌ 问题 3: DOM 操作频繁
    this.updateUI();  // 频繁重排重绘

    // ❌ 问题 4: 大图片未优化
    this.images = this.loadImages();  // 加载未优化的图片
  }

  // ❌ 同步加载数据
  loadDataSync() {
    const data = fs.readFileSync('./data.json');  // 同步读取
    return JSON.parse(data);
  }

  // ❌ 频繁更新 DOM
  updateUI() {
    const items = this.data.items;
    for (let i = 0; i < items.length; i++) {
      // 每次迭代都更新 DOM
      document.getElementById(`item-${i}`).textContent = items[i];
    }
  }

  // ❌ 未优化的图片加载
  loadImages() {
    const images = [
      'https://example.com/huge-image-1.jpg',  // 5MB
      'https://example.com/huge-image-2.jpg',  // 6MB
      'https://example.com/huge-image-3.jpg'   // 4MB
    ];

    images.forEach(src => {
      const img = new Image();
      img.src = src;
      document.body.appendChild(img);  // 同步渲染
    });
  }

  // ❌ 未使用缓存
  expensiveCalculation(value) {
    // 每次都重新计算
    let result = 0;
    for (let i = 0; i < 10000; i++) {
      result += Math.pow(value, i);
    }
    return result;
  }
}
```

### Lighthouse 审计结果

```json
{
  "audits": {
    "first-contentful-paint": {
      "score": 0.5,
      "displayValue": "2.8 s",
      "status": "failed"
    },
    "largest-contentful-paint": {
      "score": 0.3,
      "displayValue": "5.2 s",
      "status": "failed"
    },
    "total-blocking-time": {
      "score": 0.4,
      "displayValue": "1,200 ms",
      "status": "failed"
    },
    "cumulative-layout-shift": {
      "score": 0.6,
      "displayValue": "0.25",
      "status": "needs-improvement"
"
    },
    "speed-index": {
      "score": 0.45,
      "displayValue": "4.8 s",
      "status": "failed"
    },
    "total-byte-weight": {
      "score": 0.2,
      "displayValue": "15.2 MB",
      "status": "failed"
    },
    "unused-javascript": {
      "score": 0.3,
      "displayValue": "2.1 MB",
      "status": "failed"
    },
    "render-blocking-resources": {
      "score": 0.4,
      "status": "failed"
    }
  },
  "categories": {
    "performance": {
      "score": 38
    }
  }
}
```

### 优化方案

```javascript
// file: src/app.js (已优化)

// ✅ 使用 Tree Shaking
import { debounce } from 'lodash-es';
import { format } from 'date-fns';  // 比 moment 小 96%

class App {
  constructor() {
    // ✅ 异步加载数据
    this.dataPromise = this.loadDataAsync();

    // ✅ 使用虚拟滚动
    this.setupVirtualScroll();

    // ✅ 懒加载图片
    this.setupLazyImages();

    // ✅ 使用缓存
    this.cache = new Map();
  }

  // ✅ 异步加载数据
  async loadDataAsync() {
    try {
      const response = await fetch('./data.json');
      const data = await response.json();
      return data;
    } catch (error) {
      console.error('Failed to load data:', error);
      return { items: [] };
    }
  }

  // ✅ 使用虚拟滚动减少 DOM 操作
  setupVirtualScroll() {
    const container = document.getElementById('items-container');
    const visibleItems = 20;

    const renderVisibleItems = debounce(() => {
      const scrollTop = container.scrollTop;
      const startIndex = Math.floor(scrollTop / 50);
      const endIndex = Math.min(startIndex + visibleItems, this.data.items.length);

      // 批量更新 DOM
      const fragment = document.createDocumentFragment();
      for (let i = startIndex; i < endIndex; i++) {
        const item = this.data.items[i];
        const element = this.createItemElement(i, item);
        fragment.appendChild(element);
      }

      container.innerHTML = '';
      container.appendChild(fragment);
    }, 16);  // ~60fps

    container.addEventListener('scroll', renderVisibleItems);
  }

  // ✅ 懒加载图片
  setupLazyImages() {
    const imageObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          img.src = img.dataset.src;
          img.loading = 'lazy';
          imageObserver.unobserve(img);
        }
      });
    });

    const lazyImages = document.querySelectorAll('img[data-src]');
    lazyImages.forEach(img => imageObserver.observe(img));
  }

  // ✅ 使用缓存
  expensiveCalculation(value) {
    const cacheKey = `calc-${value}`;

    if (this.cache.has(cacheKey)) {
      return this.cache.get(cacheKey);
    }

    let result = 0;
    for (let i = 0; i < 10000; i++) {
      result += Math.pow(value, i);
    }

    this.cache.set(cacheKey, result);
    return result;
  }

  // ✅ 创建优化的图片 URL
  getOptimizedImageUrl(src, width, height) {
    // 使用 CDN 优化
    const url = new URL(src);
    url.searchParams.set('auto', 'webp');
    url.searchParams.set('width', width);
    url.searchParams.set('height', height);
    url.searchParams.set('quality', '80');
    return url.toString();
  }
}
```

## Web Vitals 监控

```javascript
// file: src/performance.js

import { onCLS, onFID, onFCP, onLCP, onTTFB } from 'web-vitals';

export function setupPerformanceMonitoring() {
  // ✅ Cumulative Layout Shift (CLS)
  onCLS((metric) => {
    console.log('CLS:', metric.value);
    // CLS < 0.1 是好的
    if (metric.value > 0.1) {
      sendToAnalytics('CLS', metric);
    }
  });

  // ✅ First Input Delay (FID)
  onFID((metric) => {
    console.log('FID:', metric.value);
    // FID < 100ms 是好的
    if (metric.value > 100) {
      sendToAnalytics('FID', metric);
    }
  });

  // ✅ First Contentful Paint (FCP)
  onFCP((metric) => {
    console.log('FCP:', metric.value);
    // FCP < 1.8s 是好的
    if (metric.value > 1800) {
      sendToAnalytics('FCP', metric);
    }
  });

  // ✅ Largest Contentful Paint (LCP)
  onLCP((metric) => {
    console.log('LCP:', metric.value);
    // LCP < 2.5s 是好的
    if (metric.value > 2500) {
      sendToAnalytics('LCP', metric);
    }
  });

  // ✅ Time to First Byte (TTFB)
  onTTFB((metric) => {
    console.log('TTFB:', metric.value);
    // TTFB < 800ms 是好的
    if (metric.value > 800) {
      sendToAnalytics('TTFB', metric);
    }
  });
}

function sendToAnalytics(name, metric) {
  // 发送到分析工具
  gtag('event', name, {
    value: Math.round(metric.value),
    event_label: metric.id,
    custom_map: { metric_name: name }
  });
}
```

## 性能优化清单

```markdown
## 性能优化检查清单

### 加载性能
- [x] 启用 gzip/brotli 压缩
- [x] 使用 Tree Shaking
- [x] 代码分割
- [x] 懒加载路由
- [x] 优化图片（WebP, CDN）

### 运行时性能
- [x] 避免同步阻塞操作
- [x] 使用虚拟滚动
- [x] 防抖和节流
- [x] 使用缓存
- [x] 减少 DOM 操作

### 资源优化
- [x] 最小化 CSS/JS
- [x] 移除未使用的代码
- [x] 使用 CDN
- [x] HTTP/2 多路复用
- [x] 预加载关键资源

### 监控和分析
- [x] Web Vitals 监控
- [x] 性能分析工具
- [x] 用户真实监控
- [x] APM 集成
```

## CI/CD 性能检查

```yaml
# .github/workflows/performance-check.yml

name: Performance Check

on:
  pull_request:
    branches: [main]

jobs:
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v9
        with:
          urls: |
            http://localhost:3000
            http://localhost:3000/dashboard
          outputDir: ./lighthouse-report
          temporaryPublicStorage: true

  web-vitals:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3

      - name: Install dependencies
        run: npm install

      - name: Run Web Vitals
        run: npm run test:web-vitals

      - name: Check thresholds
        run: |
          # 检查性能指标
          node scripts/check-performance.js
```

## 关键要点

### 核心性能指标

| 指标 | 好的阈值 | 需要改进 | 差 |
|------|----------|----------|-----|
| FCP | < 1.8s | 1.8s - 3.0s | > 3.0s |
| LCP | < 2.5s | 2.5s - 4.0s | > 4.0s |
| TTI | < 3.8s | 3.8s - 7.3s | > 7.3s |
| CLS | < 0.1 | 0.1 - 0.25 | > 0.25 |
| FID | < 100ms | 100ms - 300ms | > 300ms |

### 优化策略

1. **加载优化**
   - 代码分割和懒加载
   - Tree Shaking 和最小化
   - CDN 和资源压缩
   - 预加载关键资源

2. **运行时优化**
   - 避免同步操作
   - 使用虚拟滚动
   - 防抖和节流
   - 合理使用缓存

3. **渲染优化**
   - 减少 DOM 操作
   - 使用 CSS 动画
   - 避免强制同步布局
   - 使用 will-change

---

**场景价值：** 性能直接影响用户体验和 SEO。定期性能分析和优化是保持应用竞争力的关键。
