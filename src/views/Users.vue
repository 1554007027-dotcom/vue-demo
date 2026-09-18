<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'

const router = useRouter()

const users = ref([])
const loading = ref(false)
const username = ref(localStorage.getItem('username') || '')

const showModal = ref(false)
const editingId = ref(null)
const form = ref({ username: '', email: '', password: '' })

function getToken() {
  return localStorage.getItem('accessToken')
}

async function loadUsers() {
  loading.value = true
  try {
    const res = await axios.get('http://localhost:8080/api/users', {
      headers: { 'Authorization': 'Bearer ' + getToken() }
    })
    if (res.data.code === 200) {
      users.value = res.data.data || []
    } else {
      ElMessage.error(res.data.message)
    }
  } catch (e) {
    if (e.response && (e.response.status === 401 || e.response.status === 403)) {
      ElMessage.error('登录已过期')
      setTimeout(() => router.push('/login'), 1000)
    } else {
      ElMessage.error('加载失败')
    }
  } finally {
    loading.value = false
  }
}

function openCreate() {
  editingId.value = null
  form.value = { username: '', email: '', password: '' }
  showModal.value = true
}

function openEdit(user) {
  editingId.value = user.id
  form.value = { username: user.username, email: user.email, password: '' }
  showModal.value = true
}

async function saveUser() {
  if (!form.value.username || !form.value.email) {
    ElMessage.warning('用户名和邮箱必填')
    return
  }
  if (!editingId.value && !form.value.password) {
    ElMessage.warning('新增用户必须填密码')
    return
  }

  const body = { username: form.value.username, email: form.value.email }
  if (form.value.password) body.password = form.value.password

  try {
    let res
    if (editingId.value) {
      res = await axios.put(
          `http://localhost:8080/api/users/${editingId.value}`,
          body,
          { headers: { 'Authorization': 'Bearer ' + getToken() } }
      )
    } else {
      res = await axios.post(
          'http://localhost:8080/api/users',
          body,
          { headers: { 'Authorization': 'Bearer ' + getToken() } }
      )
    }

    if (res.data.code === 200 || res.status === 201) {
      ElMessage.success(editingId.value ? '更新成功' : '创建成功')
      showModal.value = false
      loadUsers()
    } else {
      ElMessage.error(res.data.message || '操作失败')
    }
  } catch (e) {
    if (e.response) {
      ElMessage.error(e.response.data.message || '操作失败')
    } else {
      ElMessage.error('网络错误')
    }
  }
}

async function deleteUser(id) {
  try {
    await ElMessageBox.confirm('确定要删除这个用户吗？', '提示', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning'
    })
  } catch {
    return
  }

  try {
    const res = await axios.delete(`http://localhost:8080/api/users/${id}`, {
      headers: { 'Authorization': 'Bearer ' + getToken() }
    })
    if (res.status === 204 || res.data.code === 200) {
      ElMessage.success('删除成功')
      loadUsers()
    } else {
      ElMessage.error(res.data.message || '删除失败')
    }
  } catch (e) {
    if (e.response) {
      ElMessage.error(e.response.data.message || '删除失败')
    } else {
      ElMessage.error('网络错误')
    }
  }
}

function logout() {
  localStorage.clear()
  router.push('/login')
}

onMounted(() => {
  if (!getToken()) {
    router.push('/login')
    return
  }
  loadUsers()
})
</script>

<template>
  <div class="container">
    <el-card>
      <div class="header">
        <h2>用户管理</h2>
        <div class="user-info">
          <span>👤 {{ username }}</span>
          <el-button type="danger" size="small" @click="logout">退出登录</el-button>
        </div>
      </div>
    </el-card>

    <el-card style="margin-top: 20px;">
      <el-button type="primary" @click="openCreate">+ 新增用户</el-button>
      <el-button @click="loadUsers" :loading="loading">刷新</el-button>
    </el-card>

    <el-card style="margin-top: 20px;">
      <el-table :data="users" v-loading="loading" stripe>
        <el-table-column prop="id" label="ID" width="80" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="email" label="邮箱" />
        <el-table-column prop="role" label="角色" width="120" />
        <el-table-column label="操作" width="180">
          <template #default="{ row }">
            <el-button size="small" type="warning" @click="openEdit(row)">编辑</el-button>
            <el-button size="small" type="danger" @click="deleteUser(row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog
        v-model="showModal"
        :title="editingId ? '编辑用户' : '新增用户'"
        width="400px"
    >
      <el-form label-position="top">
        <el-form-item label="用户名">
          <el-input v-model="form.username" />
        </el-form-item>
        <el-form-item label="邮箱">
          <el-input v-model="form.email" />
        </el-form-item>
        <el-form-item label="密码">
          <el-input
              v-model="form.password"
              type="password"
              :placeholder="editingId ? '不修改留空' : '请输入密码'"
              show-password
          />
        </el-form-item>
      </el-form>

      <template #footer>
        <el-button @click="showModal = false">取消</el-button>
        <el-button type="primary" @click="saveUser">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<style scoped>
.container {
  max-width: 1100px;
  margin: 30px auto;
  padding: 0 20px;
}
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.header h2 { margin: 0; color: #333; }
.user-info { display: flex; align-items: center; gap: 15px; }
.user-info span { color: #666; font-size: 14px; }
</style>