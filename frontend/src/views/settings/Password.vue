<template>
  <div class="wrapper" v-if="!loading">
    <!-- 密码输入框 -->
    <van-password-input
      :value="password"
      :focused="showKeyboard"
      @focus="showKeyboard = true"
      :error-info="errorInfo"
      :info="info"
    />
    <!-- 数字键盘 -->
    <van-number-keyboard v-model="password" :show="showKeyboard" @blur="showKeyboard = false" />
  </div>
  <div class="loading" v-if="loading">
    <van-loading size="24px" vertical>加载中...</van-loading>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { showToast } from 'vant'
import request from '@/common/request'

const router = useRouter()
const user = ref(null)
const password = ref('')
const showKeyboard = ref(true)
const errorInfo = ref('')
const loading = ref(false)
const info = ref('请输入支付密码')
const type = ref(null) // new.新设密码 modify.修改密码 remove.关闭密码

let oldPassword = null
let firstPassword = null
let secondPassword = null

onMounted(() => {
  user.value = localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : null
  type.value = localStorage.getItem('passwordType') || 'new'
  if (type.value === 'modify') {
    info.value = '请输入原支付密码'
  }
})

// ----------------------------
// Слежение за вводом пароля
watch(password, (newVal) => {
  if (newVal !== '') errorInfo.value = ''
  if (newVal.length === 6) {
    switch (type.value) {
      case 'new':
        onNewPassword(newVal)
        break
      case 'remove':
        onRemovePassword(newVal)
        break
      case 'modify':
        onModifyPassword(newVal)
        break
    }
    password.value = ''
  }
})

// ----------------------------
// 新密码
const onNewPassword = async (newVal) => {
  if (!firstPassword) {
    firstPassword = newVal
    info.value = '请再次输入密码'
    return
  }
  secondPassword = newVal
  info.value = ''
  if (firstPassword !== secondPassword) {
    firstPassword = null
    errorInfo.value = '两次输入的密码不一致'
    info.value = '请输入支付密码'
    return
  }
  try {
    loading.value = true
    let res = await request('/api/user/password', 'POST', { password: secondPassword })
    secondPassword = null
    loading.value = false
    if (res.code && res.type === 'login') return router.replace({ name: 'Login' })
    if (!res.code) {
      showToast({ message: '成功设置密码', position: 'bottom' })
      localStorage.setItem('hasPassword', 'true')
      return router.back()
    }
    firstPassword = null
    secondPassword = null
    info.value = '请输入新密码'
    showToast({ message: res.message, position: 'bottom' })
  } catch (error) {
    loading.value = false
    firstPassword = null
    secondPassword = null
    showToast({ message: error.message, position: 'bottom' })
  }
}

// ----------------------------
// 禁用密码
const onRemovePassword = async (newVal) => {
  try {
    loading.value = true
    let res = await request('/api/user/password', 'POST', { password: null, old_password: newVal })
    loading.value = false
    if (res.code && res.type === 'login') return router.replace({ name: 'Login' })
    if (!res.code) {
      showToast({ message: '成功禁用密码', position: 'bottom' })
      localStorage.setItem('hasPassword', 'false')
      localStorage.setItem(`${user.value?.id || 'default'}:biometric`, false)
      return router.back()
    }
    showToast({ message: res.message, position: 'bottom' })
  } catch (error) {
    loading.value = false
    showToast({ message: error.message, position: 'bottom' })
  }
}

// ----------------------------
// 修改密码
const onModifyPassword = async (newVal) => {
  if (!oldPassword) {
    oldPassword = newVal
    info.value = '请输入新密码'
    return
  }
  if (!firstPassword) {
    firstPassword = newVal
    info.value = '请再次输入新密码'
    return
  }
  secondPassword = newVal
  info.value = ''
  if (firstPassword !== secondPassword) {
    firstPassword = null
    errorInfo.value = '两次输入的密码不一致'
    info.value = '请输入新密码'
    return
  }
  try {
    loading.value = true
    let res = await request('/api/user/password', 'POST', { password: secondPassword, old_password: oldPassword })
    secondPassword = null
    loading.value = false
    if (res.code && res.type === 'login') return router.replace({ name: 'Login' })
    if (!res.code) {
      showToast({ message: '成功修改密码', position: 'bottom' })
      localStorage.setItem('hasPassword', 'true')
      return router.back()
    }
    oldPassword = null
    firstPassword = null
    secondPassword = null
    info.value = '请输入新密码'
    showToast({ message: res.message, position: 'bottom' })
  } catch (error) {
    loading.value = false
    oldPassword = null
    firstPassword = null
    secondPassword = null
    showToast({ message: error.message, position: 'bottom' })
  }
}
</script>

<style scoped lang="scss">
.wrapper {
  padding: 20px 10px;
}
.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 60vh;
}
</style>