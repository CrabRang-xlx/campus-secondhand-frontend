# 校园二手交易平台前端系统

## 1. 项目简介

本项目是校园二手交易平台的前端页面，基于 Vue 3 和 Vite 开发。

前端通过 Axios 调用 Spring Boot 后端接口，实现用户登录、注册、商品浏览、商品搜索、商品详情、留言、收藏、发布商品、我的发布、我的收藏和管理员管理等功能。

本项目与后端项目配套使用：

```text
后端项目：campus-secondhand-backend
前端项目：campus-secondhand-frontend
```

---

## 配套后端项目

后端仓库地址：

```text
https://github.com/CrabRang-xlx/campus-secondhand-backend

## 2. 技术栈

| 类型 | 技术 |
|---|---|
| 前端框架 | Vue 3 |
| 构建工具 | Vite |
| HTTP 请求 | Axios |
| 状态存储 | localStorage |
| 开发语言 | JavaScript |
| 样式 | CSS |
| 开发工具 | IntelliJ IDEA |

---

## 3. 已实现功能

### 3.1 用户功能

- 用户注册
- 用户登录
- 退出登录
- 登录状态本地保存
- 根据登录用户显示昵称

### 3.2 商品功能

- 商品列表展示
- 商品搜索
- 商品详情查看
- 发布商品
- 查看我的发布
- 修改商品状态：在售 / 已售出 / 已下架
- 删除自己发布的商品

### 3.3 收藏功能

- 收藏商品
- 取消收藏
- 查看我的收藏

### 3.4 留言功能

- 查看商品留言
- 发布商品留言
- 留言显示用户昵称

### 3.5 管理员功能

- 管理员入口
- 查看所有用户
- 禁用 / 恢复用户
- 查看所有商品
- 下架 / 恢复商品

---

## 4. 项目结构

```text
campus-secondhand-frontend
├─ node_modules
├─ public
├─ src
│  ├─ App.vue
│  ├─ main.js
│  └─ style.css
├─ index.html
├─ package.json
├─ package-lock.json
├─ README.md
└─ vite.config.js
```

---

## 5. 页面说明

目前前端主要集中在 `src/App.vue` 中，实现了以下页面状态：

| 页面 | 说明 |
|---|---|
| 首页 | 商品列表和搜索 |
| 登录 / 注册区域 | 用户账号操作 |
| 商品详情页 | 商品信息、收藏、留言 |
| 发布商品页 | 登录用户发布商品 |
| 我的发布页 | 查看和管理自己发布的商品 |
| 我的收藏页 | 查看和取消收藏商品 |
| 管理员页面 | 管理用户和商品 |

---

## 6. 后端接口依赖

前端默认请求后端地址：

```text
http://localhost:8080
```

主要使用接口包括：

```text
POST   /api/users/register
POST   /api/users/login

GET    /api/products
GET    /api/products/{id}
GET    /api/products/search
GET    /api/products/user/{userId}
POST   /api/products
PUT    /api/products/{id}/status
DELETE /api/products/{id}

POST   /api/favorites
DELETE /api/favorites
GET    /api/favorites/user/{userId}

POST   /api/comments
GET    /api/comments/product/{productId}

GET    /api/admin/users
PUT    /api/admin/users/{userId}/status
GET    /api/admin/products
PUT    /api/admin/products/{productId}/status
```

---

## 7. 本地运行方式

### 7.1 安装依赖

```bash
npm install
```

### 7.2 启动前端项目

```bash
npm run dev
```

启动成功后访问：

```text
http://localhost:5173
```

如果 5173 被占用，Vite 可能会自动使用：

```text
http://localhost:5174
```

此时需要确认后端跨域配置是否允许该端口访问。

---

## 8. 运行前准备

运行前需要先启动后端项目：

```text
campus-secondhand-backend
```

后端启动成功后，浏览器访问：

```text
http://localhost:8080/api/products
```

如果能看到 JSON 数据，说明后端服务正常。

然后再启动前端项目：

```bash
npm run dev
```

---

## 9. 测试账号说明

普通用户可以通过前端注册页面创建。

管理员账号需要在 MySQL 中设置用户角色：

```sql
UPDATE user SET role = 'ADMIN' WHERE id = 你的用户ID;
```

然后重新登录该账号，页面顶部会显示：

```text
管理员
```

---

## 项目截图

### 1. 后端商品接口返回数据

![后端商品接口返回数据](docs/images/01_backend_products_api.png)

### 2. 前端首页商品列表

![前端首页商品列表](docs/images/02_frontend_home_product_list.png)

### 3. 用户登录成功

![用户登录成功](docs/images/03_user_login_success.png)

### 4. 商品详情与留言

![商品详情与留言](docs/images/04_product_detail_comments.png)


## 10. 项目亮点

1. 使用 Vue 3 构建单页面前端应用。
2. 使用 Axios 与 Spring Boot 后端进行数据交互。
3. 支持完整的用户、商品、收藏、留言和管理员功能。
4. 使用 localStorage 保存当前登录用户状态。
5. 页面结构清晰，适合作为个人项目展示。
6. 与后端项目配套，形成完整的前后端分离项目。

---

## 11. 后续优化方向

- 拆分 Vue 组件
- 引入 Vue Router
- 引入 Pinia 管理登录状态
- 增加 Element Plus UI 组件库
- 增加商品图片上传
- 增加分页查询
- 增加移动端适配
- 对接 JWT 登录认证
- 部署到服务器或云平台