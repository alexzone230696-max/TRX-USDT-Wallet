<template>
  <div class="content">
    <van-row class="welcome">
      <div class="nickname">{{ firstName }}</div>

      <div class="settings" @click="onSettings">
        <van-icon name="setting-o" size="24" />
      </div>
    </van-row>

    <div class="child">
      <Wallet />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import Wallet from '@/components/home/Wallet.vue'
import router from '../../router'

const firstName = ref('User')

// Загрузка имени из localStorage (если есть)
onMounted(() => {
  const saved = localStorage.getItem('user')

  if (saved) {
    try {
      const parsed = JSON.parse(saved)
      if (parsed.firstName) {
        firstName.value = parsed.firstName
      }
    } catch (e) {
      console.error('localStorage user parse error:', e)
    }
  }
})

// Переход в настройки
const onSettings = () => {
  router.push({ name: 'Settings' })
}
</script>

<style scoped lang="scss">
.content {
  display: flex;
  flex-direction: column;
  height: 100vh;

  .welcome {
    display: flex;
    padding: 20px 10px 10px 10px;
    align-items: center;
    justify-content: center;
    .nickname {
      flex-grow: 1;
      font-weight: bolder;
    }
    .settings {
      cursor: pointer;
    }
  }

  .child {
    flex-grow: 1;
    overflow: hidden;
  }
}
</style>