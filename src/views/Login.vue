<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'

const router = useRouter()

const username = ref('admin')
const password = ref('123456')
const loading = ref(false)

async function login() {
  if (!username.value || !password.value) {
    ElMessage.warning('请输入用户名和密码')
    return
  }

  loading.value = true
  try {
    const res = await axios.post('http://localhost:8080/api/auth/login', {
      username: username.value,
      password: password.value
    })

    if (res.data.code === 200) {
      localStorage.setItem('accessToken', res.data.data.accessToken)
      localStorage.setItem('refreshToken', res.data.data.refreshToken)
      localStorage.setItem('username', res.data.data.username)
      ElMessage.success('登录成功')
      setTimeout(() => router.push('/users'), 500)
    } else {
      ElMessage.error(res.data.message || '登录失败')
    }
  } catch (e) {
    if (e.response) {
      ElMessage.error(e.response.data.message || '登录失败')
    } else {
      ElMessage.error('网络错误')
    }
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="login-container">
    <el-card class="login-card">
      <template #header>
        <h2 style="text-align: center; margin: 0;">用户管理系统</h2>
      </template>

      <el-form label-position="top">
        <el-form-item label="用户名">
          <el-input v-model="username" placeholder="请输入用户名" size="large" />
        </el-form-item>

        <el-form-item label="密码">
          <el-input
              v-model="password"
              type="password"
              placeholder="请输入密码"
              size="large"
              show-password
              @keyup.enter="login"
          />
        </el-form-item>

        <el-button
            type="primary"
            size="large"
            style="width: 100%;"
            :loading="loading"
            @click="login"
        >
          {{ loading ? '登录中...' : '登录' }}
        </el-button>
      </el-form>
    </el-card>
  </div>
</template>

<style scoped>
.login-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
.login-card {
  width: 400px;
}
</style>