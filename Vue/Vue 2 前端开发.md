# Vue 2 前端开发 / Vue 2 Frontend Development

## 一、Vue 基础概念 / Vue Basics

### 什么是 Vue / What is Vue

Vue 是渐进式 JavaScript 框架，用于构建用户界面。RuoYi 前端使用 Vue 2 + Element UI。

Vue is a progressive JavaScript framework for building UIs. RuoYi frontend uses Vue 2 + Element UI.

### Vue 实例 / Vue Instance

```javascript
new Vue({
  el: '#app',           // 挂载点 / Mount point
  data: {               // 数据 / Data
    message: 'Hello Vue'
  },
  methods: {            // 方法 / Methods
    handleClick() {
      this.message = 'Clicked'
    }
  },
  created() {           // 生命周期钩子 / Lifecycle hook
    console.log('组件已创建 / Component created')
  }
})
```

### 生命周期 / Lifecycle Hooks

```
beforeCreate  → created  → beforeMount  → mounted  → beforeUpdate  → updated  → beforeDestroy  → destroyed
```

| 钩子 Hook | 常用场景 Common Use Case |
|------|------|
| `created` | 初始化数据 / Initialize data |
| `mounted` | 操作 DOM、发请求 / DOM ready, send requests |
| `beforeDestroy` | 清理定时器等 / Clean up timers |

## 二、模板语法 / Template Syntax

### 文本渲染 / Text Rendering

```html
<!-- 插值 / Interpolation -->
<p>{{ message }}</p>

<!-- v-text / v-text directive -->
<p v-text="message"></p>

<!-- v-html（渲染HTML）/ v-html (render HTML) -->
<p v-html="htmlContent"></p>
```

### 条件渲染 / Conditional Rendering

```html
<!-- v-if：不满足时 DOM 不存在 / v-if: element removed from DOM -->
<p v-if="show">显示 / Visible</p>
<p v-else>隐藏 / Hidden</p>

<!-- v-show：始终存在，通过 display 控制 / v-show: always in DOM, toggle display -->
<p v-show="show">显示 / Visible</p>
```

| 区别 Difference | v-if | v-show |
|------|------|------|
| DOM | 创建/删除 Create/Destroy | 始终存在 Always present |
| 性能 Performance | 切换开销大 Higher switch cost | 初始渲染开销大 Higher initial cost |
| 使用场景 Use case | 不频繁切换 Infrequent toggle | 频繁切换 Frequent toggle |

### 列表渲染 / List Rendering

```html
<!-- v-for 遍历数组 / v-for iterate array -->
<el-table :data="noticeList">
  <el-table-column prop="noticeTitle" label="标题 / Title" />
  <el-table-column prop="noticeType" label="类型 / Type" />
  <el-table-column label="操作 / Actions">
    <template slot-scope="scope">
      <el-button @click="handleEdit(scope.row)">编辑 / Edit</el-button>
      <el-button @click="handleDelete(scope.row)">删除 / Delete</el-button>
    </template>
  </el-table-column>
</el-table>
```

### 属性绑定 / Attribute Binding

```html
<!-- v-bind 简写为 : / v-bind shorthand: -->
<img :src="imageUrl" :alt="title">
<div :class="{ active: isActive }" :style="{ color: textColor }">
```

### 事件绑定 / Event Binding

```html
<!-- v-on 简写为 @ / v-on shorthand: @ -->
<button @click="handleClick">点击 / Click</button>
<button @click="handleClick($event, extraParam)">带参数 / With params</button>
<input @keyup.enter="handleSubmit" v-model="inputValue">
```

### 双向绑定 / Two-Way Binding

```html
<!-- v-model 双向绑定 / v-model two-way binding -->
<el-input v-model="form.title" placeholder="请输入标题 / Enter title" />
<el-select v-model="form.type">
  <el-option label="通知 / Notification" value="1" />
  <el-option label="公告 / Announcement" value="2" />
</el-select>
```

## 三、组件 / Components

### 定义组件 / Define Component

```vue
<template>
  <div class="notice-card">
    <h3>{{ title }}</h3>
    <p>{{ content }}</p>
    <el-button @click="handleClick">详情 / Detail</el-button>
  </div>
</template>

<script>
export default {
  name: 'NoticeCard',
  // 父组件传入的数据 / Data passed from parent
  props: {
    title: {
      type: String,
      required: true
    },
    content: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      isOpen: false
    }
  },
  methods: {
    handleClick() {
      // 向父组件发送事件 / Emit event to parent
      this.$emit('click', this.title)
    }
  }
}
</script>

<style scoped>
.notice-card {
  padding: 20px;
  border: 1px solid #eee;
}
</style>
```

### 使用组件 / Use Component

```vue
<template>
  <div>
    <!-- 监听子组件事件 / Listen to child event -->
    <notice-card
      title="系统维护通知"
      content="今晚22:00维护"
      @click="handleNoticeClick"
    />
  </div>
</template>

<script>
import NoticeCard from '@/components/NoticeCard.vue'

export default {
  components: { NoticeCard },
  methods: {
    handleNoticeClick(title) {
      console.log('点击了 / Clicked:', title)
    }
  }
}
</script>
```

### 组件通信 / Component Communication

| 方式 Method | 方向 Direction | 说明 Description |
|------|------|------|
| props | 父→子 Parent→Child | 传递数据 / Pass data |
| $emit | 子→父 Child→Parent | 触发事件 / Trigger event |
| $refs | 父→子 Parent→Child | 直接调用子组件方法 / Call child method |
| EventBus | 任意 Any | 全局事件总线 / Global event bus |
| Vuex | 任意 Any | 全局状态管理 / Global state management |

## 四、与后端交互 / Backend Interaction

### 发送请求 / Send Request

```javascript
import { listNotice, addNotice } from '@/api/loop/notice'

export default {
  methods: {
    // 查询列表 / Query list
    async loadData() {
      const res = await listNotice({ pageNum: 1, pageSize: 10 })
      this.list = res.rows
      this.total = res.total
    },

    // 新增 / Create
    async handleAdd() {
      await addNotice({ title: '新公告', type: '2', content: '<p>内容</p>' })
      this.$message.success('新增成功 / Added successfully')
      this.loadData()
    }
  }
}
```

### API 定义 / API Definition

```javascript
// src/api/loop/notice.js
import request from '@/utils/request'

export function listNotice(query) {
  return request({
    url: '/api/v1/ops/notice',
    method: 'get',
    params: query
  })
}

export function addNotice(data) {
  return request({
    url: '/api/v1/ops/notice',
    method: 'post',
    data: data
  })
}
```

## 五、Element UI 常用组件 / Element UI Components

### 表格 Table

```html
<el-table :data="list" border>
  <el-table-column type="index" label="序号" width="50" />
  <el-table-column prop="title" label="标题" />
  <el-table-column prop="createTime" label="创建时间" />
  <el-table-column label="操作" width="200">
    <template slot-scope="scope">
      <el-button size="mini" @click="handleEdit(scope.row)">编辑</el-button>
      <el-button size="mini" type="danger" @click="handleDelete(scope.row)">删除</el-button>
    </template>
  </el-table-column>
</el-table>
```

### 对话框 Dialog

```html
<el-dialog :title="title" :visible.sync="dialogVisible" width="500px">
  <el-form :model="form" :rules="rules" ref="form" label-width="80px">
    <el-form-item label="标题" prop="title">
      <el-input v-model="form.title" />
    </el-form-item>
    <el-form-item label="类型" prop="type">
      <el-select v-model="form.type">
        <el-option label="通知" value="1" />
        <el-option label="公告" value="2" />
      </el-select>
    </el-form-item>
  </el-form>
  <span slot="footer">
    <el-button @click="dialogVisible = false">取消</el-button>
    <el-button type="primary" @click="handleSubmit">确定</el-button>
  </span>
</el-dialog>
```

### 分页 Pagination

```html
<el-pagination
  :current-page="queryParams.pageNum"
  :page-size="queryParams.pageSize"
  :total="total"
  @current-change="handlePageChange"
  layout="total, prev, pager, next"
/>
```

## 六、项目结构 / Project Structure

```
ruoyi-ui/
├── src/
│   ├── api/              → 接口定义 / API definitions
│   │   └── loop/         → LOOP 模块接口 / LOOP module APIs
│   ├── views/            → 页面 / Pages
│   │   └── loop/         → LOOP 模块页面 / LOOP module pages
│   ├── components/       → 公共组件 / Shared components
│   ├── utils/            → 工具函数 / Utilities
│   ├── router/           → 路由配置 / Router config
│   ├── store/            → Vuex 状态管理 / Vuex state
│   └── permission.js     → 权限控制 / Permission control
├── vue.config.js         → 构建配置 / Build config
└── package.json          → 依赖管理 / Dependencies
```

## 七、常用技巧 / Common Tips

### 表单校验 / Form Validation

```javascript
rules: {
  title: [
    { required: true, message: '请输入标题 / Please enter title', trigger: 'blur' },
    { max: 50, message: '不超过50个字符 / Max 50 characters', trigger: 'blur' }
  ],
  type: [
    { required: true, message: '请选择类型 / Please select type', trigger: 'change' }
  ]
}

// 提交前校验 / Validate before submit
this.$refs.form.validate(valid => {
  if (valid) {
    // 校验通过，提交 / Valid, submit
  }
})
```

### 消息提示 / Message Tips

```javascript
this.$message.success('操作成功 / Success')
this.$message.error('操作失败 / Failed')
this.$message.warning('请先选择 / Please select first')

this.$confirm('确认删除？/ Confirm delete?', '提示', {
  type: 'warning'
}).then(() => {
  // 确认删除 / Confirmed
})
```
