# Vue2/Vue3 模板提炼参考指南

## 提炼流程概览

```
源项目 src/
├── views/user/List.vue          ──▶  page.vue.tera / page.vue3.tera
├── components/TableForm.vue     ──▶  component.vue.tera
├── api/user.js                  ──▶  api.js.tera
├── store/modules/user.js        ──▶  store.js.tera
└── router/modules/user.js       ──▶  router.js.tera
```

## Vue2 vs Vue3 差异处理

| 特性 | Vue2 (Options API) | Vue3 (Composition API) |
|------|-------------------|----------------------|
| 组件定义 | `export default { name, data, methods }` | `<script setup>` + `defineOptions` |
| 响应式数据 | `data() { return { ... } }` | `ref()` / `reactive()` |
| 路由 | `this.$router` | `useRouter()` |
| 状态管理 | `this.$store` (Vuex) | `useStore()` (Pinia) |
| 模板语法 | 相同 | 相同（支持多根节点） |

## 提炼对照示例

### 1. 页面组件 → page.vue3.tera

**源文件 (views/user/List.vue):**
```vue
<template>
  <div class="user-list">
    <el-form :model="searchForm" inline>
      <el-form-item label="用户名">
        <el-input v-model="searchForm.userName" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleSearch">搜索</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="tableData" border>
      <el-table-column prop="userName" label="用户名" width="120" />
      <el-table-column prop="email" label="邮箱" />
      <el-table-column label="操作" width="200">
        <template #default="{ row }">
          <el-button @click="handleEdit(row)">编辑</el-button>
          <el-button type="danger" @click="handleDelete(row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { userApi } from '@/api/user'

defineOptions({ name: 'UserList' })

const router = useRouter()
const searchForm = ref({ userName: '' })
const tableData = ref([])

const handleSearch = async () => {
  const res = await userApi.list(searchForm.value)
  tableData.value = res.data
}

const handleEdit = (row) => {
  router.push(`/user/edit/${row.id}`)
}

const handleDelete = async (id) => {
  await userApi.delete(id)
  handleSearch()
}

onMounted(() => { handleSearch() })
</script>
```

**提炼后 (page.vue3.tera):**
```tera
<template>
  <div class="{{ componentNameKebab }}">
    <el-form :model="searchForm" inline>
{% for field in searchFields %}
      <el-form-item label="{{ field.label }}">
        <el-input v-model="searchForm.{{ field.prop }}" />
      </el-form-item>
{% endfor %}
      <el-form-item>
        <el-button type="primary" @click="handleSearch">搜索</el-button>
      </el-form-item>
    </el-form>

    <el-table :data="tableData" border>
{% for col in tableColumns %}
      <el-table-column prop="{{ col.prop }}" label="{{ col.label }}"{% if col.width %} width="{{ col.width }}"{% endif %} />
{% endfor %}
      <el-table-column label="操作" width="200">
        <template #default="{ row }">
          <el-button @click="handleEdit(row)">编辑</el-button>
          <el-button type="danger" @click="handleDelete(row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { {{ apiModule }}Api } from '@/api/{{ apiModule }}'

defineOptions({ name: '{{ componentName }}' })

const router = useRouter()
const searchForm = ref({% for field in searchFields %}{{ field.prop }}: ''{% if not loop.last %}, {% endif %}{% endfor %})
const tableData = ref([])

const handleSearch = async () => {
  const res = await {{ apiModule }}Api.list(searchForm.value)
  tableData.value = res.data
}

const handleEdit = (row) => {
  router.push(`{{ routePath }}/edit/${row.id}`)
}

const handleDelete = async (id) => {
  await {{ apiModule }}Api.delete(id)
  handleSearch()
}

onMounted(() => { handleSearch() })
</script>
```

### 2. API 服务 → api.js.tera

**源文件 (api/user.js):**
```js
import request from '@/utils/request'

export const userApi = {
  list(params) {
    return request.get('/user/list', { params })
  },
  getById(id) {
    return request.get(`/user/${id}`)
  },
  create(data) {
    return request.post('/user', data)
  },
  update(data) {
    return request.put('/user', data)
  },
  delete(id) {
    return request.delete(`/user/${id}`)
  }
}
```

**提炼后 (api.js.tera):**
```tera
import request from '@/utils/request'

export const {{ apiModule }}Api = {
  list(params) {
    return request.get('/{{ apiModule }}/list', { params })
  },
  getById(id) {
    return request.get(`/{{ apiModule }}/${id}`)
  },
  create(data) {
    return request.post('/{{ apiModule }}', data)
  },
  update(data) {
    return request.put('/{{ apiModule }}', data)
  },
  delete(id) {
    return request.delete(`/{{ apiModule }}/${id}`)
  }
}
```

### 3. 路由配置 → router.js.tera

**源文件 (router/modules/user.js):**
```js
export default {
  path: '/user',
  name: 'User',
  redirect: '/user/list',
  meta: { title: '用户管理', icon: 'user' },
  children: [
    {
      path: 'list',
      name: 'UserList',
      component: () => import('@/views/user/List.vue'),
      meta: { title: '用户列表' }
    }
  ]
}
```

**提炼后 (router.js.tera):**
```tera
export default {
  path: '{{ routePath }}',
  name: '{{ componentName }}',
  redirect: '{{ routePath }}/list',
  meta: { title: '{{ pageTitle }}', icon: '{{ routeIcon }}' },
  children: [
    {
      path: 'list',
      name: '{{ componentName }}List',
      component: () => import('@/views/{{ apiModule }}/List.vue'),
      meta: { title: '{{ pageTitle }}列表' }
    }
  ]
}
```

### 提炼关键点

1. **组件名**: `UserList` → `{{ componentName }}`
2. **CSS类名**: `user-list` → `{{ componentNameKebab }}`
3. **API模块**: `user` → `{{ apiModule }}`
4. **路由路径**: `/user` → `{{ routePath }}`
5. **页面标题**: `用户管理` → `{{ pageTitle }}`
6. **字段循环**: 硬编码表单项/表格列 → `{% for field in ... %}`
7. **模板语法保持一致**: Vue模板语法不需要转义，Tera的`{{ }}`只用于变量替换