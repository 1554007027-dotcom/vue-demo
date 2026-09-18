<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

const router = useRouter()

const users = ref([])
const loading = ref(false)
const message = ref('')
const username = ref(localStorage.getItem('username') || '')

function getToken() {
  return localStorage.getItem('accessToken')
}

async function loadUsers() {
  loading.value = true
  message.value = ''

  try {
    const res = await axios.get('http://localhost:8080/api/users', {
      headers: { 'Authorization': 'Bearer ' + getToken() }
    })

    if (res.data.code === 200) {
      users.value = res.data.data || []
    } else {
      message.value = '加载失败：' + res.data.message
    }
  } catch (e) {
    if (e.response && (e.response.status === 401 || e.response.status === 403)) {
      message.value = '登录已过期，请重新登录'
      setTimeout(() => router.push('/login'), 1000)
    } else if (e.response) {
      message.value = '错误：' + (e.response.data.message || e.response.status)
    } else {
      message.value = '网络错误：' + e.message
    }
  } finally {
    loading.value = false
  }
}

async function deleteUser(id) {
  if (!confirm('确定要删除这个用户吗？')) return

  try {
    const res = await axios.delete(`http://localhost:8080/api/users/${id}`, {
      headers: { 'Authorization': 'Bearer ' + getToken() }
    })

    if (res.status === 204 || res.data.code === 200) {
      loadUsers()
    } else {
      alert(res.data.message || '删除失败')
    }
  } catch (e) {
    if (e.response) {
      alert(e.response.data.message || '删除失败')
    } else {
      alert('网络错误：' + e.message)
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
    <header>
      <h1>用户管理</h1>
      <div class="user-info">
        <span>👤 {{ username }}</span>
        <button class="logout-btn" @click="logout">退出登录</button>
      </div>
    </header>

    <div class="toolbar">
      <button class="btn-primary" @click="loadUsers" :disabled="loading">
        {{ loading ? '加载中...' : '刷新' }}
      </button>
    </div>

    <p v-if="message" class="msg">{{ message }}</p>

    <div class="card">
      <table>
        <thead>
        <tr>
          <th>ID</th>
          <th>用户名</th>
          <th>邮箱</th>
          <th>角色</th>
          <th>操作</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="u in users" :key="u.id">
          <td>{{ u.id }}</td>
          <td>{{ u.username }}</td>
          <td>{{ u.email }}</td>
          <td>{{ u.role || '-' }}</td>
          <td>
            <button class="btn-sm btn-delete" @click="deleteUser(u.id)">删除</button>
          </td>
        </tr>
        <tr v-if="users.length === 0">
          <td colspan="5" class="empty">暂无用户</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.container { max-width: 1100px; margin: 30px auto; padding: 0 20px; font-family: sans-serif; }
header { background: #fff; padding: 20px 30px; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
h1 { font-size: 22px; color: #333; margin: 0; }
.user-info { display: flex; align-items: center; gap: 15px; }
.user-info span { color: #666; font-size: 14px; }
.logout-btn { padding: 8px 16px; background: #e74c3c; color: #fff; border: none; border-radius: 6px; cursor: pointer; }
.toolbar { background: #fff; padding: 20px 30px; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); margin-bottom: 20px; }
.btn-primary { padding: 10px 20px; background: #42b883; color: #fff; border: none; border-radius: 6px; cursor: pointer; font-size: 14px; }
.card { background: #fff; border-radius: 12px; box-shadow: 0 2px 12px rgba(0,0,0,0.06); overflow: hidden; }
table { width: 100%; border-collapse: collapse; }
th, td { padding: 14px 20px; text-align: left; border-bottom: 1px solid #eee; }
th { background: #f8f9fc; color: #555; font-size: 13px; text-transform: uppercase; }
td { color: #333; font-size: 14px; }
tr:hover { background: #f8f9fc; }
.empty { text-align: center; padding: 40px; color: #999; }
.btn-sm { padding: 6px 12px; font-size: 12px; border-radius: 4px; cursor: pointer; border: none; }
.btn-delete { background: #e74c3c; color: #fff; }
.msg { padding: 15px; border-radius: 5px; background: #f0f0f0; text-align: center; }
</style>