<template>
  <div class="wrapper">
    <van-cell-group title="安全设置">
      <van-cell
        v-if="showBiometric"
        title="指纹/面容登录"
        label="登录时验证本机指纹或面容"
      >
        <template #right-icon>
          <van-switch v-model="enableBio" :loading="loading" @change="onBiometric" />
        </template>
      </van-cell>

      <van-cell title="支付密码" label="付款时验证支付密码" is-link @click="onSetPassword" />

      <van-cell title="双因素验证支付" label="付款时验证Google或微软验证器生成的动态密码">
        <template #right-icon>
          <van-switch v-model="enable2fa" :loading="loading2fa" @change="on2FA" />
        </template>
      </van-cell>

      <!-- <van-cell
        title="导出助记词"
        label="用于恢复或迁移钱包, 丢失将导致资产损失"
        is-link
        @click="onMnemonic"
      /> -->
    </van-cell-group>

    <van-cell-group title="钱包">
      <van-cell
        title="重置钱包"
        label="用新的助记词重置钱包, 重置前请备份老助记词"
        is-link
        @click="showResetDlg"
      />
    </van-cell-group>

    <van-cell-group title="关于妙蛙钱包">
      <van-cell>
        <template #title>
          <span class="custom-title">
            <span style="font-weight: bold">妙蛙钱包</span>
            是一款独立加密钱包小程序。它由知名Web3团队打造，帮助您更加安全、方便地管理加密资产。
          </span>
        </template>
      </van-cell>
    </van-cell-group>
  </div>

  <van-action-sheet
    v-model:show="show"
    :actions="actions"
    cancel-text="取消"
    close-on-click-action
    @select="onPwAction"
  />

  <van-action-sheet
    v-model:show="showReset"
    :actions="resetActions"
    cancel-text="取消"
    close-on-click-action
    @select="onResetAction"
  />

  <van-popup
    v-if="showPassword"
    v-model:show="showPassword"
    position="bottom"
    :style="{ height: '80%' }"
  >
    <Password @password="onPassword" @cancel="cancelPassword" />
  </van-popup>

  <van-popup
    v-if="show2FACode"
    v-model:show="show2FACode"
    position="bottom"
    :style="{ height: '80%' }"
  >
    <Password
      @password="onUnBind"
      info="请输入验证器上的数字"
      @cancel="cancel2FACode"
      :loading="loading2fa"
    />
  </van-popup>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { showToast } from 'vant'
import Password from '@/components/wallet/Password.vue'
import request from '@/common/request'

const router = useRouter()
const user = ref(null)
const showBiometric = ref(false)
const enableBio = ref(false)
const show = ref(false)
const hasPassword = ref(false)
const loading = ref(false)
const showPassword = ref(false)
const localToken = ref(null)
const enable2fa = ref(false)
const loading2fa = ref(false)
const show2FACode = ref(false)
const actions = [
  { name: '更改密码', val: 'modify' },
  { name: '禁用密码', val: 'remove', color: '#ee0a24' }
]
const showReset = ref(false)
const resetActions = [{ name: '用"新的助记词"重建钱包', val: 'reset', color: '#ee0a24' }]
let passwordAction = null

function isMobileOS() {
  const ua = navigator.userAgent
  return /Android|iPhone|iPad|iPod/i.test(ua)
}

onMounted(() => {
  enable2fa.value = localStorage.getItem('enable2fa') === 'true'
  hasPassword.value = localStorage.getItem('hasPassword') === 'true'
  localToken.value = localStorage.getItem('token')
  user.value = localStorage.getItem('user') ? JSON.parse(localStorage.getItem('user')) : null
  if (user.value && user.value.id) {
    enableBio.value = localStorage.getItem(`${user.value.id}:biometric`) === 'true'
  }
  showBiometric.value = isMobileOS()
})

// ---------------------------------
// Пароль
const onSetPassword = () => {
  if (loading.value) return
  if (!hasPassword.value) {
    localStorage.setItem('passwordType', 'new')
    router.push({ name: 'Password' })
  } else {
    show.value = true
  }
}

const onPwAction = (action) => {
  localStorage.setItem('passwordType', action.val)
  router.push({ name: 'Password' })
}

// ---------------------------------
// Сброс кошелька
const showResetDlg = () => {
  if (!hasPassword.value) {
    return showToast({
      message: '请先设置支付密码',
      position: 'bottom'
    })
  }
  showReset.value = true
}

const onResetAction = (action) => {
  if (action.val === 'reset') {
    passwordAction = 'reset'
    showPassword.value = true
  }
}

// ---------------------------------
// Биометрия (локальный switch)
const onBiometric = () => {
  if (!hasPassword.value) {
    enableBio.value = false
    return showToast({
      message: '请先设置支付密码',
      position: 'bottom'
    })
  }
  localStorage.setItem(`${user.value?.id || 'default'}:biometric`, enableBio.value)
  showToast({
    message: enableBio.value ? '生物识别已开启' : '生物识别已关闭',
    position: 'bottom'
  })
}

// ---------------------------------
// 2FA
const on2FA = async () => {
  try {
    loading2fa.value = true
    if (enable2fa.value) {
      let res = await request('/api/user/get_2fa', 'POST', {})
      loading2fa.value = false
      if (res.code && res.type === 'login') {
        return router.replace({ name: 'Login' })
      } else if (!res.code) {
        localStorage.setItem('otpauthUrl', res.data.otpauthUrl)
        router.push({ name: 'Enable2FA' })
      } else {
        if (res.type !== '2fa_bind') {
          showToast({ message: res.message, position: 'bottom' })
          enable2fa.value = !enable2fa.value
        }
      }
    } else {
      show2FACode.value = true
    }
  } catch (error) {
    loading2fa.value = false
    enable2fa.value = !enable2fa.value
    showToast({ message: error.message, position: 'bottom' })
  }
}

const onUnBind = async (code) => {
  try {
    show2FACode.value = false
    let res = await request('/api/user/disable_2fa', 'POST', { code })
    loading2fa.value = false
    if (res.code && res.type === 'login') {
      return router.replace({ name: 'Login' })
    } else if (!res.code) {
      showToast({ message: '成功关闭双因素验证', position: 'bottom' })
      localStorage.setItem('enable2fa', false)
      enable2fa.value = false
    } else {
      enable2fa.value = !enable2fa.value
      showToast({ message: res.message, position: 'bottom' })
    }
  } catch (error) {
    loading2fa.value = false
    showToast({ message: error.message, position: 'bottom' })
  }
}

// ---------------------------------
// Обработка пароля (биометрия больше не использует Telegram)
const cancelPassword = () => {
  showPassword.value = false
  if (loading.value) loading.value = false
}

const cancel2FACode = () => {
  show2FACode.value = false
  if (loading2fa.value) loading2fa.value = false
}

const onPassword = async (password) => {
  try {
    showPassword.value = false
    loading.value = true
    if (passwordAction === 'reset') {
      let res = await request('/api/wallet/reset', 'POST', { password })
      loading.value = false
      if (res.code && res.type === 'login') return router.replace({ name: 'Login' })
      if (!res.code) return router.replace({ name: 'Login' })
      showToast({ message: res.message, position: 'bottom' })
    }
  } catch (error) {
    loading.value = false
    showToast({ message: error.message, position: 'bottom' })
  }
}
</script>

<style scoped lang="scss">
.wrapper {
  padding: 20px 0;
}
</style>