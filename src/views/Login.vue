<script setup>
import { ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

const router = useRouter()

const username = ref('admin')
const password = ref('123456')
const message = ref('')
const loading = ref(false)

async function login() {
  loading.value = true
  message.value = ''

  try {
    const res = await axios.post('http://localhost:8080/api/auth/login', {
      username: username.value,
      password: password.value
    })

    if (res.data.code === 200) {
      message.value = '登录成功，跳转中...'
      localStorage.setItem('accessToken', res.data.data.accessToken)
      localStorage.setItem('refreshToken', res.data.data.refreshToken)
      localStorage.setItem('username', res.data.data.username)

      setTimeout(() => {
        router.push('/users')
      }, 500)
    } else {
      message.value = '登录失败：' + res.data.message
    }
  } catch (e) {
    if (e.response) {
      message.value = '错误：' + (e.response.data.message || e.response.status)
    } else {
      message.value = '网络错误：' + e.message
    }
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="app">
    <h1>用户管理系统</h1>
    <div class="form">
      <input v-model="username" placeholder="用户名" />
      <input v-model="password" type="password" placeholder="密码" />
      <button @click="login" :disabled="loading">
        {{ loading ? '登录中...' : '登录' }}
      </button>
    </div>
    <p v-if="message" class="msg">{{ message }}</p>
  </div>
</template>

<style scoped>
.app { max-width: 600px; margin: 100px auto; padding: 30px; font-family: sans-serif; }
h1 { color: #42b883; text-align: center; }
.form { display: flex; flex-direction: column; gap: 10px; margin: 30px 0; }
input { padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; }
button { padding: 12px; background: #42b883; color: white; border: none; border-radius: 5px; cursor: pointer; font-size: 16px; }
button:hover:not(:disabled) { background: #369c6c; }
button:disabled { background: #aaa; cursor: not-allowed; }
.msg { padding: 15px; border-radius: 5px; background: #f0f0f0; text-align: center; }
</style>