# admin/ - Web 管理后台模块

[根目录](../CLAUDE.md) > **admin**

---

## 📋 变更记录

### 2026-01-18 20:57:24 - 初始文档创建
- 完成管理后台架构梳理
- 建立 API 路由索引
- 提供前后端扩展指引

---

## 🎯 模块职责

Web 管理后台是 AllBot 的**可视化控制中心**，基于 FastAPI + Bootstrap 5 构建，提供以下核心功能：

- **控制面板**：系统概览、机器人状态监控、实时日志
- **插件管理**：安装/卸载/启用/禁用插件、配置编辑
- **适配器管理**：多平台适配器状态查看与配置
- **文件管理**：上传/下载/删除机器人使用的文件
- **账号管理**：多微信账号绑定与切换
- **联系人管理**：好友/群组列表查看与搜索
- **通知管理**：系统事件通知与告警配置
- **AI 平台管理**：模型平台密钥配置
- **系统设置**：全局配置项编辑与保存
- **插件市场**：浏览与安装插件市场资源

---

## 🚀 入口与启动

### 启动流程

1. **主程序启动**（`main.py` 调用）
   ```python
   from admin.server import start_server

   server_thread = start_server(
       host_arg="0.0.0.0",
       port_arg=9090,
       username_arg="admin",
       password_arg="admin123",
       debug_arg=False,
       bot=None
   )
   ```

2. **FastAPI 应用初始化**（`admin/server.py`）
   ```python
   app = FastAPI(title="AllBot 管理后台")
   app.mount("/static", StaticFiles(directory="admin/static"))
   templates = Jinja2Templates(directory="admin/templates")
   ```

3. **路由注册**（`admin/routes/register_routes.py`）
   ```python
   from admin.routes.plugin_routes import router as plugin_router
   app.include_router(plugin_router, prefix="/api/plugins")
   ```

4. **启动 Uvicorn**
   ```python
   uvicorn.run(app, host="0.0.0.0", port=9090)
   ```

### 配置项（main_config.toml）

```toml
[Admin]
enabled = true
host = "0.0.0.0"
port = 9090
username = "admin"
password = "admin123"
debug = false
log_level = "INFO"
```

---

## 🔌 对外接口（API 路由）

### 核心 API 列表

#### 1. 插件管理（`admin/routes/plugin_routes.py`）
- `GET /api/plugins/list`：获取所有插件列表
- `POST /api/plugins/toggle`：启用/禁用插件
- `POST /api/plugins/reload`：重载插件
- `GET /api/plugins/config`：获取插件配置
- `POST /api/plugins/config`：保存插件配置

#### 2. 系统监控（`admin/system_stats_api.py`）
- `GET /api/system/stats`：系统资源占用（CPU/内存/磁盘）
- `GET /api/system/logs`：实时日志流
- `GET /api/bot/status`：机器人在线状态

#### 3. 文件管理（`admin/file_manager.py`）
- `GET /api/files/list`：文件列表
- `POST /api/files/upload`：上传文件
- `DELETE /api/files/delete`：删除文件
- `GET /api/files/download`：下载文件

#### 4. 账号管理（`admin/account_manager.py`）
- `GET /api/accounts/list`：账号列表
- `POST /api/accounts/switch`：切换账号
- `POST /api/accounts/logout`：登出账号

#### 5. 朋友圈（`admin/friend_circle_api.py`）
- `GET /api/pyq/list`：朋友圈列表
- `POST /api/pyq/like`：点赞
- `POST /api/pyq/comment`：评论

#### 6. 提醒管理（`admin/reminder_api.py`）
- `GET /api/reminders/list`：提醒列表
- `POST /api/reminders/add`：添加提醒
- `DELETE /api/reminders/delete`：删除提醒

#### 7. 终端管理（`admin/terminal_routes.py`）
- `WebSocket /api/terminal/ws`：Web 终端 WebSocket 连接

### API 认证机制

所有 API 需要通过 JWT Token 或 Session 认证：
```python
from admin.auth_helper import require_auth

@app.get("/api/protected")
@require_auth
async def protected_route(request: Request):
    # 受保护的路由
    pass
```

---

## 🔗 关键依赖与配置

### 后端依赖

- **FastAPI**：Web 框架（~0.110.0）
- **Uvicorn**：ASGI 服务器（~0.30.0）
- **Jinja2**：模板引擎（~3.1.3）
- **python-multipart**：文件上传（~0.0.9）
- **itsdangerous**：Session 签名（~2.1.2）
- **psutil**：系统监控（~5.9.8）

### 前端依赖

**CSS 框架**：
- Bootstrap 5.3+（`admin/static/css/lib/bootstrap.min.css`）
- Bootstrap Icons（`admin/static/css/lib/bootstrap-icons.css`）

**JavaScript 库**：
- Vue 3（`admin/static/js/lib/vue.global.min.js`）
- jQuery 3.6+（`admin/static/js/lib/jquery.min.js`）
- Chart.js（图表展示）
- Marked.js（Markdown 渲染）
- AOS.js（动画效果）

**自定义资源**：
- `admin/static/js/custom.js`：全局 JS 逻辑
- `admin/static/js/file-manager.js`：文件管理功能
- `admin/static/css/qrcode.css`：二维码样式

### 目录结构

```
admin/
├── server.py              # FastAPI 主应用入口
├── run_server.py          # 独立启动脚本
├── routes/                # API 路由模块
│   ├── __init__.py
│   ├── register_routes.py # 路由注册器
│   └── plugin_routes.py   # 插件相关 API
├── templates/             # Jinja2 HTML 模板
│   ├── ai_platforms.html  # AI 平台管理页
│   ├── notification.html  # 通知设置页
│   └── ...
├── static/                # 静态资源
│   ├── css/               # 样式文件
│   ├── js/                # JavaScript 文件
│   └── img/               # 图片资源
├── utils/                 # 管理后台工具函数
│   └── plugin_manager.py  # 插件管理工具
├── auth_helper.py         # 认证辅助函数
├── system_stats_api.py    # 系统监控 API
├── friend_circle_api.py   # 朋友圈 API
├── reminder_api.py        # 提醒 API
├── terminal_routes.py     # 终端 WebSocket
└── switch_account_api.py  # 账号切换 API
```

---

## 📊 数据模型

### 插件信息对象
```python
{
    "name": "PluginName",
    "description": "插件功能描述",
    "author": "作者名称",
    "version": "1.0.0",
    "enabled": True,
    "priority": 80,
    "config": {...}  # config.toml 内容
}
```

### 系统状态对象
```python
{
    "cpu_usage": 25.5,       # CPU 使用率（%）
    "memory_usage": 512.3,   # 内存使用量（MB）
    "memory_total": 8192.0,  # 总内存（MB）
    "disk_usage": 45.2,      # 磁盘使用率（%）
    "bot_status": "online",  # 机器人状态
    "wxid": "wxid_xxx",      # 当前微信 ID
    "nickname": "昵称"       # 昵称
}
```

### 文件对象
```python
{
    "name": "example.jpg",
    "path": "/app/files/example.jpg",
    "size": 123456,          # 字节
    "modified_time": 1234567890,
    "type": "image/jpeg"
}
```

---

## 🧪 测试与质量

### 手动测试步骤

1. **启动后台**：`python admin/run_server.py`
2. **访问**：http://localhost:9090
3. **登录**：使用 `main_config.toml` 中的用户名密码
4. **功能测试**：
   - 插件管理：启用/禁用/重载
   - 文件上传：测试大文件上传
   - 系统监控：查看 CPU/内存图表
   - 终端连接：WebSocket 连接测试

### API 测试（Pytest）

```python
from fastapi.testclient import TestClient
from admin.server import app

client = TestClient(app)

def test_get_plugins():
    response = client.get("/api/plugins/list")
    assert response.status_code == 200
    assert isinstance(response.json(), list)
```

### 性能优化建议

- **静态资源**：启用 CDN 或 Nginx 缓存
- **日志流**：使用 WebSocket 代替 HTTP 轮询
- **图表渲染**：前端使用懒加载，避免首屏卡顿

---

## ❓ 常见问题 (FAQ)

### Q1: 如何添加新页面？
**A**：
1. 在 `admin/templates/` 中创建 HTML 模板
2. 在 `admin/server.py` 或 `admin/routes/` 中添加路由
3. 在前端导航菜单中添加链接（修改模板的侧边栏部分）

### Q2: 如何自定义主题？
**A**：修改 `admin/static/css/custom.css`，覆盖 Bootstrap 默认样式。

### Q3: 如何启用 HTTPS？
**A**：在 Uvicorn 启动时指定 SSL 证书：
```python
uvicorn.run(app, host="0.0.0.0", port=9090,
            ssl_keyfile="key.pem", ssl_certfile="cert.pem")
```

### Q4: 如何限制管理员 IP？
**A**：在 `auth_helper.py` 中添加 IP 白名单检查逻辑。

### Q5: 如何查看实时日志？
**A**：访问控制面板的"系统日志"标签，或使用 `GET /api/system/logs` API。

---

## 📁 相关文件清单

### 核心文件
- `admin/server.py`：FastAPI 主应用（约 500+ 行）
- `admin/run_server.py`：独立启动脚本
- `admin/routes/register_routes.py`：路由注册器
- `admin/auth_helper.py`：认证中间件

### 前端模板
- `admin/templates/ai_platforms.html`：AI 平台管理
- `admin/templates/notification.html`：通知设置
- `admin/templates/base.html`：基础模板（含侧边栏）

### 静态资源
- `admin/static/js/custom.js`：全局 JS 逻辑
- `admin/static/js/file-manager.js`：文件管理功能
- `admin/static/css/lib/bootstrap.min.css`：Bootstrap 框架

### 配置文件
- `main_config.toml`：管理后台配置（[Admin] 部分）
- `admin/config.json`：运行时配置缓存

---

## 🔧 扩展指引

### 添加新 API 路由

**步骤**：
1. 在 `admin/routes/` 中创建新文件（如 `my_routes.py`）
2. 定义路由：
   ```python
   from fastapi import APIRouter
   router = APIRouter()

   @router.get("/my-endpoint")
   async def my_endpoint():
       return {"message": "Hello"}
   ```
3. 在 `admin/routes/register_routes.py` 中注册：
   ```python
   from admin.routes.my_routes import router as my_router
   app.include_router(my_router, prefix="/api/my")
   ```

### 添加前端页面

**步骤**：
1. 在 `admin/templates/` 中创建 `my_page.html`
2. 继承基础模板：
   ```html
   {% extends "base.html" %}
   {% block content %}
       <h1>My Page</h1>
   {% endblock %}
   ```
3. 在 `admin/server.py` 中添加路由：
   ```python
   @app.get("/my-page")
   async def my_page(request: Request):
       return templates.TemplateResponse("my_page.html", {"request": request})
   ```

---

**维护者提示**：管理后台代码较为复杂，修改时请确保不破坏现有 API 兼容性。建议使用 API 版本控制（如 `/api/v2/`）实现新功能。
