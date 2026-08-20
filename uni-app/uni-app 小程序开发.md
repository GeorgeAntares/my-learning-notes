# uni-app 小程序开发 / uni-app Mini-Program Development

## 一、项目结构 / Project Structure

LOOP 项目有两个小程序端：

The LOOP project has two mini-program clients:

| 小程序 Mini-Program | 目录 Directory | 说明 Description |
|------|------|------|
| 用户端 User client | `user-mini/` | 面向普通用户 / For regular users |
| 场馆端 Venue client | `venue-mini/` | 面向场馆运营 / For venue operators |

### 目录结构 / Directory Structure

```
user-mini/（或 venue-mini/）
├── pages/              → 页面 / Pages
├── components/         → 自定义组件 / Custom components
├── static/             → 静态资源（图片等）/ Static assets
├── utils/
│   ├── config.js       → 后端 API 地址配置 / Backend API URL config
│   └── dev-mock.js     → Mock 登录（开发用）/ Mock login (dev only)
├── App.vue             → 应用入口 / App entry
├── main.js             → JS 入口 / JS entry
├── pages.json          → 页面路由配置 / Page routing config
├── manifest.json       → 应用配置 / App manifest
└── uni.scss            → 全局样式 / Global styles
```

## 二、环境配置 / Environment Configuration

### 后端地址 / Backend URL

在 `utils/config.js` 中配置后端 API 地址：

Configure backend API URL in `utils/config.js`:

```javascript
// 每次重连 WiFi 后 IP 可能变化，需要同步更新
// IP may change after reconnecting WiFi, update accordingly
const BASE_URL = 'http://192.168.6.150:8080'

export default {
  BASE_URL
}
```

### Mock 登录 / Mock Login

开发阶段用 mock 登录绕过微信登录：

Use mock login during development to bypass WeChat login:

```javascript
// utils/dev-mock.js
// 在 main.js 中引入：import './utils/dev-mock'
// Import in main.js: import './utils/dev-mock'
```

| 注意 Note | 说明 Description |
|------|------|
| 仅开发环境用 Dev only | 不要提交到生产 / Don't commit to production |
| main.js 需引入 Need import in main.js | `import './utils/dev-mock'` |
| 场馆端 mock 无真实权限 Venue mock no real permission | API 返回"无权操作该场馆"/ API returns "no permission" |

## 三、页面注册 / Page Registration

在 `pages.json` 中注册页面：

Register pages in `pages.json`:

```json
{
  "pages": [
    {
      "path": "pages/index/index",
      "style": {
        "navigationBarTitleText": "首页 / Home"
      }
    },
    {
      "path": "pages/practice/practice",
      "style": {
        "navigationBarTitleText": "练习页 / Practice"
      }
    }
  ]
}
```

### 每日检查清单 / Daily Checklist

每天拉取更新后检查：

Check after pulling updates daily:

| 检查项 Check Item | 文件 File |
|------|------|
| uview-ui 引用是否清理 / uview-ui refs cleaned | `uni.scss`、`main.js`、`App.vue`、`pages.json`、`vue.config.js` |
| 练习页是否注册 / Practice page registered | `pages.json` |
| dev-mock 是否引入 / dev-mock imported | `main.js` |
| BASE_URL 是否正确 / BASE_URL correct | `utils/config.js` |

## 四、组件开发 / Component Development

### 基本组件结构 / Basic Component Structure

```vue
<template>
  <view class="component-name">
    <text>{{ message }}</text>
  </view>
</template>

<script>
export default {
  props: {
    message: {
      type: String,
      default: '默认值 / Default'
    }
  },
  data() {
    return {}
  },
  methods: {}
}
</script>

<style scoped>
.component-name {
  padding: 20rpx;
}
</style>
```

### uni-app 尺寸单位 / Size Units

| 单位 Unit | 说明 Description |
|------|------|
| rpx | 响应式像素（推荐）/ Responsive pixel (recommended) |
| px | 固定像素 / Fixed pixel |
| % | 百分比 / Percentage |

750rpx = 屏幕宽度 / Screen width

## 五、依赖安装 / Dependency Installation

### 用户端 / User Client

```bash
cd user-mini
npm install
```

### 场馆端 / Venue Client

场馆端有 webpack 版本冲突，需要加 `--legacy-peer-deps`：

Venue client has webpack version conflict, need `--legacy-peer-deps`:

```bash
cd venue-mini
npm install --legacy-peer-deps
```

## 六、微信开发者工具 / WeChat DevTools

### 域名校验 / Domain Verification

开发时需要关闭域名校验：

Disable domain verification during development:

```
详情 → 本地设置 → 勾选"不校验合法域名"
Details → Local Settings → Check "Do not verify domain"
```

### 常见问题 / Common Issues

| 问题 Issue | 解决 Solution |
|------|------|
| 白屏 White screen | 检查 `main.js` 是否引入了不存在的依赖 / Check if main.js imports non-existent deps |
| 网络请求失败 Network request failed | 检查 `config.js` 的 BASE_URL 和后端是否启动 / Check BASE_URL and backend |
| 登录失败 Login failed | 检查 dev-mock 是否引入，或微信 AppID/AppSecret 是否配置 / Check dev-mock or WeChat config |
| uview-ui 报错 uview-ui error | 删除 5 个文件中的 uview-ui 引用 / Remove uview-ui refs from 5 files |

## 七、与后端联调 / Frontend-Backend Integration

### 请求流程 / Request Flow

```
小程序发起请求 → config.js 的 BASE_URL → 后端 Controller → 数据库
Mini-program sends request → BASE_URL → Backend Controller → Database
```

### 调用后端接口 / Calling Backend API

```javascript
import config from '@/utils/config.js'

uni.request({
  url: config.BASE_URL + '/api/v1/notices',
  method: 'GET',
  success: (res) => {
    console.log('公告列表 / Notice list:', res.data)
  }
})
```

### 需要 token 的接口 / APIs Requiring Token

```javascript
uni.request({
  url: config.BASE_URL + '/api/v1/notices/1/read',
  method: 'POST',
  header: {
    'Authorization': 'Bearer ' + uni.getStorageSync('token')
  },
  success: (res) => {
    console.log('标记已读 / Marked as read')
  }
})
```
