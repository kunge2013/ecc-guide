# 使用场景 2: 数据导出功能

## 需求描述

实现数据导出功能，支持：
- 多种格式（CSV、Excel、JSON）
- 大数据集处理
- 异步任务处理
- 进度跟踪
- 文件下载

## 技术要求

- 支持 CSV、Excel、JSON 格式
- 异步处理大数据集
- 提供导出进度
- 文件有效期 24 小时
- 限制同时导出任务数

## 完整实现流程

### 步骤 1: 架构设计

```bash
Agent(planner, """
设计数据导出功能架构：

需求：
- 支持多种格式（CSV、Excel、JSON）
- 大数据集处理（100万+ 记录）
- 异步任务处理
- 进度跟踪
- 文件下载

技术栈：Python + Flask + Celery + Redis + Pandas + OpenPyXL

请设计：
1. 任务队列架构
2. 文件存储方案
3. 进度跟踪机制
4. API 接口设计
5. 清理策略
""")
```

**核心设计：**

```python
# 使用 Celery 处理异步任务
from celery import Celery
import pandas as pd
from openpyxl import Workbook

app = Celery('export', broker='redis://localhost:6379/0')

@app.task
def export_data_task(task_id, data_query, format, filters):
    """
    异步导出任务
    """
    # 1. 更新任务状态为处理中
    update_task_status(task_id, 'processing', 0)

    # 2. 查询数据
    data = query_data(data_query, filters)
    total = len(data)

    # 3. 根据格式导出
    if format == 'csv':
        file_path = export_to_csv(data, task_id, total)
    elif format == 'excel':
        file_path = export_to_excel(data, task_id, total)
    elif format == 'json':
        file_path = export_to_json(data, task_id, total)

    # 4. 更新任务状态为完成
    update_task_status(task_id, 'completed', 100, file_path)

    return file_path

def export_to_csv(data, task_id, total):
    """CSV 格式导出"""
    file_path = f'/exports/{task_id}.csv'
    chunk_size = 10000

    for i in range(0, total, chunk_size):
        chunk = data[i:i + chunk_size]
        df = pd.DataFrame(chunk)

        # 追加模式写入
        mode = 'w' if i == 0 else 'a'
        df.to_csv(file_path, mode=mode, index=False, header=(i == 0))

        # 更新进度
        progress = min(100, int((i + chunk_size) / total * 100))
        update_task_status(task_id, 'processing', progress)

    return file_path

def export_to_excel(data, task_id, total):
    """Excel 格式导出"""
    file_path = f'/exports/{task_id}.xlsx'
    chunk_size = 10000

    workbook = Workbook()
    worksheet = workbook.active
    worksheet.append(data[0].keys())  # 写入表头

    for i, row in enumerate(data):
        worksheet.append(row.values())

        if (i + 1) % chunk_size == 0:
            # 更新进度
            progress = min(100, int((i + 1) / total * 100))
            update_task_status(task_id, 'processing', progress)

    workbook.save(file_path)
    return file_path
```

### 步骤 2: API 实现

```python
# export_api.py
from flask import Flask, request, jsonify
from celery.result import AsyncResult
import uuid

app = Flask(__name__)

@app.route('/api/export', methods=['POST'])
def create_export():
    """创建导出任务"""
    data = request.json

    # 生成任务 ID
    task_id = str(uuid.uuid4())

    # 验证格式
    valid_formats = ['csv', 'excel', 'json']
    if data.get('format') not in valid_formats:
        return jsonify({'error': '无效的导出格式'}), 400

    # 创建异步任务
    task = export_data_task.delay(
        task_id=task_id,
        data_query=data.get('query'),
        format=data.get('format'),
        filters=data.get('filters', {})
    )

    # 存储任务信息
    store_task_info(task_id, task.id)

    return jsonify({
        'task_id': task_id,
        'status': 'pending',
        'progress': 0
    }), 202

@app.route('/api/export/<task_id>/status')
def get_export_status(task_id):
    """获取导出任务状态"""
    task_info = get_task_info(task_id)

    if not task_info:
        return jsonify({'error': '任务不存在'}), 404

    task = AsyncResult(task_info['celery_task_id'])
    return jsonify({
        'task_id': task_id,
        'status': task.status,
        'progress': task_info.get('progress', 0),
        'file_url': task_info.get('file_url')
    })

@app.route('/api/export/<task_id>/download')
def download_export(task_id):
    """下载导出文件"""
    task_info = get_task_info(task_id)

    if not task_info:
        return jsonify({'error': '任务不存在'}), 404

    if task_info['status'] != 'completed':
        return jsonify({'error': '导出未完成'}), 400

    file_path = task_info['file_url']

    # 检查文件是否存在
    if not os.path.exists(file_path):
        return jsonify({'error': '文件不存在'}), 404

    return send_file(file_path, as_attachment=True)
```

### 步骤 3: 测试

```python
# test_export.py
import pytest
from app import app

@pytest.fixture
def client():
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client

def test_create_export_task(client):
    """测试创建导出任务"""
    response = client.post('/api/export', json={
        'format': 'csv',
        'query': 'SELECT * FROM users',
        'filters': {'status': 'active'}
    })

    assert response.status_code == 202
    data = response.get_json()
    assert 'task_id' in data
    assert data['status'] == 'pending'

def test_get_export_status(client):
    """测试获取导出状态"""
    # 先创建任务
    create_response = client.post('/api/export', json={
        'format': 'csv',
        'query': 'SELECT * FROM users'
    })
    task_id = create_response.get_json()['task_id']

    # 获取状态
    status_response = client.get(f'/api/export/{task_id}/status')

    assert status_response.status_code == 200
    data = status_response.get_json()
    assert data['task_id'] == task_id
    assert 'progress' in data

def test_export_invalid_format(client):
    """测试无效格式"""
    response = client.post('/api/export', json={
        'format': 'invalid',
        'query': 'SELECT * FROM users'
    })

    assert response.status_code == 400
    data = response.get_json()
    assert 'error' in data
```

## 关键学习点

### 1. 异步任务处理

使用 Celery 处理长时间运行的导出任务：
- 避免阻塞 HTTP 请求
- 支持任务队列和并发控制
- 提供任务状态跟踪

### 2. 大数据集处理

分批处理大数据集：
- 使用分块读写避免内存溢出
- 追加模式写入文件
- 实时更新进度

### 3. 进度跟踪

提供实时进度反馈：
- 任务状态（pending, processing, completed, failed）
- 进度百分比
- 预计剩余时间

### 4. 文件管理

- 临时文件存储
- 定期清理过期文件
- 提供下载链接

---

**提示：** 数据导出功能展示了如何处理长时间运行的任务和大数据集。
